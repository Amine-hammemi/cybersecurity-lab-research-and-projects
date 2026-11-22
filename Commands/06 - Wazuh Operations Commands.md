---
tags:
  - cybersecurity/reference
  - wazuh
  - siem
  - command-reference
status: maintained
---

# Wazuh Operations Commands

A compact reference for the Wazuh-specific commands I have recorded while validating the server components and checking agent connectivity.

## Wazuh Manager, Indexer and Dashboard services

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `sudo systemctl status wazuh-manager` | Checks whether the Wazuh Manager service is running and shows recent status details. | Use `start`, `stop` or `restart` only when a service action is intended. Review logs before restarting a component. | [Wazuh SOC Lab — Manager validation](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md#9-validation-checklist-when-restarting-the-lab) · [Wazuh Architecture — Manager status](../Cybersecurity/04-Security%20Monitoring/Wazuh%20Architecture.md#7-what-happens-if-an-agent-disconnects) |
| `sudo systemctl status wazuh-indexer` | Checks the Wazuh Indexer service state. | The Indexer is responsible for storing/searching indexed data; check it separately from the Manager and Dashboard. | [Wazuh SOC Lab — Indexer validation](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md#9-validation-checklist-when-restarting-the-lab) |
| `sudo systemctl status wazuh-dashboard` | Checks the Wazuh Dashboard service state. | A working Dashboard does not, by itself, prove that the Manager is receiving agent telemetry or that the Indexer is healthy. | [Wazuh SOC Lab — Dashboard validation](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md#9-validation-checklist-when-restarting-the-lab) |

## Agent inventory and connectivity

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `sudo /var/ossec/bin/agent_control -lc` | Lists configured Wazuh agents and their connection status. | Useful for checking enrollment and recent connectivity from the Manager. It is a status snapshot, not a full endpoint health assessment. | [Wazuh Architecture — list agent status](../Cybersecurity/04-Security%20Monitoring/Wazuh%20Architecture.md#7-what-happens-if-an-agent-disconnects) |

## Windows-side companion checks

The Windows agent and Sysmon services are checked from PowerShell:

- [`Get-Service` and `Get-CimInstance` examples](04%20-%20PowerShell%20Commands.md#windows-services-and-service-to-process-mapping)
- [Wazuh SOC Lab — Windows agent and Sysmon checks](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md#9-validation-checklist-when-restarting-the-lab)

## Related notes

- [Wazuh Architecture](../Cybersecurity/04-Security%20Monitoring/Wazuh%20Architecture.md)
- [Wazuh SOC Lab](../Cybersecurity/04-Security%20Monitoring/Wazuh%20SOC%20Lab.md)
- [Linux Administration](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md)


## Raw archive collection and Filebeat validation

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `sudo /var/ossec/bin/wazuh-analysisd -t` | Tests the Wazuh Manager analysis configuration for syntax/configuration errors. | Run after editing the Manager configuration and before restarting. A successful test does not prove agent event delivery or Indexer ingestion. | [Project 01 — validate Manager archive configuration](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#1-enable-json-archives-on-the-manager) |
| `sudo filebeat test config -c /etc/filebeat/filebeat.yml` | Checks whether Filebeat can parse the specified configuration file. | `-c` selects the configuration path. `Config OK` means the configuration parses; it does not prove output connectivity or successful indexing. | [Project 01 — validate Filebeat configuration](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#2-enable-archive-indexing-in-filebeat) |
| `sudo filebeat test output` | Tests Filebeat's configured output connection, including host resolution, TCP, TLS and authentication. | Uses the configured output settings. A successful result proves the output endpoint is reachable/authenticated, not that a particular input event was indexed. | [Project 01 — verify Indexer connectivity](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#how-i-verified-each-stage) |
| `sudo systemctl restart filebeat` | Restarts Filebeat so it loads the updated configuration. | Restart only after backing up and validating the configuration. Check the service afterward with `systemctl is-active filebeat`. | [Project 01 — enable archive indexing](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#2-enable-archive-indexing-in-filebeat) |
| `sudo systemctl is-active filebeat` | Returns the current runtime state of Filebeat. | `active` means the service is running; it does not establish that events are being published or indexed. | [Project 01 — validate Filebeat service state](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#2-enable-archive-indexing-in-filebeat) |
| `sudo journalctl -u filebeat -n 50 --no-pager` | Reads the latest 50 systemd journal lines for Filebeat without opening a pager. | Add `--since '10 minutes ago'` to narrow by time. The journal may show only service lifecycle messages; Filebeat can also write its own application log. | [Project 01 — inspect Filebeat service logs](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md) |
| `sudo tail -n 40 /var/log/filebeat/filebeat` | Reads the last 40 lines of Filebeat's application log. | Use the path configured by `logging.files.path` and `logging.files.name`; log level and output configuration affect what appears. | [Project 01 — inspect Filebeat application logs](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#how-i-verified-each-stage) |
| `sudo grep -F 'archives.json' /var/lib/filebeat/registry/filebeat/log.json` | Searches Filebeat's registry for state entries associated with the archive input file. | `-F` treats the search text literally. Registry offsets show tracked file progress; they are not proof that every event reached the Indexer. Do not edit or delete registry files manually. | [Project 01 — verify Filebeat archive input state](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#how-i-verified-each-stage) |

| `sudo filebeat modules list` | Lists Filebeat modules known to the module-management command. | This command expects `filebeat.config.modules.path`. In this lab, modules were declared through the `filebeat.modules:` block in the main YAML, so the command returned a modules-path error. That error alone did not prove the Wazuh module was inactive. | [Project 01 — understand the installed Wazuh archives module](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#3-understand-what-the-installed-wazuh-module-reads-and-creates) |
## Important distinction

- `wazuh-analysisd -t` validates Manager configuration.
- `filebeat test config` validates Filebeat configuration syntax.
- `filebeat test output` validates the configured output connection.
- Index Management and Discover provide evidence that documents actually exist and can be searched.

These checks answer different questions; I should not treat one as proof of another.
