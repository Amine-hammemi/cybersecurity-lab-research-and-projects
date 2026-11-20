# cybersecurity-lab-research-and-projects

A practical record of my cybersecurity learning, lab engineering, technical experiments, and security projects.

I use this repository to document what I study and build, how I validate it, what the evidence actually demonstrates, and what I still need to practise. The goal is to keep the notes useful to another learner while being honest about my current level of experience.

> **How I report progress:** configured does not automatically mean validated, and validated does not automatically mean independently mastered. I keep those distinctions visible throughout the notes.

## What you'll find here

| Area | Focus |
|---|---|
| Infrastructure | Linux virtualization, KVM/QEMU, libvirt, VM design and virtual networking |
| Linux administration | Processes, systemd, sockets, firewall basics and troubleshooting |
| Endpoint security | Windows internals, Windows Event Logs and Sysmon |
| Security monitoring | SIEM fundamentals, Wazuh architecture and endpoint telemetry |
| Network analysis | Packet captures, Ethernet, ARP, ICMP and Wireshark |
| Command reference | Practical Linux, networking, virtualization, PowerShell, Windows, Wazuh and Wireshark references |
| Projects | Hands-on work with objectives, implementation notes, validation evidence and remaining tasks |
| CTF and practice | Notes from OverTheWire Bandit and other practical exercises |

## Repository structure

```text
.
├── README.md
├── Commands/
├── Cybersecurity/
│   ├── 01-Infrastructure/
│   ├── 02-Linux Administration/
│   ├── 03-Endpoint Security/
│   ├── 04-Security Monitoring/
│   └── 05-Network Analysis/
├── Projects/
│   └── Project 01 - Windows Endpoint Detection & Investigation/
├── CTF & Practice/
├── Evidence/
├── Assets/
└── Portfolio Evidence Plan.md
```

## Current lab at a glance

The lab uses an Ubuntu host with KVM/QEMU and libvirt. Its virtual machines include:

- **Kali Linux** — testing and network-analysis workstation.
- **Windows 11** — monitored endpoint with Sysmon and the Wazuh Agent.
- **Wazuh** — Manager, Indexer and Dashboard.

The VMs are attached to a dedicated libvirt network. The host is also connected to the virtual bridge, so I do not describe the network as an air gap or as isolated from the host. The host and guest firewall, routing, services and hypervisor configuration still affect the actual security boundary.

The public notes use documentation-only IP addresses from reserved example ranges where an address is needed. They are not the live addresses of my lab. Real MAC addresses, home-network details, local usernames and host-specific firewall exports are intentionally excluded.

For the lab design and validation commands, start with [Host Inventory & Lab Setup](Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md). For the monitoring architecture, see [Wazuh SOC Lab](Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md).

## Featured project

### [Project 01 — Windows Endpoint Detection & Investigation](Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/README.md)

The project uses Windows Event Logs, Sysmon, PowerShell Script Block Logging and Wazuh to build a small endpoint monitoring and investigation workflow.

**Current stage:** telemetry foundation / pre-detection.

Sysmon and PowerShell provider events have been observed in Wazuh. The exact controlled test marker has not yet been confirmed in a Wazuh event document. Custom detection engineering, a complete controlled scenario, event correlation and the final investigation report remain open.

## How the notes are organised

Where useful, a technical note follows this sequence:

1. **Concept** — what the technology or mechanism does.
2. **Implementation** — how I configured or used it.
3. **Validation** — what I tested and observed.
4. **Interpretation** — what the evidence demonstrates, and what it does not.
5. **Next practice** — what I still need to repeat or investigate independently.

The [Command Reference Library](Commands/00%20-%20Command%20Library%20Index.md) collects commands used across the notes. The [Portfolio Evidence Plan](Portfolio%20Evidence%20Plan.md) describes the screenshots and supporting material I intend to add after reproducing each exercise.

## Evidence and repository hygiene

- Evidence should come from real lab activity. I do not fabricate screenshots or tool output.
- Screenshots and exports must be reviewed for credentials, tokens, personal information, host identifiers and network details before publication.
- Private keys, passwords, API keys, enrollment secrets, `.env` files, VM disk images, packet captures and raw event exports do not belong in this public repository.
- Historical output should be labelled with its date and environment.
- An event or alert is an investigation lead, not proof of compromise.
- I document limitations and incomplete validation rather than presenting them as completed work.

The `Evidence/` folder currently contains only its README. I will add genuine, reviewed evidence as the projects progress.

## Scope

This is a personal learning repository and a work in progress. It is not a production SOC deployment, a formal security audit, or a claim of professional operational experience. The notes reflect my current learning and the work I have actually completed.
