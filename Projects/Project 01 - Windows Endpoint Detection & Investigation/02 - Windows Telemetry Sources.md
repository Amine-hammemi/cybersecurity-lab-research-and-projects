---
title: Windows Telemetry Sources
type: reference
status: learning
---

# Windows Telemetry Sources

## 1. Sysmon

Sysmon is a Windows system service and driver that records detailed system activity in the `Microsoft-Windows-Sysmon/Operational` event channel.

It supplements standard Windows logging. It does not replace Windows Security logs or PowerShell-specific logging.

### Event IDs encountered in this project

| Event ID | Meaning | Investigation value |
|---:|---|---|
| 1 | Process creation | Image, command line, user, parent process and process identifiers |
| 7 | Image loaded | Records image/DLL loading when enabled by configuration |
| 11 | File created | Records file creation activity when enabled by configuration |
| 13 | Registry value set | Records registry value modifications when enabled by configuration |

The actual events produced depend on the installed Sysmon configuration. A configured event type may be excluded or filtered.

## 2. PowerShell Operational log

PowerShell writes events to:

```text
Microsoft-Windows-PowerShell/Operational
```

### Event ID 4104 — Script Block Logging

- Records script-block content processed by the PowerShell engine when Script Block Logging is enabled.
- Long script blocks can be split across multiple event records. Messages such as `Creating Scriptblock text (2 of 2)` indicate fragments, not severity.
- It is useful for understanding what code PowerShell processed.
- It does not generally record the output/results of commands.

Example:

```powershell
Get-ComputerInfo
```

A 4104 event may record the script content containing that command. It does not normally contain the returned computer-information output.

## 3. Process creation versus script content

| Question | Sysmon Event ID 1 | PowerShell Event ID 4104 |
|---|---|---|
| Was a process created? | Yes | Not its primary purpose |
| Which executable ran? | Yes | Not necessarily |
| Who launched it? | User context is available | Event metadata may provide context |
| Which parent launched it? | Yes | Not its primary purpose |
| What script content was processed? | No | Yes |
| What output did the command return? | No | Not generally |

Neither event alone proves malicious intent. Analysts need context, sequence, authorization, and corroborating evidence.

## 4. Other useful Windows telemetry for later investigation

| Source / Event                         | Why it may matter                          |
| -------------------------------------- | ------------------------------------------ |
| Security Event ID 4698                 | A scheduled task was created               |
| TaskScheduler/Operational Event ID 106 | A task was registered                      |
| TaskScheduler/Operational Event ID 140 | A task was updated                         |
| Sysmon Event ID 1                      | Process launched by a task or other parent |
| Sysmon Event ID 11                     | Script or payload file created             |

Availability depends on audit policy, channel configuration, and Sysmon configuration. Verify that the relevant events are enabled and collected before relying on them.
