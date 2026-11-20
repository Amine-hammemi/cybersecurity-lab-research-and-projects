---
tags:
  - cybersecurity/reference
  - wazuh
  - siem
  - command-reference
status: maintained
---

# Wazuh Operations Commands

A compact reference for the Wazuh-specific commands I have recorded while validating the server components and checking agent connectivity.

## Wazuh Manager, Indexer and Dashboard services

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `sudo systemctl status wazuh-manager` | Checks whether the Wazuh Manager service is running and shows recent status details. | Use `start`, `stop` or `restart` only when a service action is intended. Review logs before restarting a component. | [Wazuh SOC Lab — Manager validation](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md#9-validation-checklist-when-restarting-the-lab) · [Wazuh Architecture — Manager status](../Cybersecurity/04-Security%20Monitoring/Wazuh%20Architecture.md#7-what-happens-if-an-agent-disconnects) |
| `sudo systemctl status wazuh-indexer` | Checks the Wazuh Indexer service state. | The Indexer is responsible for storing/searching indexed data; check it separately from the Manager and Dashboard. | [Wazuh SOC Lab — Indexer validation](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md#9-validation-checklist-when-restarting-the-lab) |
| `sudo systemctl status wazuh-dashboard` | Checks the Wazuh Dashboard service state. | A working Dashboard does not, by itself, prove that the Manager is receiving agent telemetry or that the Indexer is healthy. | [Wazuh SOC Lab — Dashboard validation](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md#9-validation-checklist-when-restarting-the-lab) |

## Agent inventory and connectivity

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `sudo /var/ossec/bin/agent_control -lc` | Lists configured Wazuh agents and their connection status. | Useful for checking enrollment and recent connectivity from the Manager. It is a status snapshot, not a full endpoint health assessment. | [Wazuh Architecture — list agent status](../Cybersecurity/04-Security%20Monitoring/Wazuh%20Architecture.md#7-what-happens-if-an-agent-disconnects) |

## Windows-side companion checks

The Windows agent and Sysmon services are checked from PowerShell:

- [`Get-Service` and `Get-CimInstance` examples](04%20-%20PowerShell%20Commands.md#windows-services-and-service-to-process-mapping)
- [Wazuh SOC Lab — Windows agent and Sysmon checks](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md#9-validation-checklist-when-restarting-the-lab)

## Related notes

- [Wazuh Architecture](../Cybersecurity/04-Security%20Monitoring/Wazuh%20Architecture.md)
- [Wazuh SOC Lab](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md)
- [Linux Administration](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md)
