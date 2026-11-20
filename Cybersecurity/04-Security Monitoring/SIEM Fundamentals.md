# SIEM Fundamentals

> My notes on SIEM concepts before going deeper into Wazuh and SOC investigations.

## 1. What is a SIEM?

**SIEM = Security Information and Event Management**

A SIEM platform helps an organization:

- Collect security-related logs and events from multiple systems.
- Centralize and search the collected data.
- Normalize and parse different log formats.
- Correlate related events.
- Detect activity that matches defined detection logic.
- Generate alerts for analysts to investigate.

A SIEM supports security monitoring and investigation. It does not automatically prove that an attack happened.

## 2. Event vs alert

| Event | Alert |
|---|---|
| A recorded occurrence or activity | A detection result that requires attention |
| Can be normal or suspicious | Generated when alert conditions are met |
| May not trigger a detection rule | Is associated with detection logic |

Key idea:

- Not every event becomes an alert.
- An alert is based on event data or a set of related event data.
- An alert is an investigation lead, not proof of compromise.

## 3. Log collection and normalization

- Different systems produce logs in different formats.
- A SIEM needs to parse relevant information into structured fields.
- Structured fields make searching, filtering and detection easier.

Example of useful fields:

```text
timestamp
hostname
username
source_ip
destination_ip
process_name
event_id
```

The exact fields depend on the source and the event.

## 4. Correlation

Correlation means connecting related events to understand a bigger picture.

Example:

1. A user logs in.
2. A process starts.
3. The process launches PowerShell.
4. PowerShell creates a file.
5. The process makes a network connection.

Each event alone may have a legitimate explanation. Together, they may justify investigation.

## 5. Detection rules

- Detection rules define conditions that identify activity of interest.
- Rules can match a single event or combine conditions.
- A matching rule can generate an alert.
- Detection quality depends on the rule logic, telemetry quality and context.

A rule can produce:

- **True positive:** the alert correctly identifies activity of concern.
- **False positive:** legitimate activity triggers an alert.
- **False negative:** suspicious activity is not detected.

## 6. Severity and priority

- Alert severity helps prioritize investigation.
- A higher severity generally means the rule considers the activity more important.
- Severity is not the same as confidence.
- A high-severity alert can still be a false positive.

## 7. Typical SIEM workflow

1. Collect telemetry.
2. Parse and normalize the data.
3. Evaluate detection logic.
4. Generate alerts when conditions match.
5. Search and investigate the evidence.
6. Determine whether the activity is benign, suspicious or malicious.
7. Document findings and take an appropriate response.

## 8. Wazuh as my lab SIEM

I use Wazuh to centralize and investigate telemetry from my Windows endpoint.

My current sources include:

- Windows Event Logs.
- Sysmon Operational events.

The Wazuh components and data flow are documented in [Wazuh Architecture](Wazuh%20Architecture.md).

## 9. What I still need to practise

- Distinguishing events from alerts in real investigations.
- Understanding false positives and false negatives.
- Reading detection rules and their conditions.
- Correlating multiple events into a timeline.
- Writing a clear investigation summary.

## Related notes

- [Windows Event Logs](../03-Endpoint%20Security/Windows%20Event%20Logs.md)
- [Sysmon](../03-Endpoint%20Security/Sysmon.md)
- [Wazuh Architecture](Wazuh%20Architecture.md)
- [Wazuh SOC Lab](Wazuh%20SOC%20Lab.md)

## Further learning

- [Wazuh documentation](https://documentation.wazuh.com/current/)
- [Wazuh ruleset documentation](https://documentation.wazuh.com/current/user-manual/ruleset/index.html)
- [Microsoft security auditing](https://learn.microsoft.com/en-us/windows/security/operating-system-security/system-security-auditing/)
