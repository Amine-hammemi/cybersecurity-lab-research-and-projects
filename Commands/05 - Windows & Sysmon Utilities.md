---
tags:
  - cybersecurity/reference
  - windows
  - sysmon
  - command-reference
status: maintained
---

# Windows & Sysmon Utilities

Windows built-in utilities and Sysmon's command-line interface that appear in my endpoint and lab notes. Network configuration commands such as `ipconfig` are listed in [02 - Networking Commands](02%20-%20Networking%20Commands.md). PowerShell cmdlets are documented separately in [04 - PowerShell Commands](04%20-%20PowerShell%20Commands.md).

## Windows built-in utilities

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `tasklist /svc` | Lists running processes and the services hosted by each process. | `/fi` adds filters; `/fo table` or `/fo csv` changes output format. Useful when investigating `svchost.exe` and service-to-process relationships. | [Windows Internals — tasklist service mapping](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md#5-what-is-svchostexe) |
| `eventvwr.msc` | Opens Windows Event Viewer. | Can be launched from Run, Command Prompt or PowerShell. Use it to browse channels and inspect event details. | [Windows Event Logs — open Event Viewer](../Cybersecurity/03-Endpoint%20Security/Windows%20Event%20Logs.md#3-event-viewer) |

## Sysmon command-line options

> Run Sysmon management commands from an elevated terminal. Check the installed binary name and path on the endpoint before invoking it.

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `sysmon.exe -s` | Displays the configuration schema supported by the installed Sysmon binary. | Useful to confirm which schema version the configuration file must target. | [Sysmon — inspect schema](../Cybersecurity/03-Endpoint%20Security/Sysmon.md#4-sysmon-configuration) |
| `sysmon.exe -c` | Displays the currently active Sysmon configuration. | Use it to inspect the effective configuration rather than assuming a downloaded file was applied. | [Sysmon — inspect active configuration](../Cybersecurity/03-Endpoint%20Security/Sysmon.md#4-sysmon-configuration) |
| `sysmon.exe -c C:\Sysmon\sysmonconfig.xml` | Applies a Sysmon configuration file. | Validate the configuration/schema and review the change before applying it. `-c` without a file displays the active configuration; with a file it updates the configuration. | [Sysmon — apply configuration](../Cybersecurity/03-Endpoint%20Security/Sysmon.md#4-sysmon-configuration) |

## Related notes

- [Windows Internals Overview](../Cybersecurity/03-Endpoint%20Security/Windows%20Internals%20Overview.md)
- [Windows Event Logs](../Cybersecurity/03-Endpoint%20Security/Windows%20Event%20Logs.md)
- [Sysmon](../Cybersecurity/03-Endpoint%20Security/Sysmon.md)
- [04 - PowerShell Commands](04%20-%20PowerShell%20Commands.md)
