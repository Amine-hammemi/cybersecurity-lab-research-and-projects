---
tags:
  - cybersecurity/reference
  - command-reference
status: maintained
---

# Command Reference Library

A central index of the command-line tools, PowerShell cmdlets, Windows utilities and Wireshark display filters that appear in my cybersecurity learning notes.

## How I use this library

- Each command is documented once in the most relevant category.
- **What it does** explains the immediate behaviour of the command.
- **General use and options** gives reusable patterns beyond the exact lab example.
- **Where I used it** links back to the specific note and section where the command appears in my learning record.
- Links point to documented examples. They do not automatically mean that every option or generic template has been independently practised.
- Replace placeholders such as `SERVICE`, `VM_NAME`, `<PID>` or `/path/to/file` with values from the environment being inspected.

## Categories

| Reference | Scope |
|---|---|
| [01 - Linux Commands](01%20-%20Linux%20Commands.md) | Linux process, service, log, file, shell and administration commands |
| [02 - Networking Commands](02%20-%20Networking%20Commands.md) | Linux and Windows network inspection, connectivity, routes and neighbour-cache commands |
| [03 - Virtualization & Libvirt Commands](03%20-%20Virtualization%20%26%20Libvirt%20Commands.md) | KVM readiness, QEMU, libvirt, `virsh` and VM management |
| [04 - PowerShell Commands](04%20-%20PowerShell%20Commands.md) | PowerShell cmdlets for Windows services, event logs, processes and network configuration |
| [05 - Windows & Sysmon Utilities](05%20-%20Windows%20%26%20Sysmon%20Utilities.md) | Windows built-in utilities and Sysmon command-line options |
| [06 - Wazuh Operations Commands](06%20-%20Wazuh%20Operations%20Commands.md) | Wazuh Manager service checks and agent-control CLI |
| [07 - Wireshark Display Filters](07%20-%20Wireshark%20Display%20Filters.md) | Display filters and protocol fields used during packet analysis |

## Conventions and safety

- Most Linux commands are shown as they appeared in my notes. Add `sudo` only when the operation requires elevated privileges.
- Inspection commands are generally safe, but commands such as `ufw allow`, `ufw deny`, `ip neigh flush` and `sysmon.exe -c` change system state. Understand the effect before running them.
- A command shown with a lab IP address is an example from my private virtual network, not a permanent address or a public target.
- I do not store passwords, tokens, private keys or enrollment secrets in this reference.

## Reverse navigation

The `Where I used it` column in each category returns to the exact learning note section. The learning notes also link back to the matching command category near their related-notes section.
