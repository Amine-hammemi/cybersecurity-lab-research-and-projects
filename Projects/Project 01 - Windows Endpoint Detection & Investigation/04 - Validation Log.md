---
title: Validation Log
type: project-evidence
status: partial
---

# Validation Log

This note records what was actually tested during the current Project 01 work.

## Validated

### Windows endpoint
- Windows 11 VM was running.
- Wazuh Agent service `WazuhSvc` was running.
- Sysmon was installed and running.
- Sysmon Event ID 1 process-creation records were visible locally.
- PowerShell Operational channel was enabled.
- Historical Event ID 4104 records were present.
- The explicit Script Block Logging policy value was created and verified as `1`.
- A controlled script block containing `SOC-Project01-4104-Test` was found in a local Event ID 4104 event.

### Wazuh
- The Windows endpoint appeared as an active agent in the Dashboard during the project work.
- Sysmon provider events were visible using:

```text
data.win.system.providerName: "Microsoft-Windows-Sysmon"
```

- A PowerShell collection block was added to the Windows agent `ossec.conf`.
- The agent was restarted after the configuration change.
- The PowerShell provider query then returned events:

```text
data.win.system.providerName: "Microsoft-Windows-PowerShell"
```

## Not yet validated

- Exact controlled marker `SOC-Project01-4104-Test` visible in a Wazuh event document.
- Exact controlled PowerShell 4104 marker visible in the Wazuh archive index.
- A custom Wazuh detection rule for the target behaviour.
- A complete controlled suspicious-activity simulation.
- Correlation across Sysmon, PowerShell, Security and Task Scheduler telemetry.
- A complete investigation timeline.
- A final incident report.

### Controlled Sysmon Event ID 11 — end-to-end validation (2026-11-24)

I validated a narrowly scoped Sysmon `FileCreate` rule and followed one controlled event through the complete raw-event pipeline.

#### What I changed
- Backed up the original Sysmon Modular configuration before editing it.
- Added one `TargetFilename` include rule inside the existing `<FileCreate onmatch="include">` group. The rule matches the test filename `SOC-Project01-FileCreation-Test.txt` and does not enable broad file-creation monitoring.
- Parsed the XML with PowerShell before applying it. The XML syntax check passed.
- Reloaded the configuration with Sysmon and checked the active configuration output. The new rule appeared under `FileCreate`, not `FileCreateTime`.

#### Why the first test did not produce the expected event
- The first test file already existed. Running `Set-Content` again overwrote its contents; it did not create a new file.
- Sysmon Event ID 11 records file creation, so overwriting an existing file was not a valid way to trigger this test.
- I removed the old test file and used `New-Item` to create a fresh file. The matching Event ID 11 then appeared locally.

#### Correlated evidence
The same event was verified at each stage. Public documentation intentionally omits the live username, hostname, private IP and full user profile path.

| Evidence                  | Observed value                                                      |
| ------------------------- | ------------------------------------------------------------------- |
| Provider / channel        | `Microsoft-Windows-Sysmon` / `Microsoft-Windows-Sysmon/Operational` |
| Event ID                  | `11` — FileCreate                                                   |
| Event Record ID           | `17776`                                                             |
| Rule name                 | `Project 01 controlled file-creation validation`                    |
| Event time                | `2026-11-24 19:33:39 UTC`                                           |
| Process                   | `powershell.exe`                                                    |
| Process ID                | `5028`                                                              |
| Process GUID              | `{f9f27d66-bdef-6b05-5803-000000000c00}`                            |
| Target                    | `%TEMP%\SOC-Project01-FileCreation-Test.txt`                        |
| Wazuh decoder             | `windows_eventchannel`                                              |
| Wazuh agent name          | `win11`                                                             |
| Manager archive timestamp | `2026-11-24T19:33:39.586+0000`                                      |

The Windows event XML and Wazuh archive record matched on Event ID, Event Record ID, Process GUID, Process ID, target filename and event time. This establishes that the Manager archive record represents the same endpoint event, rather than an unrelated event with similar content.

#### Discover validation
- Found the event in the `wazuh-archives` data view.
- Confirmed the structured fields were available and populated:
  - `data.win.eventdata.ruleName`
  - `data.win.eventdata.processGuid`
  - `data.win.eventdata.processId`
  - `data.win.eventdata.image`
  - `data.win.eventdata.targetFilename`
  - `data.win.system.eventRecordID`
- The query also returned a separate Event ID 4672 record. That is a Windows Security special-privileges logon event, not the Sysmon file-creation event. Its presence alone is not evidence of malicious activity.

#### What this proves — and what it does not
- **Proven:** Sysmon generated the controlled Event ID 11; the Wazuh agent collected it; the Manager decoded and archived it; Filebeat/Indexer made it searchable; Discover exposed the expected structured fields.
- **Not proven:** a custom Wazuh alert or detection rule. This was raw-event collection and search validation, not detection engineering.
- **Not proven:** a complete suspicious-activity investigation or incident classification. One benign file-creation test is only a telemetry validation case.

### Raw archive collection and indexing (2026-11-22)
- Enabled Manager JSON archive generation with `<logall_json>yes</logall_json>`; left `<logall>no</logall>` unchanged.
- Confirmed Windows/Sysmon records in `/var/ossec/logs/archives/archives.json`.
- Enabled `archives.enabled: true` in the Filebeat Wazuh module; left alert indexing enabled.
- Filebeat configuration test returned `Config OK`; Filebeat service was active after restart.
- `filebeat test output` verified Indexer connectivity, TLS and authentication (Indexer reported version `7.10.2`).
- Filebeat registry contained state for the archive JSON file. This proves the input was tracked, not that every event was indexed.
- Index `wazuh-archives-4.x-2026.11.22` was green/open. Document count was 799, then 888 (+89) on a later check.
- Created the `wazuh-archives` data view with pattern `wazuh-archives-4.x-*` and time field `timestamp`.
- Retrieved a Sysmon Event ID 13 document in Discover from the archive index. The indexed document showed `services.exe` setting the `UsoSvc\DelayedAutostart` registry value.
- The document timestamp was 00:20:27 UTC, corresponding to approximately 01:20:27 local time (UTC+1).
- This validates raw Sysmon event indexing and search. The exact controlled PowerShell 4104 marker is still not confirmed in the archive index.
