---
tags:
  - virtualization/qemu
  - linux
status: learning
---

# QEMU

## What it is

**QEMU** is a user-space machine emulator and virtualizer. It creates a VM process and models virtual hardware such as storage controllers, network cards, display devices and machine types.

When configured to use KVM on a Linux host, QEMU delegates guest CPU execution to KVM for hardware acceleration.

## Relationship to KVM

- QEMU: VM process and virtual devices.
- KVM: kernel facility for hardware-assisted guest CPU execution.
- libvirt: higher-level management of QEMU VMs and networks.
- virt-manager: graphical management client.

Without KVM acceleration, QEMU can use software emulation for supported guest architectures, often with much lower performance. Exact behavior depends on machine and accelerator configuration.

## What I verified

```bash
command -v qemu-system-x86_64
```

Result:

```text
/usr/bin/qemu-system-x86_64
```

This confirms the QEMU x86-64 system-emulation executable is available.

Package check:

```text
qemu-kvm: not installed
libvirt-daemon-system: installed
libvirt-clients: installed
virt-manager: installed
p7zip-full: installed
```

**Important:** `qemu-kvm` being reported as not installed by that exact package name does not mean QEMU or KVM is missing. The executable exists, `kvm-ok` succeeds, and the system is configured to use KVM. Inspect actual binaries and capabilities before installing more packages.

## Useful commands

```bash
command -v qemu-system-x86_64
qemu-system-x86_64 --version
```

`virsh list --all` is a libvirt command, not a QEMU command; it shows VMs known to the selected libvirt connection.

## Security and lab notes

- Download VM images from the official project source and verify checksums/signatures where provided.
- Treat guest images as software: keep a clean baseline and use snapshots where appropriate.
- A VM can have multiple virtual NICs. QEMU running successfully does not prove the VM is isolated; inspect its NICs and network attachments.

## Command reference

- [QEMU and libvirt command reference](../../../Commands/03%20-%20Virtualization%20%26%20Libvirt%20Commands.md)

## Related notes

- [00 - Virtualization Fundamentals](00%20-%20Virtualization%20Fundamentals.md)
- [01 - KVM](01%20-%20KVM.md)
- [03 - libvirt](03%20-%20libvirt.md)
- [Host Inventory & Lab Setup](../Host%20Inventory%20%26%20Lab%20Setup.md)