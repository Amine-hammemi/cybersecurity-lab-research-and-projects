---
tags:
  - networking
  - virtualization
  - cybersecurity/labs
status: learning
---

# Virtual Networking for a KVM Lab

> **Public-copy note:** addresses in this note that describe my lab use documentation-only TEST-NET placeholders. They replace the real subnet and should not be used as live connection details.

## Purpose

Understand how VM NICs, bridges, subnets, gateways, DHCP, NAT and libvirt network XML work together.

## 1. VM virtual NIC

A virtual network interface card is presented to the guest OS like a network adapter. The guest configures it with an IP address and sends/receives Ethernet frames through it.

A VM NIC must be connected to some network source—for example, a libvirt virtual network. VM compute can work while networking is misconfigured.

## 2. Linux bridge

A Linux bridge is a software Layer 2 switch. It forwards Ethernet frames between its ports according to Layer 2 rules.

My host has:
- `virbr0`: bridge for libvirt's `default` network.
- `virbr1`: bridge for `isolated-lab`.

The bridge interface also has a host-side IP address in these configurations. That IP can serve as the gateway for VMs on the subnet.

## 3. Subnet and `/24`

`192.0.2.1/24` means the interface has address `192.0.2.1` with a 24-bit network prefix (netmask `255.255.255.0`).

For `192.0.2.0/24`:
- Network address: `192.0.2.0`
- Usable host range conventionally: `192.0.2.1` to `192.0.2.254`
- Broadcast: `192.0.2.255`

In this setup, `.1` is used by the host-side bridge. The DHCP pool is `.10` through `.100`.

## 4. Gateway

A gateway is the next-hop router a host uses to reach destinations outside its directly connected subnet.

In my isolated network XML, the host-side bridge has `192.0.2.1/24`. Guests may use `192.0.2.1` as their gateway. However, a gateway IP existing does **not** mean it forwards traffic to the internet. Forwarding and routing behavior must be configured separately.

For example:
- Kali `192.0.2.10` can reach a target `192.0.2.20` on the same subnet directly at Layer 2.
- For a destination outside `192.0.2.0/24`, Kali consults its routing table and may send packets to its configured gateway.
- If the network has no forwarding configured, do not expect that gateway to provide internet access.

## 5. DHCP

DHCP dynamically provides configuration such as an IP address, subnet mask, gateway and DNS servers to guests. The pool in the libvirt XML is:

```xml
<dhcp>
  <range start='192.0.2.10' end='192.0.2.100'/>
</dhcp>
```


## 6. NAT vs isolated network

### `default` network

My XML contains:

```xml
<forward mode='nat'>
  <nat>
    <port start='1024' end='65535'/>
  </nat>
</forward>
```

This configures NAT forwarding for the libvirt network. In this public copy, `198.51.100.0/24` is a documentation-only placeholder for the host's actual private subnet. With NAT configured, guests can normally make outbound connections through the host's network. Inbound reachability is not automatically equivalent to unrestricted internet exposure; firewall and port-forwarding rules matter.

### `isolated-lab` network

Its XML has no `<forward>` element. Libvirt is therefore not configured to provide NAT or routed forwarding for this network.

This is a useful starting point for a private lab segment: VMs on the same bridge can communicate, but the network does not have libvirt-configured forwarding to the outside.


## 7. Interpreting bridge state

I ran:

```bash
ip -br addr show virbr1
ip -details link show virbr1
ip route
```

Observed:
- `virbr1` had `192.0.2.1/24`.
- Detailed link output showed `UP`, `state UP`.
- The route for `192.0.2.0/24` showed `linkdown`.

## 8. Commands I used

```bash
sudo virsh net-list --all
sudo virsh net-info isolated-lab
sudo virsh net-dumpxml isolated-lab
sudo virsh net-dumpxml default
ip -br addr show virbr1
ip route
```


## 9. My network inventory

### `isolated-lab`
- State: active
- Persistent: yes
- Autostart: yes
- Bridge: `virbr1`
- Host-side address: `192.0.2.1/24`
- DHCP: `192.0.2.10–192.0.2.100`
- Forwarding element: absent

### `default`
- State: active
- Persistent: yes
- Autostart: yes
- Bridge: `virbr0`
- Host-side address: `198.51.100.1/24`
- DHCP: `198.51.100.2–198.51.100.254`
- Forwarding: NAT

### Host physical network
The Ubuntu host also has its normal physical network connection, separate from the libvirt virtual subnets. I omit the host's Wi-Fi address, router address and other home-network identifiers from this public note.

## Command reference

- [Networking command reference](../../../Commands/02%20-%20Networking%20Commands.md)
- [libvirt network command reference](../../../Commands/03%20-%20Virtualization%20%26%20Libvirt%20Commands.md)

## Related notes

- [00 - Virtualization Fundamentals](00%20-%20Virtualization%20Fundamentals.md)
- [03 - libvirt](03%20-%20libvirt.md)
- [Host Inventory & Lab Setup](../Host%20Inventory%20%26%20Lab%20Setup.md)