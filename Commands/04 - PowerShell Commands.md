---
tags:
  - cybersecurity/reference
  - windows
  - powershell
  - command-reference
status: maintained
---

# PowerShell Commands

PowerShell cmdlets I have documented for Windows service inspection, event-log queries and endpoint troubleshooting. Cmdlets return structured .NET objects, so I can filter and select properties with the pipeline.

## Windows services and service-to-process mapping

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `Get-Service` | Lists services and their current status. | `Get-Service -Name NAME` filters to a service; `Where-Object Status -eq 'Running'` filters results in the pipeline. | [Windows Internals — services](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md#4-windows-services) |
| `Get-Service -Name WazuhSvc` | Checks the Windows Wazuh Agent service. | Use the service name rather than its display name where known. `Get-Service` reports service state, not whether the agent is successfully communicating with the Manager. | [Windows Internals — Wazuh service](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md#4-windows-services) · [Wazuh Architecture — agent troubleshooting](../Cybersecurity/04-Security%20Monitoring/Wazuh%20Architecture.md#7-what-happens-if-an-agent-disconnects) |
| `Get-CimInstance Win32_Service` | Retrieves Windows service objects through CIM/WMI. | Use `-Filter` to narrow results server-side, then `Select-Object` to choose properties. | [Windows Internals — service inventory](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md#4-windows-services) |
| `Get-CimInstance Win32_Service -Filter "ProcessId = 4436" \| Select-Object Name, DisplayName, State, ProcessId, PathName` | Finds service records associated with the example process ID and displays selected fields. | Replace `4436` with the PID being investigated. A service may share a host process with other services; corroborate the result with process and event evidence. | [Windows Internals — map service to PID](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md#4-windows-services) |
| `Get-CimInstance Win32_Service \| Where-Object ProcessId -eq 4436 \| Select-Object Name, DisplayName, State, ProcessId, PathName` | Filters service objects by process ID using the PowerShell pipeline. | `Where-Object` filters objects locally; `Select-Object` controls displayed properties. Replace the example PID with the one under investigation. | [Windows Internals — investigate services hosted by svchost](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md#5-what-is-svchostexe) |

## Windows Event Log queries

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `Get-WinEvent -ListLog *` | Lists available Windows event logs. | Add `-ComputerName` for a remote computer where access is configured; filter the output with `Where-Object` to find a channel. | [Windows Event Logs — enumerate channels](../Cybersecurity/03-Endpoint%20Security/Windows%20Event%20Logs.md#5-useful-powershell-commands) |
| `Get-WinEvent -LogName System -MaxEvents 20` | Reads the 20 most recent events from the System log. | Change `-LogName` to the required channel and `-MaxEvents` to the desired sample size. | [Windows Event Logs — System channel](../Cybersecurity/03-Endpoint%20Security/Windows%20Event%20Logs.md#5-useful-powershell-commands) |
| `Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20` | Reads recent events from the Sysmon Operational channel. | The channel must exist and be enabled. Use `-FilterHashtable` to constrain by Event ID, provider, log name or time. | [Sysmon — query the Operational channel](../Cybersecurity/03-Endpoint%20Security/Sysmon.md#2-where-are-sysmon-events-stored) · [Windows Event Logs — Sysmon channel](../Cybersecurity/03-Endpoint%20Security/Windows%20Event%20Logs.md#5-useful-powershell-commands) |
| `Get-WinEvent -FilterHashtable @{ LogName = "Microsoft-Windows-Sysmon/Operational"; Id = 1 } -MaxEvents 10` | Filters Sysmon events to Event ID 1 (Process Create) and returns up to ten results. | Add `StartTime` and `EndTime` keys to focus on an investigation window. Event ID interpretation depends on the provider and channel. | [Windows Event Logs — filter Event ID 1](../Cybersecurity/03-Endpoint%20Security/Windows%20Event%20Logs.md#5-useful-powershell-commands) |
| `Get-WinEvent -FilterHashtable @{ LogName = "Microsoft-Windows-Sysmon/Operational"; Id = 13 } -MaxEvents 1 \| Select-Object TimeCreated, Id, ProviderName, Message` | Retrieves the newest matching Sysmon Event ID 13 and displays its time, ID, provider and message. | Change `Id` to the event being investigated; add `StartTime` / `EndTime` to constrain the time window. An empty result means no event matched that query at that moment; it does not prove Sysmon is broken. | [Project 01 — validate raw Sysmon archive event](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#the-event-i-found-in-discover) |

## Network configuration

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `Get-NetIPConfiguration` | Returns network interface configuration including IP addressing, gateways and DNS information. | Pipe to `Format-List` for more detail or select properties such as `InterfaceAlias`, `IPv4Address` and `IPv4DefaultGateway`. | [Host inventory — Windows guest network configuration](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#6-validation-before-a-lab-session) |

## Pipeline patterns used in these commands

| Pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `\| Where-Object PROPERTY -eq VALUE` | Keeps objects whose selected property matches a value. | Supports comparison operators such as `-eq`, `-ne`, `-like`, `-match`, `-gt` and `-lt`. | [Windows Internals — service filtering](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md#5-what-is-svchostexe) |
| `\| Select-Object Name, State, ProcessId` | Selects the properties displayed from each object. | Use `-First N` for an initial sample or `-ExpandProperty PROPERTY` to output a property's value directly. | [Windows Internals — choose service properties](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md#4-windows-services) |

## Related notes

- [Windows Internals Overview](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md)
- [Windows Event Logs](../Cybersecurity/03-Endpoint%20Security/Windows%20Event%20Logs.md)
- [Sysmon](../Cybersecurity/03-Endpoint%20Security/Sysmon.md)
- [Host Inventory & Lab Setup](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md)
- [05 - Windows & Sysmon Utilities](05%20-%20Windows%20%26%20Sysmon%20Utilities.md)
