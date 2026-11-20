---
tags:
  - cybersecurity/reference
  - networking
  - command-reference
status: maintained
---

# Networking Commands

Commands I have used or documented to inspect interfaces, addresses, routes, neighbour discovery and connectivity in my virtual lab.

## Linux interface and address inspection (`ip`)

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `ip -br addr` / `ip -br address` | Displays interface names, state and assigned addresses in a compact format. | `-br` means brief; `ip -br a` is a shorter alias. Add `show DEV` to inspect one interface. | [Host inventory — guest interface checks](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) · [Wireshark — command summary](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |
| `ip -br addr show virbr1` | Shows the host-side address and state of the `virbr1` bridge in brief format. | Replace `virbr1` with the interface name to inspect. Run on the host, not inside a guest unless the same interface exists there. | [Host inventory — bridge validation](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) · [Virtual Networking — bridge state](../Cybersecurity/01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md#7-interpreting-bridge-state) |
| `ip link show` / `ip link show virbr1` | Shows Layer 2 interface properties and administrative/link state. | Use `ip -details link show DEV` for more detail. `ip link set` can change interface state, so do not use a modifying form without understanding the effect. | [Wireshark — interface checks](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) · [Virtual Networking — detailed link state](../Cybersecurity/01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md#7-interpreting-bridge-state) |
| `ip a show virbr1` | Displays address information for a specific interface. | `ip a` is shorthand for `ip address`; `ip address show DEV` is the longer form. | [Wireshark — host bridge inspection](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |
| `ip route` | Displays the kernel IPv4 routing table. | `ip -6 route` inspects IPv6 routes; `ip route get DESTINATION` asks the kernel which route it would use for a destination. | [Host inventory — host routes](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) · [Virtual Networking — route interpretation](../Cybersecurity/01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md#7-interpreting-bridge-state) · [Wireshark — route check](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |

## Neighbour discovery and ARP cache (`ip neigh`)

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `ip neigh` | Displays the kernel neighbour table, which maps IP addresses to link-layer addresses on local links. | `ip neigh show dev DEV` filters by interface; states can include `REACHABLE`, `STALE`, `DELAY`, `PROBE` and `FAILED`. | [Wireshark — ARP cache states](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#arp-cache) · [Wireshark — neighbour inspection](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |
| `ip -s neigh show dev eth0` | Displays neighbour entries for `eth0` with statistics. | `-s` adds statistics. Substitute the actual interface name from `ip -br addr`. | [Wireshark — ARP experiments](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#experiments) · [Wireshark — neighbour statistics](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |
| `sudo ip neigh flush all` | Flushes cached neighbour entries so the kernel must resolve them again when needed. | This changes local cache state and may briefly cause new ARP/ND resolution. Use a device-specific selector where appropriate, e.g. `dev eth0`; don't flush caches on a production system casually. | [Wireshark — force fresh ARP resolution](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#experiments) · [Wireshark — neighbour-cache reset](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |

## Connectivity checks

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `ping HOST` | Sends ICMP Echo Requests to test IP reachability and round-trip response. | `-c N` limits the number of requests on Linux; a successful ping does not prove that a specific TCP/UDP service is reachable. | [Wireshark — ICMP and ping](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#icmp-and-ping) · [Wireshark — connectivity experiment](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#experiments) |
| `ping -c 3 192.0.2.1` | Sends three ICMP Echo Requests to the host-side bridge address in the lab subnet. | Replace the address with the intended target. Interpret failure alongside routing, ARP and firewall evidence; ping can be filtered. | [Wireshark — test reachability to the lab host](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#experiments) · [Wireshark — repeatable ARP/ICMP test](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |
| `curl URL` | Makes an HTTP(S) request and displays the response by default. | `-I` requests headers only, `-v` shows connection details, and `-o FILE` writes the response body to a file. The note mentions `curl` as a test tool but does not preserve a complete URL invocation. | [Wireshark — application-level connectivity experiment](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#experiments) |

## Windows network configuration

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `ipconfig` | Displays Windows IP configuration for network adapters. | `ipconfig /all` includes more details; `ipconfig /release` and `/renew` change DHCP lease state, so use them intentionally. | [Host inventory — Windows guest IP check](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) |
| `Get-NetIPConfiguration` | Returns Windows interface, IP address, gateway and DNS configuration as PowerShell objects. | Pipe the result to `Format-List` for expanded output or select specific properties. | [Host inventory — PowerShell network check](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) |

For the PowerShell object-based equivalent, see [`Get-NetIPConfiguration`](04%20-%20PowerShell%20Commands.md#network-configuration).

## Neighbour-resolution parameter inspection

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `cat /proc/sys/net/ipv4/neigh/eth0/mcast_solicit` | Reads a kernel neighbour-discovery parameter for the selected interface. | The Wireshark note also inspected `base_reachable_time_ms`, `delay_first_probe_time` and `ucast_solicit`. These are runtime kernel parameters; reading them does not change configuration. | [Wireshark — neighbour-cache parameter checks](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#experiments) |

## Related notes

- [Host Inventory & Lab Setup](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md)
- [Virtual Networking](../Cybersecurity/01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md)
- [Wireshark Module 1](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md)
- [07 - Wireshark Display Filters](07%20-%20Wireshark%20Display%20Filters.md)
- [04 - PowerShell Commands](04%20-%20PowerShell%20Commands.md)
