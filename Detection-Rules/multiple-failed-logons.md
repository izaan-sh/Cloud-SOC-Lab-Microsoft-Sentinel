# Multiple Failed Windows Logons

## Detection Overview

This detection identifies repeated failed Windows authentication
attempts originating from the same source IP within a short
time window.

The purpose of the detection is to identify potential
brute-force or repeated authentication activity against the
Windows system.

## Data Source

- Microsoft Sentinel
- Log Analytics Workspace
- Windows Security Events

## Event ID

`4625` — An account failed to log on.

## Detection Threshold

- **20 or more failed logon attempts**
- **Same source IP**
- **5-minute window**

## Detection Logic

The detection groups Windows Event ID 4625 events by source IP
within five-minute windows.

When the number of failed authentication attempts reaches the
configured threshold, Microsoft Sentinel generates an incident
for investigation.

## KQL

See:

[`../KQL/multiple-failed-logons-detection.kql`](../KQL/multiple-failed-logons-detection.kql)

## Investigation

When an alert is generated, the following information is
reviewed:

- Source IP address
- Number of failed attempts
- First and last observed attempt
- Targeted accounts
- Authentication timeline
- Whether successful authentication occurred
- Whether the activity is still ongoing

## Incident Response

Detected source IP addresses can be passed to the automated
response workflow documented in:

[`../Automation/SOC-AutoBlock-Malicious-IP.md`](../Automation/SOC-AutoBlock-Malicious-IP.md)

## Validation

The detection was validated using controlled authentication
activity against the Windows VM. Repeated failed authentication
attempts generated the expected Sentinel incident.