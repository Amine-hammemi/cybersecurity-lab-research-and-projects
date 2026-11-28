---
title: Validation Log
type: project-evidence
status: partial
---
# Validation Log

> **Project:** Project 01 — Windows Endpoint Detection & Investigation
> **Purpose:** Record practical tests, collected evidence, validation results and remaining limitations.
> **Last updated:** 2026-11-28

---

## Quick Navigation

|Section|Description|
|---|---|
|[Validation Overview](#validation-overview)|Current validation status|
|[Windows Endpoint Baseline](#windows-endpoint-baseline)|Initial endpoint checks|
|[Raw Archive Collection and Indexing](#raw-archive-collection-and-indexing-2026-11-22)|Wazuh archive infrastructure|
|[Sysmon Event ID 11](#controlled-sysmon-event-id-11--end-to-end-validation-2026-11-24)|Controlled file-creation test|
|[PowerShell Event ID 4104](#controlled-powershell-event-id-4104--end-to-end-validation-2026-11-28)|Script Block Logging test|
|[Outstanding Validation](#outstanding-validation)|Remaining project work|

---

## Validation Overview

### Current status

|Component|Validation status|Date|
|---|---|---|
|Windows endpoint and Wazuh Agent|Validated|Initial setup|
|Sysmon process telemetry (Event ID 1)|Validated locally|Initial setup|
|Wazuh raw archive collection|Validated|2026-11-22|
|Sysmon Event ID 13 indexing|Validated|2026-11-22|
|Sysmon Event ID 11 end-to-end|Validated|2026-11-24|
|PowerShell Event ID 4104 local collection|Validated|2026-11-28|
|PowerShell Event ID 4104 Wazuh indexing|Validated|2026-11-28|
|Custom Wazuh detection rules|Pending|—|
|Suspicious activity simulation|Pending|—|
|Cross-source event correlation|Pending|—|
|Complete incident investigation|Pending|—|

### Validation scope

**What has been demonstrated:**

- Windows endpoint telemetry generation.

- Wazuh Agent collection and forwarding.

- Manager raw archive generation.

- Filebeat archive ingestion.

- Wazuh Indexer storage and Discover search.

- End-to-end validation of controlled Sysmon and PowerShell events.


**What has not yet been demonstrated:**

- Custom detection and alert generation.

- Detection accuracy and false-positive tuning.

- Complete suspicious-activity investigation.

- Incident classification and response.


---

## Windows Endpoint Baseline

### Validated configuration

|Component|Observed state|
|---|---|
|Windows endpoint|Windows 11 VM running|
|Wazuh Agent service|`WazuhSvc` running|
|Sysmon|Installed and running|
|Sysmon Event ID 1|Process-creation records visible locally|
|PowerShell Operational channel|Enabled|
|Historical Event ID 4104|Records present locally|
|Script Block Logging policy|`EnableScriptBlockLogging = 1`|

### Wazuh collection checks

- Windows endpoint appeared as an active agent in Wazuh Dashboard.

- Sysmon provider events were searchable using:


```
data.win.system.providerName: "Microsoft-Windows-Sysmon"
```

- A PowerShell collection block was added to the Windows agent `ossec.conf`.

- The Wazuh Agent was restarted after the configuration change.

- PowerShell provider events subsequently appeared using:


```
data.win.system.providerName: "Microsoft-Windows-PowerShell"
```

### Earlier local-only PowerShell test

A controlled script block containing:

```
SOC-Project01-4104-Test
```

was found in a local Event ID 4104 event.

**Limitation:** The exact `SOC-Project01-4104-Test` marker was not confirmed in a corresponding Wazuh archive document. A separate controlled test on 2026-11-28 subsequently validated Event ID 4104 end-to-end.

---

## Raw Archive Collection and Indexing (2026-11-22)

### Configuration changes

|Component|Configuration|
|---|---|
|Wazuh Manager|Enabled `<logall_json>yes</logall_json>`|
|Standard log generation|Kept `<logall>no</logall>`|
|Archive output|`/var/ossec/logs/archives/archives.json`|
|Filebeat Wazuh module|Enabled `archives.enabled: true`|
|Alert indexing|Left enabled|
|Discover data view|`wazuh-archives`|
|Index pattern|`wazuh-archives-4.x-*`|
|Time field|`timestamp`|

### Validation evidence

- Confirmed Windows/Sysmon records in `archives.json`.

- Filebeat configuration test returned `Config OK`.

- Filebeat service was active after restart.

- `filebeat test output` verified Indexer connectivity, TLS and authentication.

- Indexer reported version `7.10.2`.

- Filebeat registry contained state for the archive JSON input.


> **Evidence limitation:** Registry state confirms that the input was tracked; it does not independently prove that every event was indexed.

### Index validation

|Check|Observed result|
|---|---|
|Index|`wazuh-archives-4.x-2026.11.22`|
|Index health|Green|
|Initial document count|799|
|Later document count|888|
|Document count increase|89|
|Data view|`wazuh-archives`|

### Event verification

- Retrieved a Sysmon Event ID 13 document in Discover.

- The indexed event showed `services.exe` setting the `UsoSvc\DelayedAutostart` registry value.

- Event timestamp: `00:20:27 UTC`, approximately `01:20:27` Tunisia time.


**Result:** Raw Sysmon event indexing and search were validated.

---

## Controlled Sysmon Event ID 11 — End-to-End Validation (2026-11-24)

### Objective

Validate a controlled Sysmon file-creation event across the complete Wazuh raw telemetry pipeline.

### Configuration and procedure

1. Backed up the original Sysmon Modular configuration.

2. Added one narrowly scoped `TargetFilename` inclusion rule inside the existing `<FileCreate onmatch="include">` group.

3. Configured the rule to match:


```
SOC-Project01-FileCreation-Test.txt
```

4. Parsed the XML configuration with PowerShell. Syntax validation passed.

5. Reloaded the configuration with Sysmon.

6. Verified that the new rule appeared under `FileCreate`, not `FileCreateTime`.


### Test issue and correction

- The initial test file already existed.

- Using `Set-Content` overwrote the existing file rather than creating a new one.

- Sysmon Event ID 11 records file creation, so this did not trigger the expected event.

- Removed the old file and created a fresh one using `New-Item`.

- The expected Event ID 11 then appeared locally.


### Correlated evidence

The same event was identified in Windows local event evidence and Wazuh archives.

|Evidence field|Observed value|
|---|---|
|Provider|`Microsoft-Windows-Sysmon`|
|Channel|`Microsoft-Windows-Sysmon/Operational`|
|Event ID|`11` — FileCreate|
|Event Record ID|`17776`|
|Rule name|`Project 01 controlled file-creation validation`|
|Event time|`2026-11-24 19:33:39 UTC`|
|Process|`powershell.exe`|
|Process ID|`5028`|
|Process GUID|`{f9f27d66-bdef-6b05-5803-000000000c00}`|
|Target|`%TEMP%\SOC-Project01-FileCreation-Test.txt`|
|Wazuh decoder|`windows_eventchannel`|
|Wazuh agent|`win11`|
|Manager archive timestamp|`2026-11-24T19:33:39.586+0000`|

Public documentation intentionally omits the live username, hostname, private IP and full user profile path.

### Discover validation

- Located the event in the `wazuh-archives` data view.

- Verified the following structured fields:


```
data.win.eventdata.ruleName
data.win.eventdata.processGuid
data.win.eventdata.processId
data.win.eventdata.image
data.win.eventdata.targetFilename
data.win.system.eventRecordID
```

- A separate Event ID 4672 record also appeared in the query results.

- Event ID 4672 is a Windows Security special-privileges logon event, not the Sysmon file-creation event. Its presence alone does not indicate malicious activity.


### Validated telemetry path

```
Sysmon
  → Wazuh Agent
  → Wazuh Manager
  → archives.json
  → Filebeat
  → Wazuh Indexer
  → Discover
```

### Result and limitations

**Result: SUCCESS — Raw telemetry collection and indexing validated.**

**Proven:**

- Sysmon generated the controlled Event ID 11.

- The Wazuh Agent collected and forwarded it.

- The Manager decoded and archived it.

- Filebeat and the Indexer made it searchable.

- Discover exposed the expected structured fields.


**Not proven:**

- A custom Wazuh alert or detection rule.

- A complete suspicious-activity investigation.

- Malicious intent or incident classification.


---

## Controlled PowerShell Event ID 4104 — End-to-End Validation (2026-11-28)

### Objective

Validate PowerShell Script Block Logging (Event ID 4104) from the Windows 11 endpoint through Wazuh raw archives and Discover.

### Configuration verification

|Configuration|Observed value|
|---|---|
|Registry path|`HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging`|
|`EnableScriptBlockLogging`|`1`|
|PowerShell Operational channel|Enabled|
|Existing records before test|1,611|

### Controlled test

Executed a harmless PowerShell command that generated a unique validation marker:

```
Soc4104-Validation-20261128-075339
```

The corresponding script block was found in the local Windows PowerShell Operational log.

### Correlated evidence

|Evidence field|Observed value|
|---|---|
|Agent name|`win11`|
|Agent ID|`001`|
|Agent IP|`192.168.100.17`|
|Computer|`DESKTOP-GTT1CIR`|
|Provider|`Microsoft-Windows-PowerShell`|
|Event ID|`4104`|
|Event Record ID|`1616`|
|ScriptBlock ID|`22e42299-d0ec-423b-9ee9-d10baf801298`|
|Process ID|`6172`|
|Channel|`Microsoft-Windows-PowerShell/Operational`|
|Windows event time (UTC)|`2026-11-28T06:53:39.1898809Z`|
|Wazuh index|`wazuh-archives-4.x-2026.11.28`|

### Wazuh Discover verification

The matching event was retrieved from:

```
wazuh-archives-4.x-2026.11.28
```

Important indexed fields:

```
data.win.system.eventID
data.win.system.eventRecordID
data.win.system.systemTime
data.win.system.channel
data.win.system.processID
data.win.eventdata.scriptBlockText
data.win.eventdata.scriptBlockId
agent.name
agent.id
agent.ip
```

The event was correlated using:

- Event ID: `4104`

- Event Record ID: `1616`

- ScriptBlock ID

- Agent identity

- Original Windows event timestamp


### Timestamp clarification

|Timestamp field|Value|Meaning|
|---|---|---|
|`data.win.system.systemTime`|`2026-11-28T06:53:39.1898809Z`|Original Windows event time|
|`@timestamp` / `timestamp`|Archive/index timestamps|Wazuh-side ingestion and indexing time|

Use `data.win.system.systemTime` for event chronology and cross-source correlation.

### Validated telemetry path

```
PowerShell Script Execution
  → Windows PowerShell Operational Log
  → Wazuh Agent
  → Wazuh Manager
  → archives.json
  → Filebeat
  → Wazuh Indexer
  → Discover
```

### Result and limitations

**Result: SUCCESS — Event ID 4104 raw telemetry collection and indexing validated.**

**Proven:**

- Script Block Logging was enabled.

- A controlled script block generated Event ID 4104 locally.

- The matching event reached Wazuh.

- The event was indexed in the raw archive index.

- The expected script block and correlation fields were available in Discover.


**Not proven:**

- Custom Wazuh detection or alert generation.

- Detection of malicious PowerShell activity.

- Complete correlation with Sysmon, Windows Security and Task Scheduler.

- Incident investigation or response.


---

## Outstanding Validation

The following tasks remain before the project can be considered a complete SOC detection and investigation case.

|Priority|Task|Status|
|---|---|---|
|1|Design a controlled suspicious-activity scenario|Pending|
|2|Identify expected Sysmon and Windows event telemetry|Pending|
|3|Build custom Wazuh detection rules|Pending|
|4|Validate alert generation and rule accuracy|Pending|
|5|Correlate process, PowerShell, file and other relevant events|Pending|
|6|Build a chronological investigation timeline|Pending|
|7|Produce an evidence-based incident report|Pending|

### Next validation milestone

**Detection Engineering and Controlled Activity Simulation**

Move beyond confirming that events are collected. The next objective is to determine whether Wazuh can identify a defined suspicious behaviour, generate an appropriate alert and provide enough evidence for an analyst to investigate it.

---

_Documentation principle: A successful telemetry test proves that an event was collected and indexed. It does not, by itself, prove that the behaviour was detected or that an incident was investigated._