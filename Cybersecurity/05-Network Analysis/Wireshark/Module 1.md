---
tags: [cybersecurity, wireshark, networking, lab, arp]
date: 2026-11-06
module: Wireshark 1
status: done, with open items
---

# Wireshark Module 1: packets, layers and ARP

> [!summary] Summary
> I captured my own lab traffic and used it to study how a frame is built, why ARP is needed before local IP communication, how ping works at packet level, and how Linux keeps its ARP cache fresh. I tested each claim with a capture or a command instead of trusting an explanation.
>
> **Public-copy privacy:** the real interface MAC and IP addresses from my capture have been replaced. `KALI_MAC` and `BRIDGE_MAC` are labels, while the IP addresses shown below are documentation-only TEST-NET placeholders. They are not the live or historical addresses of my VMs.

Related: [Host Inventory & Lab Setup](../../01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md) | Next planned topic: **Wireshark Module 2 — TCP and HTTP** (not documented yet).
## Lab topology

```
Kali VM                       eth0  KALI_MAC   192.0.2.88
   │
[ virbr1: virtual switch "isolated-lab", 192.0.2.0/24, no forward tag ]
   │
Host (Ubuntu)                 virbr1  BRIDGE_MAC   192.0.2.1
```

- The host has an address on this network. The inspected `isolated-lab` definition has no libvirt `<forward>` element, so libvirt is not configured to provide NAT or routed forwarding for this network. This does not make the segment an air gap: the host is attached to it and host routing, services or firewall/bridge rules can affect reachability.
- QEMU/libvirt commonly assigns virtual NICs MAC addresses from the `52:54:00` prefix; the actual values are redacted in this public copy.

## Glossary

|Term|Meaning (my words)|
|---|---|
|NIC|Network card. Here it is virtual (virtio), not a physical port|
|MAC address|Address of a network card on the local link, 6 bytes, e.g. `00:11:22:33:44:55`|
|IP address|Address used to deliver packets across networks|
|Subnet `/24`|The first 3 numbers are the network, the last one identifies the device|
|Frame / packet / segment|Ethernet layer unit / IP layer unit / TCP layer unit|
|Unicast|Sent to one specific device|
|Broadcast|Sent to everyone on the link (`ff:ff:ff:ff:ff:ff`)|
|Switch / bridge|Connects devices on one local network. `virbr1` is a virtual one|
|ARP|Maps an IP address to a MAC address on a local link|
|ICMP|Control protocol used by `ping`. Rides directly on IP|
|TTL|Counter in the IP header. Each router subtracts 1|
|Neighbor table|The ARP cache on Linux (`ip neigh`)|
|virtio|Paravirtualized virtual hardware, faster because the guest knows it is virtual|

## How Wireshark works

- Wireshark doesn't read the cable itself. A helper (`dumpcap`) asks the kernel for a copy of each frame on an interface, and Wireshark decodes the copies.
- The helper needs privileges. Being in the `wireshark` group allows that without running the whole GUI as root, which is safer because a bug in a packet parser would otherwise run as root.
- Three panes: packet list (top), packet details showing the layers (middle), raw bytes (bottom).
- **Capture filter** limits what is recorded (e.g. `host 192.0.2.1`). **Display filter** only hides packets afterwards (e.g. `arp || icmp`).
- My lab is a switch, so Kali only sees traffic to and from itself plus broadcasts. It cannot see other machines' conversations.
- Pick the interface that has the `192.0.2.x` address (check with `ip -br a`), not a guess.

## Layers

```
Ethernet frame    MAC addresses: who gets this frame on this local link
 └ IP packet      IP addresses and TTL: who, across networks
    └ TCP segment ports, sequence numbers, flags: which conversation, in order
       └ HTTP     the request or response text
```

- The details pane reads top to bottom, outside in.
- ICMP (ping) rides directly on IP. No transport layer, so no ports and no handshake.
- ARP sits directly on Ethernet and does not use IP. That is why it cannot cross a router.

**How each layer says what is inside it (to verify in my capture)**

|Where|Field|Value|
|---|---|---|
|Ethernet|EtherType|`0x0806` = ARP, `0x0800` = IPv4|
|IP|Protocol|1 = ICMP, 6 = TCP, 17 = UDP|

**Frame sizes** (the Length column is not part of the Info text)

|Frame|Total|Breakdown|
|---|---|---|
|ARP|42 bytes|14 Ethernet + 28 ARP message|
|Ping|98 bytes|14 Ethernet + 20 IP + 8 ICMP header + 56 data|

## My capture, packet by packet

Display filter: `arp || icmp`. The packet numbers skip (94, 95, 96, 97, 99...) because the filter hides other packets.

|No.|Time (s)|Source|Destination|Protocol|Info|
|---|---|---|---|---|---|
|94|17.185889|`KALI_MAC`|Broadcast|ARP|Who has 192.0.2.1? Tell 192.0.2.88|
|95|17.186053|`BRIDGE_MAC`|`KALI_MAC`|ARP|192.0.2.1 is at BRIDGE_MAC|
|96|17.186080|192.0.2.88|192.0.2.1|ICMP|Echo request, seq=1, ttl=64|
|97|17.186289|192.0.2.1|192.0.2.88|ICMP|Echo reply, seq=1, ttl=64|
|99|18.213490|192.0.2.88|192.0.2.1|ICMP|Echo request, seq=2|
|100|18.214131|192.0.2.1|192.0.2.88|ICMP|Echo reply, seq=2|
|107|19.214965|192.0.2.88|192.0.2.1|ICMP|Echo request, seq=3|
|108|19.215373|192.0.2.1|192.0.2.88|ICMP|Echo reply, seq=3|

What I read from it:

- ARP rows show **MAC** addresses in Source and Destination, ICMP rows show **IPs**. ARP has no IP layer, so Wireshark falls back to the Ethernet addresses.
- ARP request to reply: about 0.16 ms. The first ping went out right after (about 0.03 ms later), because it had to wait for the ARP answer.
- Ping reply times: seq 1 about 0.21 ms, seq 2 about 0.64 ms, seq 3 about 0.41 ms. Virtual links, so tiny but not constant.
- Pings are about 1 second apart (the default of `ping`).
- All echo requests share `id=0xd5f4`: that identifies this ping process. `seq` counts up.
- TTL 64 on the reply: Linux starts at 64 and each router subtracts 1, so 64 on arrival means **no router was crossed**.
- Wireshark cross-references the pairs ("reply in 97", "request in 96").

## ARP

**Problem:** IP identifies the destination, but Ethernet frames on a local link are delivered using MAC addresses. Before sending, Kali needs the MAC address of the next hop: the destination itself when it is on the same subnet, or the configured gateway when the destination is on another network.

```
Kali -> broadcast: "Who has 192.0.2.1? Tell 192.0.2.88"
Host -> Kali:      "192.0.2.1 is at BRIDGE_MAC"
```

- The request is a **broadcast** (`ff:ff:ff:ff:ff:ff`) because Kali doesn't know who to ask.
- The reply is **unicast**, because the request already told the host Kali's MAC.
- The target MAC in the request is `00:00:00:00:00:00`, because that is the thing being asked.
- IP identifies the destination across networks. The Ethernet destination MAC identifies the next hop on the current link. ARP resolves an IPv4 address on that local link to a MAC address; for off-subnet traffic, the address being resolved is normally the gateway, not the remote destination. Packet 96's destination MAC is exactly the one the ARP reply taught Kali.
- If the target is outside the subnet, Kali asks for its **gateway's** MAC instead.

**Fields I read in the ARP packet:** hardware type Ethernet (1), protocol type IPv4 (0x0800), hardware size 6, protocol size 4, opcode request (1) or reply (2), sender MAC and IP, target MAC and IP.

**Addresses I verified myself**

|MAC|Belongs to|Found with|
|---|---|---|
|`KALI_MAC`|Kali `eth0`|`ip link show`|
|`BRIDGE_MAC`|host `virbr1`|`ip a show virbr1`|

## ICMP and ping

- `ping` sends an ICMP Echo request (type 8). The target answers with an Echo reply (type 0).
- The ICMP message is 64 bytes (8 header + 56 data), which is the "64 bytes from..." that ping prints.
- ICMP carries no ports. It is a control protocol, so there is no handshake and no stream.
- In my recorded experiment, the local "Destination Host Unreachable" message appeared after ARP resolution to the target failed (see [Failure signature](Module%201.md#failure-signature)). This interpretation is specific to that capture and route context.

## ARP cache

- Kali stores the IP to MAC answer in the neighbor table (`ip neigh`) and reuses it. That is why a second ping caused no ARP.
- The table only ever shows `BRIDGE_MAC` for `.1`. It stores the **neighbor's** MAC, never my own.

**Entry states**

```
REACHABLE -> STALE -> DELAY -> (PROBE) -> REACHABLE
FAILED: nobody answered
```

- REACHABLE: recently confirmed, used without asking.
- STALE: old confirmation, still used.
- DELAY: the entry was just used, so the kernel waits for proof it still works.
- PROBE: no proof arrived, so ARP requests are sent to verify.

**Settings I read from my machine**

|Setting|My value|Meaning|
|---|---|---|
|`mcast_solicit`|3|ARP attempts before FAILED (matches the 3 requests I saw)|
|`base_reachable_time_ms`|30000|REACHABLE lasts about 15 to 45 s (randomized)|
|`delay_first_probe_time`|5|seconds to wait in DELAY for proof of life|

**What I observed with a STALE entry**

1. I waited for `STALE`, then pinged.
2. The ICMP packet went out immediately with **no ARP before it**.
3. The entry moved to `DELAY`, then to `REACHABLE`.

So a stale entry is **used first and verified afterwards**, not checked before use. If no proof arrives in the 5 seconds, the kernel moves to `PROBE`.

> [!note] Limit of my observation `watch -n1` samples once per second, so a short PROBE could be missed. The capture is the real evidence.

## Experiments

|#|What I did|What I saw|What it shows|
|---|---|---|---|
|1|`sudo ip neigh flush all`, then `ping -c 3 192.0.2.1`|ARP request, ARP reply, then 3 pings|ARP happens before the first packet|
|2|Pinged again without flushing|No ARP|The cache is used|
|3|Pinged an address that doesn't exist (`.5`)|3 ARP requests, no reply, entry `FAILED`|The failure signature|
|4|Pinged while `STALE`|No ARP before the ping, then DELAY, then REACHABLE|Use first, verify later|

## Failure signature

> [!tip] Pattern to remember Repeated ARP requests with no reply followed by a `FAILED` neighbor entry show that address resolution did not complete. Possible causes include an offline or absent host, the wrong subnet/VLAN, a disconnected interface, or filtering/segmentation at Layer 2. The message alone does not prove the target is powered off. This differs from a TCP connection failure, which happens after IP connectivity and transport-layer behavior are considered.

## Scale and limits

- Kali only keeps entries for machines it talks to, not all of them.
- Every ARP request is a broadcast, so on a network of 100 machines all of them receive it and only the owner answers. Large flat networks get noisy, which is one reason networks are split into subnets or VLANs.
- Linux caps the number of neighbor entries (`gc_thresh1` to `gc_thresh3`).
- Nmap commonly uses ARP for host discovery on a directly connected Ethernet network when its default host-discovery behavior applies. ARP is not normally handled by an ordinary IP-layer host firewall, but Layer 2 filtering, VLAN separation or network controls can still prevent a reply.

## Security notes

- ARP has **no authentication**. Any machine on the link can claim an IP, and the receiver caches it. That is **ARP spoofing**, which I'll practice later in my lab.
- Red flags a defender can watch (to research): the MAC for the gateway changing, two MACs claiming one IP, unsolicited ARP replies.
- Known defenses to look up: static ARP entries, `arpwatch`, and dynamic ARP inspection on managed switches.
- A scan of a local range shows up as many ARP requests to addresses that never answer.

## Mistakes I corrected

|What I thought|What is true|
|---|---|
|`virbr1` is the router port linking the lab to my home LAN|It is the host's connection to a virtual Layer 2 switch. The inspected libvirt network has no `<forward>` element, but host-level access and routing still need to be considered|
|`eth0` is a physical port|It is a virtual card (virtio)|
|The `42` is part of the Info text|It is the Length column|
|The ARP table might show different MACs per state|It always shows the neighbor's MAC. Mine never appears there|
|I pinged `.50` and the packet said `.5`|I pinged `.5` myself. No discrepancy|
|A stale entry is checked before use|It is used immediately and verified afterwards|

## Self-test

> [!question]- 1. Why does ping cause ARP traffic first? Ping knows the destination IP, but Ethernet delivers by MAC, so Kali must ask "who has this IP?" before the first ICMP packet can be framed.

> [!question]- 2. Why is the ARP request a broadcast and the reply a unicast? The asker doesn't know the owner's MAC, so it asks everyone. The request already carried the asker's MAC, so the owner can answer directly.

> [!question]- 3. Why is the target MAC all zeros in the request? That is the unknown being asked for.

> [!question]- 4. Which layer is missing from a ping compared with a web request? Transport (TCP). ICMP rides directly on IP.

> [!question]- 5. What does TTL 64 on the reply tell me? Linux starts at 64 and each router subtracts 1, so no router was crossed.

> [!question]- 6. Why did the second ping cause no ARP? The neighbor table already had the mapping.

> [!question]- 7. Three ARP requests with no reply and then FAILED. What can I conclude? Address resolution did not complete. The target may be offline or absent, but a wrong subnet/VLAN, disconnected interface or Layer 2 filtering can produce a similar result. I need more evidence before concluding why.

> [!question]- 8. What happens when I use a STALE entry? The packet goes out immediately with the cached MAC. The entry moves to DELAY, and if no proof of life arrives in 5 s it is probed.

> [!question]- 9. Why can't ARP cross a router? It sits directly on Ethernet and is a broadcast on one local link.

> [!question]- 10. What makes ARP spoofing possible? ARP has no authentication, and receivers cache any reply they get.

## Open items

- [ ] Did a PROBE happen between DELAY and REACHABLE? Check the capture, not the table.
- [ ] Is the refresh a unicast probe to `:83` or a broadcast? Check `eth.dst` in a column.
- [ ] Some repeated ARP packets looked like "source `:83`, who has .1, tell .88". That doesn't add up. Likely I mixed up the request with the reply, or read Destination instead of Source. Check with columns `eth.src`, `eth.dst`, `arp.opcode`, `arp.src.proto_ipv4`, `arp.dst.proto_ipv4`.
- [ ] Does ping count as proof of life, or only TCP? Compare `ping` vs `curl` on a STALE entry.
- [ ] What `used a/b/c` and `probes` mean in `ip -s neigh` (read `man ip-neighbour`). My guess about `probes` is unconfirmed.
- [ ] Read `ucast_solicit`.
- [ ] Verify the EtherType and IP protocol values in the table above, using my capture.

I decided not to go deeper on these now. They belong to a later pass, so they don't block Module 2.

## Benefits

**Skills gained**

- Reading a capture layer by layer and knowing which layer each field belongs to
- Telling ARP, ICMP and normal traffic apart, and recognizing the failure pattern of a dead host
- Verifying claims myself (MACs, settings, behavior) instead of trusting an explanation
- Spotting my own wrong assumptions, like the router mistake
- Using custom columns and filters to answer a question with evidence

**Why it matters for my goal**

- SOC and blue team: ARP anomalies, scans and unanswered requests are what alerts are built from
- Pentest: I know how a scanner finds local hosts and why ARP can't be blocked locally
- Troubleshooting: I can separate "no answer" from "dropped" with a capture
- Foundation for ARP spoofing, Nmap host discovery and Suricata rules later

**Interview answer**

> "ARP maps an IP to a MAC on a local link. The host broadcasts a request and the owner replies directly. The answer is cached, entries go from reachable to stale to delay, and a stale entry is used first and re-verified after. It has no authentication, which is what ARP spoofing exploits."

## Command summary

|Command|What it does|Used in|
|---|---|---|
|`ip -br a`|Short list of interfaces and their IPs|[How Wireshark works](Module%201.md#how-wireshark-works)|
|`ip link show`|Shows interface MAC addresses|[ARP](Module%201.md#arp)|
|`ip a show virbr1`|Host's MAC and IP on the lab bridge|[ARP](Module%201.md#arp)|
|`ip neigh`|Shows the ARP (neighbor) cache|[ARP cache](Module%201.md#arp-cache)|
|`ip -s neigh show dev eth0`|Same, with used/confirmed/updated timers|[Open items](Module%201.md#open-items)|
|`sudo ip neigh flush all`|Empties the cache so ARP happens again|[Experiments](Module%201.md#experiments)|
|`ping -c 3 192.0.2.1`|Sends 3 ICMP echoes|[Experiments](Module%201.md#experiments)|
|`watch -n1 'ip neigh show dev eth0'`|Watches states change every second|[ARP cache](Module%201.md#arp-cache)|
|`cat /proc/sys/net/ipv4/neigh/eth0/mcast_solicit`|ARP attempts before FAILED|[ARP cache](Module%201.md#arp-cache)|
|`cat .../base_reachable_time_ms`|Base time an entry stays REACHABLE|[ARP cache](Module%201.md#arp-cache)|
|`cat .../delay_first_probe_time`|Seconds in DELAY before probing|[ARP cache](Module%201.md#arp-cache)|
|`cat .../ucast_solicit`|Unicast probe attempts (not read yet)|[Open items](Module%201.md#open-items)|
|`man ip-neighbour`|Manual for the neighbor table fields|[Open items](Module%201.md#open-items)|

**Wireshark display filters**

> The bridge MAC is intentionally not published. Replace `<host-bridge-MAC>` in the example with the value shown by your own capture; the angle-bracketed text is a placeholder, not a literal Wireshark filter.

|Filter|What it does|Used in|
|---|---|---|
|`arp \| icmp`|ARP and ping only|[My capture, packet by packet](Module%201.md#my-capture-packet-by-packet)|
|`arp`|ARP only|[Failure signature](Module%201.md#failure-signature)|
|`arp.opcode == 1` / `== 2`|Requests only / replies only|[Open items](Module%201.md#open-items)|
|`eth.src == <host-bridge-MAC>`|Template: substitute the bridge MAC from my current local capture|[Open items](Module%201.md#open-items)|

**Wireshark habits:** Apply as Column (`eth.src`, `eth.dst`, `arp.opcode`, `arp.src.proto_ipv4`, `arp.dst.proto_ipv4`). View > Time Display Format > Seconds Since Previous Displayed Packet.

## Redo this lab from scratch

1. In Kali: `ip -br a`, and note the interface with `192.0.2.x`.
2. Start Wireshark on that interface. Display filter: `arp || icmp`.
3. In a terminal: `sudo ip neigh flush all`, then `ping -c 3 192.0.2.1`.
4. Stop the capture. Identify the ARP pair, then the 3 echo pairs.
5. Click the first ICMP packet and expand **Ethernet II**. Check that its destination MAC is the one in the ARP reply.
6. Check `ip neigh` and `ip link show` against the MACs.
7. Ping again without flushing, and confirm no ARP.
8. Ping a non-existent address and watch the `FAILED` entry.

**Evidence to attach**

- [ ] Screenshot of the `arp || icmp` capture, caption: "ARP resolves the gateway MAC before the first ping"
- [ ] Screenshot of `ip neigh` showing the `FAILED` entry
- [ ] Screenshot of the ARP packet details (opcode, sender, target)

## Next

**Planned next module — TCP and HTTP:** capture my own `curl` traffic and follow the TCP handshake, HTTP request/response, and connection teardown. This note has not been created yet.

Predictions to write before capturing:

- Total packets for one `curl` to `192.0.2.1:3000`: ____
- Flags on the first three packets: ____
- Number of TCP segments carrying the response: ____
- Which side sends FIN first: ____

## Command reference

- [Networking command reference](../../../Commands/02%20-%20Networking%20Commands.md)
- [Wireshark display-filter reference](../../../Commands/07%20-%20Wireshark%20Display%20Filters.md)

