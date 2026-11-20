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
- Whether raw PowerShell events are stored in Wazuh archives or a raw-event index.
- A custom Wazuh detection rule for the target behaviour.
- A complete controlled suspicious-activity simulation.
- Correlation across Sysmon, PowerShell, Security and Task Scheduler telemetry.
- A complete investigation timeline.
- A final incident report.

## Evidence handling

When capturing screenshots or exporting event data for a public portfolio:
- Remove private IP addresses if they expose the home network or sensitive topology.
- Remove MAC addresses, usernames where appropriate, hostnames if sensitive, tokens, credentials, and personal paths.
- Do not publish the full unredacted agent configuration.
- Record timestamps, event IDs, relevant fields, and the exact validation method.
- Keep screenshots focused on the evidence needed to support the claim.

## Current evidence maturity

| Capability | Current status |
|---|---|
| Sysmon installation and local telemetry | Validated |
| PowerShell Script Block Logging | Validated locally |
| Sysmon event visibility in Wazuh | Validated |
| PowerShell provider visibility in Wazuh | Validated |
| Exact test marker end-to-end | Not confirmed |
| Custom alert generation | Not started |
| Full incident investigation | Not started |
