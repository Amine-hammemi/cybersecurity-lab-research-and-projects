---
title: Project 01 - Windows Endpoint Detection & Investigation
type: project
status: in-progress
---

# Project 01 - Windows Endpoint Detection & Investigation

## Project goal

Build and validate a small SOC-style detection and investigation workflow using a Windows 11 endpoint, Sysmon, PowerShell logging, and Wazuh.

The objective is to learn how to:
- Generate controlled endpoint activity.
- Collect useful Windows telemetry.
- Forward endpoint events to Wazuh.
- Search and inspect events.
- Correlate process execution with script activity.
- Develop and validate detection logic.
- Investigate an alert using evidence and document a defensible conclusion.

This project is not complete yet. The current work has established the endpoint telemetry foundation. Detection engineering, a complete attack simulation, correlation, investigation, and the final incident report remain.

## Lab components

| Component | Role |
|---|---|
| Windows 11 VM (`win11`) | Monitored endpoint |
| Sysmon | Detailed Windows system activity telemetry |
| Wazuh Agent | Collects configured Windows event channels and forwards events |
| Wazuh Manager | Receives events, decodes them, evaluates rules |
| Wazuh Indexer | Stores searchable data |
| Wazuh Dashboard | Search, alert review, and investigation interface |
| Kali Linux VM | Available for later controlled testing; not required for the initial local baseline |

## Current lab network

The VMs use a dedicated libvirt network. In this public copy, its subnet is represented by the documentation-only placeholder `192.0.2.0/24`; it is not the live lab subnet. Kali, Wazuh, and Windows 11 each have one virtual NIC connected to the dedicated network. The inspected libvirt definition had no `<forward>` element.

The host is connected to this virtual network, so do not describe this as isolation from the host. Do not publish real MAC addresses, home-network details, credentials, or private host information.

## Notes in this folder

- [01 - Project Scope and Architecture](01%20-%20Project%20Scope%20and%20Architecture.md)
- [02 - Windows Telemetry Sources](02%20-%20Windows%20Telemetry%20Sources.md)
- [03 - PowerShell Logging and Wazuh Collection](03%20-%20PowerShell%20Logging%20and%20Wazuh%20Collection.md)
- [04 - Validation Log](04%20-%20Validation%20Log.md)
- [05 - Investigation Methodology and Learning Plan](05%20-%20Investigation%20Methodology%20and%20Learning%20Plan.md)
- [06 - Project Status and Next Milestones](06%20-%20Project%20Status%20and%20Next%20Milestones.md)

## Completion criteria

The project can be considered complete when I can demonstrate and document:
1. A controlled, repeatable endpoint activity scenario.
2. Relevant Windows and Sysmon telemetry.
3. Successful Wazuh collection and searchable evidence.
4. A detection rule with a clear rationale and test results.
5. An investigation that correlates events into a timeline.
6. A reasoned classification supported by evidence.
7. A concise incident report and sanitized public portfolio documentation.

## Evidence discipline

I distinguish between:
- **Configured:** a setting or component has been installed or configured.
- **Locally validated:** the endpoint itself generated the expected telemetry.
- **Collection validated:** the event is visible in Wazuh.
- **Detection validated:** a rule generated the expected alert.
- **Investigation validated:** I can explain the event sequence and support a conclusion with evidence.

I do not claim a later stage merely because an earlier stage succeeded.
