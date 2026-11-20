---
title: Project Scope and Architecture
type: project-note
status: in-progress
---

# Project Scope and Architecture

## Goal

Create a practical Windows endpoint monitoring and investigation project that demonstrates a basic SOC workflow.

The initial investigation theme is suspicious PowerShell activity. The project should eventually let me detect and investigate a sequence such as:
- A process launches PowerShell.
- PowerShell executes script content.
- The script creates a file or downloads another script.
- The activity may establish persistence or launch another process.

The scenario must be controlled and safe inside my isolated lab.

## Architecture

```text
Windows 11 endpoint
  ├── Windows Event Logs
  ├── Sysmon
  └── PowerShell Operational channel
          |
          v
      Wazuh Agent
          |
          | TCP 1514 (agent event traffic)
          v
      Wazuh Manager
          |
          v
      Wazuh Indexer
          |
          v
      Wazuh Dashboard
```

The Wazuh Agent collects only the event channels configured in its `ossec.conf`. Seeing an event locally in Windows does not automatically mean Wazuh is collecting it.

The Manager evaluates events against decoders and rules. Not every collected event becomes an alert. Alert visibility and raw-event visibility are different; raw event access depends on the Wazuh archive/indexing configuration.

## Why use both Sysmon and PowerShell logging?

They provide different perspectives:

- **Sysmon Event ID 1:** process creation details, including image, command line, user, parent process and process identifiers.
- **PowerShell Event ID 4104:** script-block content processed by the PowerShell engine, when Script Block Logging is enabled.

Event ID 4104 is not a general command-output log. It records script content, not necessarily the output returned by commands.

Combining the sources can help answer:
- Which process started PowerShell?
- Under which user?
- What script content did PowerShell process?
- What happened before and after the script execution?

For stronger correlation, use timestamps and available process identifiers. Sysmon Process GUID is generally more reliable for identifying a process instance than PID alone because PIDs can be reused.

## Scope boundaries

Initial phase:
- Local activity on Windows 11.
- Sysmon process telemetry.
- PowerShell Script Block Logging.
- Wazuh collection and search.

Later phases:
- Controlled file creation and process launch.
- Scheduled-task persistence simulation.
- Wazuh custom detection rules.
- Correlation and incident timeline.
- Optional remote activity from Kali, only after a safe remote execution mechanism is deliberately configured.

Kali is not necessary to establish the first local baseline.
