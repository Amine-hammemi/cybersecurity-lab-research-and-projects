---
title: Project Status and Next Milestones
type: project-status
status: in-progress
---

# Project Status and Next Milestones

## Executive summary

**Project 01 is not finished.** The endpoint telemetry foundation is substantially configured and partially validated. The main detection-engineering and investigation work remains.

The current achievement is meaningful: Windows can generate Sysmon and PowerShell telemetry, the Wazuh Agent is configured to collect both channels, and raw Sysmon events are confirmed in the Wazuh archive index and searchable through Discover. On 2026-11-24, a controlled Sysmon Event ID 11 was correlated from the local Windows event through the Manager archive and into Discover, including its structured fields.

However, this is infrastructure and telemetry validation—not yet a complete detection-and-response project.

## Progress by phase

| Phase | Status | Notes |
|---|---|---|
| Lab architecture | Configured | Windows, Wazuh and Kali on `isolated-lab` |
| Wazuh stack | Configured / previously validated | Manager, Indexer and Dashboard running during lab work |
| Windows endpoint | Configured | Windows 11 VM |
| Sysmon | Locally validated | Event ID 1 records observed |
| PowerShell logging | Locally validated | Script Block Logging enabled; controlled 4104 marker found locally |
| Wazuh Sysmon collection | Validated | Sysmon provider events visible |
| Wazuh PowerShell collection | Validated | PowerShell provider events visible after agent restart |
| Wazuh Manager JSON archives | Validated | Windows events written to `archives.json` |
| Filebeat archive indexing | Validated | `archives.enabled: true`; archive index populated |
| Sysmon raw event in Discover | Validated | Event ID 13 and controlled Event ID 11 retrieved from `wazuh-archives-4.x-*` |
| Controlled Sysmon Event ID 11 | End-to-end validated | Exact event correlated by Event Record ID, Process GUID, Process ID and target filename |
| Exact 4104 marker in Wazuh archives | Not confirmed | Still needs retrieval from the archive data view |
| Controlled suspicious scenario | In progress / not yet complete | One benign file-creation test is validated; broader repeatable scenario remains |
| Custom Wazuh detection | Not started | Rule logic and tuning remain |
| Event correlation | Not started | Need process/script/file/task timeline |
| Incident report | Not started | Requires investigation evidence |

## What remains

### Priority 1 — Build Windows fundamentals
Complete the five-module foundation track in [05 - Investigation Methodology and Learning Plan](05%20-%20Investigation%20Methodology%20and%20Learning%20Plan.md).

### Priority 2 — Finish telemetry validation
- Generate a fresh controlled 4104 event.
- Search for the exact marker in Wazuh.
- Inspect the indexed 4104 document's actual fields.
- Record evidence and any configuration limitations.
- The controlled Sysmon Event ID 11 path is now validated end-to-end; see [04 - Validation Log](04%20-%20Validation%20Log.md) and [07 - Wazuh Raw Archive Collection and Indexing](07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md).

### Priority 3 — Design a controlled scenario
Create a benign simulation that exercises:
- PowerShell process creation.
- Script Block Logging.
- A harmless file creation.
- A child process launch, if needed.
- Optionally a scheduled task in a disposable test context, only after understanding Task Scheduler.

Do not download or execute untrusted scripts. Use scripts written for the lab and clearly identified test markers.

### Priority 4 — Detection engineering
- Identify the telemetry needed for each behaviour.
- Review existing Wazuh rules before writing custom ones.
- Create a narrowly scoped custom rule.
- Test both expected matches and benign lookalikes.
- Tune false positives and document limitations.

### Priority 5 — Investigation and reporting
- Reconstruct the process and script timeline.
- Identify user, parent process, command line, file paths, and persistence evidence if applicable.
- Separate observed facts from hypotheses.
- Reach a supported classification.
- Produce an incident report with evidence and confidence.

### Priority 6 — Portfolio packaging
- Add sanitized architecture and telemetry diagrams.
- Include screenshots and sample event fields with sensitive data redacted.
- Document setup, test procedure, detection logic, results, limitations, and lessons learned.
- Never claim that an alert or investigation was validated if it was not.

## Current overall assessment

The project is at the **telemetry foundation / pre-detection stage**.

A reasonable next move is to pause the attack scenario and complete the Windows fundamentals track. This directly addresses the knowledge gaps exposed during the first investigation exercises and will make later detection work more meaningful.


## Latest checkpoint — 2026-11-24

- A narrowly scoped Sysmon `FileCreate` include rule was added to the existing Sysmon Modular configuration after creating a backup.
- The XML parsed successfully, Sysmon accepted the updated configuration, and the active configuration showed the rule under Event ID 11's `FileCreate` filter.
- A fresh test file generated local Event ID 11. The same event was found in `archives.json` and in the `wazuh-archives` Discover data view.
- The event's Event Record ID, Process GUID, Process ID and target filename matched across endpoint and Manager evidence.
- This closes the raw-event collection/indexing checkpoint for this controlled Event ID 11. It does not mean a custom detection or full investigation is complete.
- [04 - Validation Log](04%20-%20Validation%20Log.md) records the evidence and troubleshooting. [07 - Wazuh Raw Archive Collection and Indexing](07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md) documents the archive/indexing pipeline.
