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
| Sysmon event visibility in Wazuh alerts/search | Validated |
| Raw Sysmon archive indexing | Validated |
| PowerShell provider visibility in Wazuh | Validated |
| Exact PowerShell 4104 marker end-to-end | Not confirmed |
| Custom alert generation | Not started |
| Full incident investigation | Not started |


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
