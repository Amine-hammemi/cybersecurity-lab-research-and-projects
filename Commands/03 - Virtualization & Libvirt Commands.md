---
tags:
  - cybersecurity/reference
  - virtualization
  - libvirt
  - command-reference
status: maintained
---

# Virtualization & Libvirt Commands

Commands I documented while validating KVM, QEMU, libvirt, VM definitions and virtual network attachments on my Ubuntu host.

## KVM readiness and host capabilities

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `egrep -c '(vmx\|svm)' /proc/cpuinfo` | Counts processor entries whose flags include Intel VT-x (`vmx`) or AMD-V (`svm`). | This counts matching logical-processor entries, not physical cores. `grep -Ec` is the modern equivalent form. | [Host inventory — CPU virtualization check](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#virtualization-readiness-checks-i-performed) · [KVM — CPU virtualization flags](../Cybersecurity/01-Infrastructure/Virtualization/01%20-%20KVM.md#1-cpu-virtualization-flags) |
| `command -v kvm` | Locates the `kvm` executable using the current `PATH`. | Useful for checking whether a command is available and which executable would run. It does not establish that hardware acceleration is functional by itself. | [Host inventory — KVM executable check](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#virtualization-readiness-checks-i-performed) · [KVM — executable check](../Cybersecurity/01-Infrastructure/Virtualization/01%20-%20KVM.md#2-kvm-executable) |
| `lsmod \| grep kvm` | Shows loaded kernel modules and filters for KVM-related names. | `lsmod` lists modules currently loaded; `grep` filters its output. A module being loaded is one readiness signal, not a complete VM test. | [Host inventory — KVM modules](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#virtualization-readiness-checks-i-performed) · [KVM — kernel modules](../Cybersecurity/01-Infrastructure/Virtualization/01%20-%20KVM.md#3-kernel-modules) |
| `ls -l /dev/kvm` | Displays the KVM device node and its ownership/permissions. | Check the `kvm` group and ACLs if access fails. Do not solve permission problems with `chmod 777`. | [Host inventory — KVM device check](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#virtualization-readiness-checks-i-performed) · [KVM — device permissions](../Cybersecurity/01-Infrastructure/Virtualization/01%20-%20KVM.md#4-device-node-and-permissions) |
| `kvm-ok` | Checks whether KVM acceleration can be used on the host. | A useful functional readiness test when the `cpu-checker` utility is installed. Read the diagnostic output instead of assuming package presence alone proves acceleration. | [Host inventory — KVM functional check](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#virtualization-readiness-checks-i-performed) · [KVM — functional check](../Cybersecurity/01-Infrastructure/Virtualization/01%20-%20KVM.md#5-functional-check) |
| `id` | Displays the current account's group membership, including virtualization-related groups. | See [Linux Commands — `id`](01%20-%20Linux%20Commands.md#process-and-resource-inspection). | [KVM — group membership](../Cybersecurity/01-Infrastructure/Virtualization/01%20-%20KVM.md#6-user-group-membership) |

## QEMU and virt-manager

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `command -v qemu-system-x86_64` | Locates the QEMU x86-64 system emulator executable. | Confirms that the executable is discoverable in `PATH`; inspect its version and available features separately. | [Host inventory — QEMU executable check](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#virtualization-readiness-checks-i-performed) · [QEMU — executable validation](../Cybersecurity/01-Infrastructure/Virtualization/02%20-%20QEMU.md#what-i-verified) |
| `qemu-system-x86_64 --version` | Prints the QEMU system emulator version. | Useful for recording the installed version before reproducing a lab configuration. | [QEMU — version check](../Cybersecurity/01-Infrastructure/Virtualization/02%20-%20QEMU.md#useful-commands) |
| `virt-manager` | Launches the Virtual Machine Manager graphical interface. | Useful for inspecting VM hardware, storage, consoles and virtual networks. It is a management client; QEMU performs the VM process/device work. | [virt-manager — launch check](../Cybersecurity/01-Infrastructure/Virtualization/04%20-%20virt-manager.md#what-i-verified) · [Host inventory — GUI validation](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#virtualization-readiness-checks-i-performed) |

## libvirt service and connection checks

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `systemctl is-active libvirtd` | Checks the runtime state of the legacy libvirt daemon. | The result can be `active`, `inactive` or another state. Modern libvirt can also use modular daemons; interpret service state in the context of the installed architecture. | [Host inventory — libvirtd check](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#virtualization-readiness-checks-i-performed) · [libvirt — service state](../Cybersecurity/01-Infrastructure/Virtualization/03%20-%20libvirt.md#my-connection-and-service-checks) |
| `systemctl list-units --type=service --all 'virt*' 'libvirt*'` | Lists libvirt- and virtualization-related systemd service units. | Useful when checking legacy and modular daemons. See [Linux Commands — systemd](01%20-%20Linux%20Commands.md#services-and-systemd). | [libvirt — daemon inventory](../Cybersecurity/01-Infrastructure/Virtualization/03%20-%20libvirt.md#my-connection-and-service-checks) |
| `virsh -c qemu:///system list --all` | Lists all VMs known to the system libvirt connection, including shut-off VMs. | `-c qemu:///system` explicitly selects the system connection. `list` without `--all` usually shows only running guests. | [Host inventory — VM inventory](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) · [libvirt — connection check](../Cybersecurity/01-Infrastructure/Virtualization/03%20-%20libvirt.md#my-connection-and-service-checks) · [QEMU — distinction from QEMU CLI](../Cybersecurity/01-Infrastructure/Virtualization/02%20-%20QEMU.md#useful-commands) |
| `virsh uri` | Prints the URI of the current libvirt connection. | Use it to confirm whether `virsh` is connected to `qemu:///system` or another libvirt URI. | [libvirt — connection URI](../Cybersecurity/01-Infrastructure/Virtualization/03%20-%20libvirt.md#my-connection-and-service-checks) |

## VM lifecycle and virtual NIC inventory

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `virsh domiflist VM_NAME` | Lists the virtual network interfaces configured for a VM. | Run once per VM. The `Source` column identifies the network/bridge source. It shows configured interfaces; it does not alone prove live packet connectivity. | [Host inventory — NIC attachment validation](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) · [Host inventory — VM network troubleshooting](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#7-useful-troubleshooting-commands) |
| `virsh start VM_NAME` | Starts a virtual machine through libvirt. | Use the exact domain name shown by `virsh list --all`. Check available RAM and other host resources before starting multiple VMs. | [virt-manager — GUI vs CLI](../Cybersecurity/01-Infrastructure/Virtualization/04%20-%20virt-manager.md#gui-vs-cli) |

## Virtual network inspection

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `virsh net-list --all` | Lists libvirt virtual networks, including inactive definitions. | Use `--all` to include inactive networks; without it, only active networks are normally shown. | [Host inventory — network list](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) · [Virtual Networking — commands](../Cybersecurity/01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md#8-commands-i-used) |
| `virsh net-info isolated-lab` | Displays the state, persistence, autostart and bridge for a libvirt network. | Replace `isolated-lab` with another network name. Active and persistent are separate properties. | [Host inventory — network status](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) · [Virtual Networking — network status](../Cybersecurity/01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md#8-commands-i-used) |
| `virsh net-dumpxml isolated-lab` | Prints the XML definition of the selected libvirt network. | Inspect `<bridge>`, `<ip>`, `<dhcp>` and any `<forward>` element. XML describes libvirt configuration; also inspect host routes/firewall when verifying effective isolation. | [Host inventory — isolated network XML](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) · [Virtual Networking — XML inspection](../Cybersecurity/01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md#8-commands-i-used) · [libvirt — network XML](../Cybersecurity/01-Infrastructure/Virtualization/03%20-%20libvirt.md#what-is-a-libvirt-xml-file) |
| `virsh net-dumpxml default` | Prints the XML definition of the host's libvirt `default` network. | Useful for understanding whether a network is configured with NAT forwarding. It does not imply that every VM is attached to it. | [Virtual Networking — compare default and isolated networks](../Cybersecurity/01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md#8-commands-i-used) · [libvirt — network XML examples](../Cybersecurity/01-Infrastructure/Virtualization/03%20-%20libvirt.md#what-is-a-libvirt-xml-file) |

## Related notes

- [Host Inventory & Lab Setup](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md)
- [KVM](../Cybersecurity/01-Infrastructure/Virtualization/01%20-%20KVM.md)
- [QEMU](../Cybersecurity/01-Infrastructure/Virtualization/02%20-%20QEMU.md)
- [libvirt](../Cybersecurity/01-Infrastructure/Virtualization/03%20-%20libvirt.md)
- [virt-manager](../Cybersecurity/01-Infrastructure/Virtualization/04%20-%20virt-manager.md)
- [Virtual Networking](../Cybersecurity/01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md)
