---
title: Project Status and Next Milestones
type: project-status
status: in-progress
---

# Project Status and Next Milestones

## Executive summary

**Project 01 is not finished.** The endpoint telemetry foundation is substantially configured and partially validated. The main detection-engineering and investigation work remains.

The current achievement is meaningful: Windows can generate Sysmon and PowerShell telemetry, the Wazuh Agent is configured to collect both channels, and raw Sysmon events are now confirmed in the Wazuh archive index and searchable through Discover.

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
| Sysmon raw event in Discover | Validated | Event ID 13 retrieved from `wazuh-archives-4.x-*` |
| Exact 4104 marker in Wazuh archives | Not confirmed | Need to retrieve the controlled marker from the archive data view |
| Controlled suspicious scenario | Not started | Needs safe, repeatable simulation |
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


## Latest checkpoint

- [07 - Wazuh Raw Archive Collection and Indexing](07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md) documents the root cause, configuration changes, commands, validation evidence and remaining limits.
