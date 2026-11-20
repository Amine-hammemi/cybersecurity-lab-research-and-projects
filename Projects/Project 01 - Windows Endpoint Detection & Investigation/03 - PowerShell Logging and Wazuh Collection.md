---
title: PowerShell Logging and Wazuh Collection
type: technical-note
status: validated
---

# PowerShell Logging and Wazuh Collection

## PowerShell Script Block Logging

PowerShell Script Block Logging records script-block content as Event ID 4104 in:

```text
Microsoft-Windows-PowerShell/Operational
```

In this lab, the explicit policy registry value was absent initially. I enabled it on the Windows 11 endpoint.

Registry path:

```text
HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
```

Value:

```text
Name: EnableScriptBlockLogging
Type: REG_DWORD
Data: 1
```

PowerShell verification:

```powershell
Get-ItemPropertyValue `
  -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging' `
  -Name 'EnableScriptBlockLogging'
```

Expected output:

```text
1
```

This was verified in the lab.

## Wazuh Agent collection configuration

The Windows Wazuh Agent configuration file is:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

The existing Sysmon collection block was:

```xml
<localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>
```

I added the PowerShell Operational channel:

```xml
<localfile>
    <location>Microsoft-Windows-PowerShell/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>
```

Both blocks must be correctly placed inside the configuration's `<ossec_config>` root element. XML tags must be paired and blocks must not be nested inside one another.

## Restarting the Windows agent

After editing `ossec.conf`, the agent must be restarted to apply the updated collection configuration.

In Administrator PowerShell:

```powershell
Restart-Service -Name WazuhSvc
Get-Service -Name WazuhSvc
```

The service was observed running after restart.

## Search in Wazuh Dashboard

The provider query that returned Sysmon events:

```text
data.win.system.providerName: "Microsoft-Windows-Sysmon"
```

The corresponding PowerShell provider query initially returned no results before the agent was restarted after the configuration change:

```text
data.win.system.providerName: "Microsoft-Windows-PowerShell"
```

After restarting `WazuhSvc`, PowerShell provider events became visible in the Dashboard.

Important:
- A collected event does not necessarily generate an alert.
- Threat Hunting visibility depends on the data view/index and Wazuh configuration.

## Controlled local test

A benign test script block was executed:

```powershell
& {
    $marker = "SOC-Project01-4104-Test"
    Write-Output $marker
}
```

The marker was found in a local Event ID 4104 record after Script Block Logging was enabled.

The PowerShell provider events later became visible in Wazuh after the agent restart. The exact marker's presence in the Wazuh document was not separately confirmed in the recorded session, so I do not claim that exact marker was verified end-to-end.

## Troubleshooting lessons

1. Windows channel enabled does not by itself mean Script Block Logging is enabled.
2. A missing policy registry key means the explicit policy value is absent; it does not automatically prove that no 4104 events can ever exist.
3. Configure Script Block Logging, then test with a new PowerShell session.
4. Confirm local event generation before troubleshooting Wazuh.
5. Confirm the Wazuh Agent collection block.
6. Restart the agent after changing `ossec.conf`.
7. Search Wazuh using the provider field observed in actual documents.
8. Distinguish collection, indexing, and alert generation.
