---
tags:
  - virtualization/libvirt
  - linux
status: learning
---

# libvirt

> **Public-copy note:** network addresses in the XML example are documentation-only placeholders, not the live lab configuration.

## What it is

**libvirt** is a management API and set of tools for virtualization. On this host it manages QEMU/KVM virtual machines and virtual networks.

It provides a consistent management interface for operations such as defining, starting, stopping and inspecting VMs and networks.

## Important pieces

- **`libvirtd`:** the legacy monolithic libvirt daemon. My system currently uses this service.
- **Modular daemons:** newer libvirt installations can split functionality into services such as `virtqemud` and `virtnetworkd`. Their being inactive is not necessarily an error when the legacy daemon is being used.
- **`virsh`:** command-line client for libvirt.
- **XML definitions:** declarative descriptions of VM or virtual-network configuration.
- **Connection URI:** selects which libvirt connection is being managed.

## My connection and service checks

```bash
systemctl is-active libvirtd
```

The first check returned `inactive`; a later check returned `active`. The later service listing confirmed `libvirtd.service` was `active running`. Service state can change between checks, so recheck current state instead of assuming an earlier result remains true.

```bash
systemctl list-units --type=service --all 'virt*' 'libvirt*'
```

Observed services included:
- `libvirtd.service`: active/running
- `virtlockd.service`: active/running
- `virtlogd.service`: active/running
- `libvirt-guests.service`: active/exited

The modular `virtqemud` and `virtnetworkd` units I checked were inactive. This is consistent with the legacy daemon configuration observed.

```bash
virsh -c qemu:///system list --all
virsh uri
```

The URI was `qemu:///system`. The VM list was empty during this initial infrastructure check, before I created the Kali, Wazuh and Windows VMs. The current VM inventory is documented in [Host Inventory & Lab Setup](../Host%20Inventory%20%26%20Lab%20Setup.md).

## What is a libvirt XML file?

XML is a structured configuration format. libvirt uses XML definitions for objects such as:
- Virtual machines (CPU, memory, disks, NICs, boot settings, etc.).
- Virtual networks (bridge, IP address, DHCP range, forwarding mode, etc.).

For example, the network XML I inspected includes:

```xml
<network>
  <name>isolated-lab</name>
  <bridge name='virbr1' stp='on' delay='0'/>
  <ip address='192.0.2.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.0.2.10' end='192.0.2.100'/>
    </dhcp>
  </ip>
</network>
```

This is a *definition*, not the network traffic itself. libvirt uses the definition to configure the Linux bridge and associated network services.

Useful commands:

```bash
sudo virsh net-list --all
sudo virsh net-info isolated-lab
sudo virsh net-dumpxml isolated-lab
sudo virsh net-dumpxml default
```

- `net-list --all`: list known networks, including inactive ones.
- `net-info`: report state and persistence details.
- `net-dumpxml`: print the current libvirt XML definition.
- `--all` for VM listing also includes defined but stopped VMs.

A network can be **active** (currently started) and **persistent** (definition saved across daemon/host restarts). `Autostart: yes` means libvirt is configured to start it automatically.

## My current network inventory

| Name | Bridge | Subnet | Forwarding |
|---|---|---|---|
| `default` | `virbr0` | `198.51.100.0/24` | NAT |
| `isolated-lab` | `virbr1` | `192.0.2.0/24` | No `<forward>` element |

Both networks were reported active, persistent and configured for autostart in the captured inventory. The `default` network remains defined on the host, but the three lab VMs are not attached to it in the current configuration. See [Host Inventory & Lab Setup](../Host%20Inventory%20%26%20Lab%20Setup.md).

## Command reference

- [libvirt command reference](../../../Commands/03%20-%20Virtualization%20%26%20Libvirt%20Commands.md)

## Related notes

- [00 - Virtualization Fundamentals](00%20-%20Virtualization%20Fundamentals.md)
- [04 - virt-manager](04%20-%20virt-manager.md)
- [05 - Virtual Networking](05%20-%20Virtual%20Networking.md)
- [Host Inventory & Lab Setup](../Host%20Inventory%20%26%20Lab%20Setup.md)