---
tags:
  - virtualization/virt-manager
  - linux
status: learning
---

# virt-manager

## What it is

**Virtual Machine Manager (`virt-manager`)** is a graphical application for managing virtual machines and their virtual hardware through libvirt.

It is a client, not the hypervisor itself. It normally communicates with a libvirt connection such as `qemu:///system`.

## What I can do with it

- View VM state and open a guest console.
- Create and configure virtual machines.
- Configure virtual CPUs, RAM, disks and virtual NICs.
- Attach a VM NIC to a libvirt network.
- Inspect and manage virtual networks.

## What I verified

I ran:

```bash
virt-manager
```

The application opened and displayed the QEMU/KVM connection.

## GUI vs CLI

The GUI and CLI are two ways to manage the same libvirt environment.

| Task | GUI | CLI |
|---|---|---|
| Inspect VMs | VM list | `virsh list --all` |
| Inspect networks | Connection/network details | `virsh net-list --all` |
| Inspect network XML | Network details/XML view (where available) | `virsh net-dumpxml isolated-lab` |
| Start a VM | GUI controls | `virsh start VM_NAME` |


## Command reference

- [VM management command reference](../../../Commands/03%20-%20Virtualization%20%26%20Libvirt%20Commands.md)

## Related notes

- [00 - Virtualization Fundamentals](00%20-%20Virtualization%20Fundamentals.md)
- [03 - libvirt](03%20-%20libvirt.md)
- [05 - Virtual Networking](05%20-%20Virtual%20Networking.md)
- [Host Inventory & Lab Setup](../Host%20Inventory%20%26%20Lab%20Setup.md)