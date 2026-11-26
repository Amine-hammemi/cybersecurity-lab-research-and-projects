---
title: Wazuh Raw Archive Collection and Indexing
type: technical-note
status: validated
---

# Wazuh Raw Archive Collection and Indexing

## Why I investigated this

I could see some Sysmon and PowerShell events in the Wazuh Dashboard, but I could not reliably find the raw Windows events I wanted to investigate. At first, I treated this as a general collection problem. The investigation showed that there were **two separate stages** to check:

1. Does the Wazuh Manager save the raw events it receives?
2. Does Filebeat send those saved raw events to the Indexer so Discover can search them?

These are different operations. A Windows event can reach the Manager and be written to a local archive file without being available as a searchable document in the Indexer.

During the live troubleshooting, I followed commands and copied/pasted them before I fully understood what question each one was answering. I am writing this note to rebuild the investigation in the right order: **symptom → hypothesis → check → evidence → conclusion**. The commands are not just a recipe; I want to understand what each result proves and what it does not prove.

## The data path I validated

```text
Windows 11
  └─ Sysmon / Windows Event Channels
       ↓
Wazuh Agent
       ↓
Wazuh Manager
  └─ archives.json (raw JSON events)
       ↓
Filebeat — Wazuh archives module
       ↓
Wazuh Indexer
  └─ wazuh-archives-4.x-YYYY.MM.DD
       ↓
Wazuh Dashboard / Discover
  └─ wazuh-archives data view
```

### Two data paths that I must not confuse

| Data path | What it stores | What it means |
|---|---|---|
| Alerts | Events that match Wazuh rules and are promoted to alerts | Useful for detection and triage, but not every collected event becomes an alert |
| Archives | Raw events received by the Manager, including events that did not trigger an alert | Useful for threat hunting, timeline reconstruction and writing/testing detection rules |

**Important:** enabling archive collection does not mean every raw event becomes an alert. It makes the raw event stream searchable when the full archive pipeline is configured.

## What was wrong

I found two configuration points that controlled different parts of the pipeline.

| Component | Setting before the change | Effect |
|---|---|---|
| Wazuh Manager | `<logall_json>no</logall_json>` | The Manager was not configured to write all received events to the JSON archive |
| Filebeat Wazuh module | `archives.enabled: false` | Filebeat was not configured to index the archive stream |

The Manager setting controls **archive generation**. The Filebeat setting controls **archive forwarding/indexing**. Both sides were needed.

## Changes I made

### 1. Enable JSON archives on the Manager

I backed up the Manager configuration before editing it:

```bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup-project01
```

In `/var/ossec/etc/ossec.conf`, I changed only this value:

```xml
<logall_json>yes</logall_json>
```

I left `<logall>no</logall>` unchanged. The JSON archive is the format used by the Filebeat Wazuh archives module in this setup.

Then I checked the Manager configuration syntax and restarted the service:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
sudo systemctl restart wazuh-manager
systemctl is-active wazuh-manager
```

The configuration test returned no errors, and the service reported `active`.

After this change, the Manager created `/var/ossec/logs/archives/archives.json`. I verified that it contained Windows agent events, including Sysmon events. This proved that raw events were reaching the Manager and being written locally. It did **not** yet prove that they were indexed.

### 2. Enable archive indexing in Filebeat

I backed up `/etc/filebeat/filebeat.yml`, then changed the Wazuh module configuration from:

```yaml
filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
    archives:
      enabled: false
```

to:

```yaml
filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
    archives:
      enabled: true
```

I kept alerts enabled. I changed only the archives value.

I validated the YAML/Filebeat configuration before restarting the service:

```bash
sudo filebeat test config -c /etc/filebeat/filebeat.yml
sudo systemctl restart filebeat
sudo systemctl is-active filebeat
```

The configuration test returned `Config OK`, and Filebeat reported `active` after restart.

### 3. Understand what the installed Wazuh module reads and creates

I inspected the installed module files rather than guessing the index name.

The archive module manifest identified:

| Item | Value |
|---|---|
| Source file | `/var/ossec/logs/archives/archives.json` |
| Index prefix | `wazuh-archives-4.x-` |
| Ingest pipeline | `ingest/pipeline.json` |
| Input type | `log` |

The Filebeat archive input uses the `paths` and `index_prefix` variables from that manifest. The Wazuh index template also included the `wazuh-archives-4.x-*` pattern.

I initially tried `filebeat modules list`. It returned an error saying that module management required `filebeat.config.modules.path`. That command's module-management interface expects a separate modules-directory setting; it was **not proof** that the `filebeat.modules:` block in the main configuration was inactive. My configuration used the embedded `filebeat.modules` mechanism, so I checked the actual main configuration and installed module files instead.

## How I verified each stage

I used multiple checks because no single command proves the entire pipeline.

| Check | Evidence | What it proves | What it does not prove |
|---|---|---|---|
| Manager config test | `wazuh-analysisd -t` completed without errors | Manager configuration syntax passed the test | Events are being received or indexed |
| Manager archive file | `archives.json` existed and contained Windows/Sysmon JSON records | Manager wrote raw events locally | Filebeat forwarded them |
| Filebeat config test | `Config OK` | Filebeat configuration parsed successfully | Indexer accepted documents |
| Filebeat output test | DNS, TCP, TLS and server communication succeeded; Indexer reported version `7.10.2` | Filebeat could authenticate and communicate with the Indexer | Archive events were published |
| Filebeat registry | Registry entries referenced `archives.json`, with tracked offsets | Filebeat registered/tracked the archive file | Every event was successfully indexed |
| Index Management | `wazuh-archives-4.x-2026.11.22` existed, green/open, 799 documents initially | Archive index existed and contained documents | A particular Sysmon event was searchable |
| Later document count | Count increased from 799 to 888 (+89) | New documents were being added during the observation period | All 89 were from the one test event; the count can include other event sources |
| Discover result | A Sysmon Event ID 13 document was returned from the archive data view | Raw Sysmon data was searchable in the Indexer/Dashboard | That event is malicious or an alert |

### The event I found in Discover

I created the `wazuh-archives` data view using this index pattern:

```text
wazuh-archives-4.x-*
```

I selected `timestamp` as its time field and searched for:

```text
data.win.system.eventID: "13"
```

A matching document was returned from the archive index. The relevant fields were:

| Field | Observed value |
|---|---|
| Provider | `Microsoft-Windows-Sysmon` |
| Event ID | `13` — Registry value set |
| Channel | `Microsoft-Windows-Sysmon/Operational` |
| Image | `services.exe` |
| Process ID | `900` |
| Event type | `SetValue` |
| Registry target | `HKLM\System\CurrentControlSet\Services\UsoSvc\DelayedAutostart` |
| User context | `NT AUTHORITY\SYSTEM` |
| Event time in document | `2026-11-22 00:20:27 UTC` |

The local Windows display showed approximately `01:20:27`. The one-hour difference is consistent with local time being UTC+1 while the event's `utcTime` and indexed timestamps are UTC. When correlating Windows and Wazuh evidence, I must compare timestamps in the same timezone.

This event records a registry value being set by `services.exe` for the Windows Update Orchestrator service (`UsoSvc`). Event ID 13 describes the registry operation; it does not, by itself, establish malicious persistence. I need the process context, surrounding events and baseline before making a security judgement.

## Concepts I learned from the troubleshooting

### Configuration, collection, indexing and alerting are different states

| Term | Meaning |
|---|---|
| Configured | The required setting is present in a configuration file |
| Active | The service is running |
| Collected | The event reached the relevant collector/Manager |
| Archived | The Manager wrote the raw event to its archive file |
| Harvested | Filebeat registered and read the archive input |
| Indexed | The Indexer stored the event as a searchable document |
| Alerted | A Wazuh rule matched and created an alert |

I should never report a later state based only on evidence for an earlier one.

### Why some troubleshooting attempts were misleading

- `systemctl is-active filebeat` only checks the service state. It does not prove that events are indexed.
- `filebeat test config` checks configuration syntax. It does not test event delivery.
- `filebeat test output` tests connectivity, TLS and authentication to the Indexer. It does not prove that the archive input is publishing documents.
- Filebeat's registry offset shows tracked file progress, not a guaranteed count of successfully indexed documents.
- A missing archive data view in Discover does not necessarily mean the index is missing. A data view is a Dashboard search definition; it is separate from the underlying Indexer index.
- A `401 Unauthorized` from my first manual `curl` request occurred because that request did not include the HTTP authentication configured for Filebeat. Filebeat's own `test output` later authenticated successfully.
- The systemd journal showed service lifecycle messages, while the Filebeat application log showed startup details. Neither alone was enough to prove archive indexing; the Index Management and Discover checks were stronger evidence.

## Storage and operational considerations

- At the time of validation, the archive index was green/open, with 3 primary shards, 0 replicas and approximately 1.3 MB total size.
- I observed 799 documents, then 888 documents later. This is a snapshot, not a permanent expected count.
- Raw archives can grow faster than alerts because they include events that do not match alert rules. I should monitor disk usage and retention rather than leave a high-volume archive stream unmanaged indefinitely.
- I did not delete old indices or archive data during this troubleshooting. Keeping the existing evidence was preferable to cleaning it before understanding the pipeline.

## How I should approach this kind of problem next time

Instead of running unrelated commands until something changes, I can split the pipeline into boundaries and test them one at a time:

1. **Endpoint:** can Windows produce the event locally? Check the correct event channel and Event ID.
2. **Agent/Manager:** is the agent connected, and does the Manager receive/write the event? Check agent status and the raw archive file.
3. **Filebeat input:** is archive collection enabled, and is Filebeat tracking the correct file? Inspect the module configuration and registry state.
4. **Filebeat output:** can Filebeat reach and authenticate to the Indexer? Use `filebeat test output`.
5. **Indexer:** does the expected archive index exist, and is its document count changing?
6. **Dashboard:** can I retrieve a real event from the archive data view using its provider, Event ID and timestamp?

At every boundary, I should write down the exact evidence before moving to the next one. If a check fails, investigate that boundary instead of changing several components at once.

## Current result

- [x] Manager JSON archive generation enabled.
- [x] Windows raw events observed in `archives.json`.
- [x] Filebeat archive module enabled.
- [x] Filebeat configuration validated and service restarted.
- [x] Filebeat output connection to the Indexer verified.
- [x] Archive index created/populated and visible in Index Management.
- [x] Archive data view created.
- [x] Sysmon Event ID 13 retrieved from Discover.
- [x] Controlled Sysmon Event ID 11 retrieved from the Manager archive and Discover, with matching endpoint identifiers.
- [ ] Exact controlled PowerShell Event ID 4104 marker retrieved from the archive index.
- [ ] Custom Wazuh detection rule and alert validated.

## Controlled Sysmon Event ID 11 — archive and Discover proof (2026-11-24)

After validating the event locally, I searched the Manager's raw archive for the unique test filename. The archive contained the Sysmon Event ID 11 with the same `eventRecordID` (`17776`), `processGuid`, `processId`, target filename and UTC event time as the Windows event XML.

I then searched the `wazuh-archives` data view in Discover. The Event ID 11 record was present, and I expanded it to confirm that these structured fields were available:

- `data.win.eventdata.ruleName`
- `data.win.eventdata.processGuid`
- `data.win.eventdata.processId`
- `data.win.eventdata.image`
- `data.win.eventdata.targetFilename`
- `data.win.system.eventRecordID`

This proves that the specific controlled Sysmon event crossed the Manager archive and indexing path and became searchable in Discover. It is stronger evidence than a populated index or a running Filebeat service because it follows one identifiable event through the pipeline.

For the public portfolio, I record the target as `%TEMP%\SOC-Project01-FileCreation-Test.txt` and omit the real username, hostname, private IP and full profile path. The local evidence is retained separately in the lab.

This is raw telemetry validation only. No custom Wazuh rule or alert was tested in this step.

## Related notes

- [03 - PowerShell Logging and Wazuh Collection](03%20-%20PowerShell%20Logging%20and%20Wazuh%20Collection.md)
- [04 - Validation Log](04%20-%20Validation%20Log.md)
- [06 - Project Status and Next Milestones](06%20-%20Project%20Status%20and%20Next%20Milestones.md)
- [Wazuh Architecture](../../../Cybersecurity/04-Security%20Monitoring/Wazuh%20Architecture.md)
- [Wazuh Operations Commands](../../../Commands/06%20-%20Wazuh%20Operations%20Commands.md)
