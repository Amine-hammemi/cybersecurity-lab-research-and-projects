# Sysmon (System Monitor)

> My notes on Sysmon and how I use it to collect detailed Windows security telemetry for my SOC lab.

## 1. What is Sysmon?

- Sysmon is a Windows system service and driver from Microsoft Sysinternals.
- It monitors selected system activity and records detailed events in the Windows Event Log.
- It can provide visibility into process creation, network connections, file creation, Registry activity and more.
- It is a source of telemetry for investigation and detection.

**Sysmon is not an antivirus.**

- It does not automatically decide that every recorded activity is malicious.
- It does not block, terminate or quarantine processes.
- An analyst or detection platform uses its telemetry to investigate activity.

## 2. Where are Sysmon events stored?

Sysmon writes events to:

```text
Microsoft-Windows-Sysmon/Operational
```

I can inspect the channel using Event Viewer or PowerShell:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20
```

## 3. Important Sysmon Event IDs

| Event ID | Activity | Investigation value |
|---:|---|---|
| 1 | Process Create | Process path, command line, parent process, user |
| 3 | Network Connect | Process-associated network connection |
| 5 | Process Terminated | Process termination |
| 7 | Image Loaded | DLL/image loading |
| 8 | CreateRemoteThread | Potential cross-process thread creation |
| 10 | Process Access | One process accessing another process |
| 11 | File Create | File creation |
| 12 | Registry object create/delete | Registry activity |
| 13 | Registry value set | Registry modification |
| 14 | Registry key/value rename | Registry rename activity |
| 22 | DNS Query | DNS query activity |
| 25 | Process Tampering | Certain forms of process image tampering |

These are selected examples, not the complete Sysmon event catalogue. Event generation depends on the active configuration.

**Configuration detail:** Event ID 3 (Network Connect) and Event ID 7 (Image Loaded) are disabled by default in Sysmon and must be enabled in the configuration when needed. Event ID 7 can generate a high event volume, so I should enable it selectively and review the resulting telemetry load.

**Important:** an Event ID is evidence of an observed activity, not a verdict. For example, Event ID 8 is not proof that process injection succeeded.

## 4. Sysmon configuration

- Sysmon configuration determines which event types are included and excluded.
- Filters can reduce noise and control event volume.
- A configuration should balance useful visibility, endpoint performance and storage.
- A configuration change can significantly change which events are generated.

I used the Sysmon Modular configuration maintained by Olaf Hartong:

- [Sysmon Modular GitHub project](https://github.com/olafhartong/sysmon-modular)

It is a community-maintained configuration project, not a default Microsoft configuration.

Useful commands (run in an elevated PowerShell terminal):

```powershell
sysmon.exe -s
```

Displays the configuration schema supported by the installed Sysmon binary.

```powershell
sysmon.exe -c
```

Displays the active configuration.

```powershell
sysmon.exe -c C:\Sysmon\sysmonconfig.xml
```

Applies a configuration file.

## 5. My lab implementation

I configured Sysmon on my Windows 11 VM.

- Installed Sysmon.
- Checked the supported configuration schema.
- Downloaded the Sysmon Modular configuration.
- Validated that the configuration schema matched the installed Sysmon version.
- Applied the configuration.
- Confirmed that the Sysmon service was running.
- Verified that Sysmon events were being generated.
- Observed events including IDs 1, 7, 11 and 13.
- Configured the Wazuh Agent to collect the Sysmon Operational channel.

The Wazuh Agent configuration includes:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

This tells the agent to collect events from the Sysmon Operational channel.

See [Wazuh Architecture](../04-Security%20Monitoring/Wazuh%20Architecture.md) for how these events reach the Manager.

## 6. Telemetry volume and storage

Collecting more events is not always better.

Potential impact:

1. **Windows endpoint:** more events written to local event logs.
2. **Agent/network:** more data to process and transmit.
3. **Wazuh Manager:** more incoming events to process.
4. **Wazuh Indexer:** more indexed documents and disk usage.

My lab has 16 GB of host RAM and an 8 GB Wazuh VM, so I need to keep telemetry useful and avoid collecting everything without a reason.

## 7. How Sysmon helps an investigation

Example process chain:

```text
winword.exe
└── powershell.exe
    └── unknown.exe
```

With Event ID 1, I can investigate:

- Which process started the child process?
- What command line was used?
- Which user executed it?
- What was the executable path?
- When did it happen?
- What other events occurred around the same time?

I should correlate multiple events instead of deciding that one event proves an attack.

## 8. What I still need to practise

- Read Event ID 1 fields in detail.
- Understand ProcessGuid and ProcessId.
- Correlate process creation with network and file events.
- Compare normal Windows activity with suspicious behaviour.
- Create and test a small custom detection use case.

## Command reference

- [PowerShell event-query reference](../../Commands/04%20-%20PowerShell%20Commands.md)
- [Sysmon command-line reference](../../Commands/05%20-%20Windows%20%26%20Sysmon%20Utilities.md)

## Related notes

- [Windows Internals Overview](Windows%20Internals%20Overview.md)
- [Windows Event Logs](Windows%20Event%20Logs.md)
- [SIEM Fundamentals](../04-Security%20Monitoring/SIEM%20Fundamentals.md)
- [Wazuh Architecture](../04-Security%20Monitoring/Wazuh%20Architecture.md)
- [Wazuh SOC Lab](../04-Security%20Monitoring/Wazuh%20SOC%20Lab.md)

## Official resources

- [Microsoft Sysmon documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Sysmon events reference](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon#events)
- [Sysmon Modular](https://github.com/olafhartong/sysmon-modular)
