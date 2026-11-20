---
tags:
  - virtualization/kvm
  - linux
status: learning
---

# What is KVM 

KVM (Kernel-based Virtual Machine) is a Linux kernel virtualization facility. It uses CPU virtualization extensions (Intel VT-x or AMD-V) to run guest virtual CPUs with hardware assistance.

KVM is integrated into the Linux kernel, while a user-space component such as QEMU provides the VM process and virtual devices. KVM is often described as a Type-1 hypervisor because virtualization support lives in the kernel, but the Type-1/Type-2 label is an imperfect simplification for a Linux host that also runs a general-purpose operating system.

KVM is not the graphical manager, not the virtual network, and not by itself the full VM management workflow. QEMU commonly works with KVM to provide a usable VM.

## How KVM fits with QEMU

- KVM provides the kernel facilities for hardware-assisted guest execution.
- QEMU provides the VM process and virtual hardware/device models.
- libvirt manages the VM definition and lifecycle.
- virt-manager is a graphical client for libvirt.

A simplified chain is: `virt-manager → libvirt → QEMU + KVM → Linux kernel / CPU`.

## What I checked

### 1. CPU virtualization flags

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

My result: `24`.

- `vmx` is the Intel VT-x flag.
- `svm` is the AMD-V flag.
- This command counts matching processor entries in `/proc/cpuinfo`; `24` does **not** necessarily mean 24 physical CPU cores.
- The result suggests virtualization flags are exposed to Linux. It is not the strongest check by itself.

### 2. KVM executable

```bash
command -v kvm
```

Result: `/usr/bin/kvm`.

This confirms a command at that path is available. It is not alone proof that acceleration works.

### 3. Kernel modules

```bash
lsmod | grep kvm
```

Observed:

```text
kvm_intel  540672  0
kvm       1507328  1 kvm_intel
irqbypass   16384  1 kvm
```

- `kvm_intel`: Intel-specific KVM support.
- `kvm`: common KVM kernel module.
- `irqbypass`: kernel support used to improve certain interrupt paths in virtualized workloads.

### 4. Device node and permissions

```bash
ls -l /dev/kvm
```

Observed:

```text
crw-rw----+ 1 root kvm ... /dev/kvm
```

- `c`: character device.
- Owner: `root`.
- Group: `kvm`.
- `rw` for owner and group: read/write permissions.
- `+`: additional ACL information is present.
- `/dev/kvm` is an interface through which userspace virtualization software accesses KVM.

Do **not** use `chmod 777 /dev/kvm` as a fix. Diagnose group membership, ACLs and session state instead.

### 5. Functional check

```bash
kvm-ok
```

Output:

```text
INFO: /dev/kvm exists
KVM acceleration can be used
```

This is the strongest confirmation I collected that KVM acceleration is available on this host.

### 6. User group membership

```bash
id
```

My account belongs to the `kvm` and `libvirt` groups, among others. This matches the expected access configuration for this setup.

## My current result

KVM acceleration is working. I did not need to change `/dev/kvm` permissions.

## Command reference

- [KVM and host validation commands](../../../Commands/03%20-%20Virtualization%20%26%20Libvirt%20Commands.md)

## Related notes

- [00 - Virtualization Fundamentals](00%20-%20Virtualization%20Fundamentals.md)
- [02 - QEMU](02%20-%20QEMU.md)
- [Host Inventory & Lab Setup](../Host%20Inventory%20%26%20Lab%20Setup.md)