---
title: Investigation Methodology and Learning Plan
type: learning-plan
status: in-progress
---

# Investigation Methodology and Learning Plan

## What I learned about investigation

A SOC analyst should not label an event malicious based only on a process name, command, directory, or filename.

For example:
- `explorer.exe` launching PowerShell can be normal in an interactive user session.
- `Get-ComputerInfo` is a legitimate administrative command.
- A `.ps1` file is a PowerShell script, not automatically malware.
- `AppData\Roaming` is a legitimate user-profile data location, but unexpected executable content there deserves investigation.
- A scheduled task can be legitimate automation or a persistence mechanism.

The analyst must examine the complete context:
- Parent and child process relationship.
- User and logon context.
- Command line and script content.
- File path, origin, hash, and behaviour.
- Task/service/registry configuration.
- Timing and sequence of events.
- Whether the activity was authorized and expected.
- Related network and endpoint telemetry.

## A practical investigation workflow

1. **Triage:** identify the event, affected endpoint, user, timestamp, and initial reason for concern.
2. **Validate:** confirm the event is genuine and understand what the telemetry field means.
3. **Scope:** identify related processes, files, users, and events.
4. **Correlate:** build a timeline using timestamps and stable identifiers where possible.
5. **Assess:** compare benign and malicious hypotheses.
6. **Conclude:** classify the activity only when evidence supports the conclusion.
7. **Document:** record evidence, gaps, confidence, and recommended actions.

## Learning gap identified

During the first investigation exercises, I struggled to reason about Windows paths, PowerShell scripts, AppData, scheduled tasks, and persistence.

The issue is not necessarily a lack of analytical ability. I need a stronger Windows foundation before attempting more complex SOC investigations.

## Foundation track before resuming the investigation scenario

### Module 1 — Windows filesystem
- Drive roots and directory structure.
- `C:\Windows`, `System32`, `Program Files`, `ProgramData`, `Users`.
- User profile and `AppData`.
- `Local`, `LocalLow`, `Roaming`.
- File extensions, file properties, ACLs and permissions.

### Module 2 — Processes and execution
- EXE versus DLL.
- Process versus service.
- Parent/child process relationships.
- PID, process lifetime and PID reuse.
- Command line, user context and integrity level.
- Process GUID and Sysmon correlation.

### Module 3 — PowerShell
- PowerShell host and engine.
- Cmdlets, parameters, pipelines and objects.
- `.ps1`, `.psm1`, `.psd1`.
- Script execution and execution policy.
- Script Block Logging and Event ID 4104.
- Difference between script content and command output.

### Module 4 — Persistence and automation
- Task Scheduler concepts.
- Task triggers, actions, principals and conditions.
- Windows services.
- Registry Run keys.
- Startup folders.
- How to verify creation and execution using Windows logs and Sysmon.

### Module 5 — Windows security telemetry
- Event Viewer and event channels.
- Windows Security auditing.
- Sysmon configuration and event IDs.
- Wazuh collection, indexing, rules and alerts.
- Correlation and false-positive assessment.

## Learning method

For each concept:
1. Understand the purpose and normal behaviour.
2. Inspect it directly in the Windows VM.
3. Perform a harmless controlled experiment.
4. Observe the resulting telemetry.
5. Explain the result in my own words.
6. Apply it to a small investigation scenario.

I should not memorize suspicious filenames or commands without understanding their normal purpose and context.
