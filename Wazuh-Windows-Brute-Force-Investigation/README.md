# Windows Brute-Force Detection & Investigation with Wazuh

## Overview

This project demonstrates a controlled SOC investigation of repeated Windows authentication failures using Wazuh SIEM and Windows Security Event Logs.

The objective was to generate failed authentication activity on a Windows 11 endpoint, verify the underlying Windows events, observe how Wazuh detected the activity, investigate why the initial correlation rule did not trigger, and successfully generate a higher-severity brute-force detection.

> This project was performed in an isolated personal cybersecurity lab. All authentication attempts were intentionally generated for educational and detection-testing purposes.

---

## Lab Environment

| Component | Configuration |
|---|---|
| SIEM | Wazuh 4.14.7 |
| Endpoint | Windows 11 |
| Virtualization | VirtualBox |
| Wazuh Agent | SOC-LAB-WIN01 |
| Windows Logging | Windows Security Event Log |
| Network | Isolated virtual lab network |
| Framework | MITRE ATT&CK |

---

## Investigation Objective

The investigation focused on identifying and analyzing repeated failed Windows logon attempts.

The primary events and detections investigated were:

- Windows Security Event ID 4625
- Wazuh Rule 60122
- Wazuh Rule 60204
- MITRE ATT&CK T1110 - Brute Force

---

## Phase 1 - Windows Authentication Telemetry

Controlled incorrect password attempts were generated against a local Windows account.

Windows recorded the activity as:

**Event ID:** 4625  
**Description:** An account failed to log on

Multiple authentication failures occurred within a short period of time.

The events were first verified directly through Windows Security logs using PowerShell and Event Viewer.

This established that the endpoint itself was successfully recording the authentication activity before investigating the SIEM.

---

## Phase 2 - Wazuh Detection

The Wazuh agent forwarded Windows Security telemetry from the endpoint to the Wazuh server.

Individual authentication failures generated:

**Wazuh Rule:** 60122  
**Severity:** Level 5  
**Description:** Logon Failure - Unknown user or bad password

Multiple Rule 60122 alerts confirmed that Wazuh was successfully receiving and analyzing Windows Event ID 4625.

---

## Phase 3 - Detection Engineering / Troubleshooting

The initial failed-login simulation generated multiple Rule 60122 alerts but did not trigger the expected Rule 60204 correlation alert.

Instead of assuming the detection was broken, I reviewed the Wazuh Windows security ruleset.

The Rule 60204 configuration showed that Wazuh correlated authentication failures using:

- A frequency threshold
- A 240-second timeframe
- Matching source IP addresses

The ruleset also defined:

`MS_FREQ = 8`

This explained why the original test did not meet the correlation threshold.

A dedicated local test account named `SOC-Test` was then created so additional controlled authentication testing could be performed without using the primary lab account.

---

## Phase 4 - Brute-Force Detection

A second controlled authentication test was performed against the `SOC-Test` account.

After the configured threshold was reached, Wazuh generated:

**Rule ID:** 60204  
**Rule Level:** 10  
**Description:** Multiple Windows Logon Failures  
**Frequency:** 8

The underlying Windows telemetry showed:

- Event ID: 4625
- Target account: SOC-Test
- Logon Type: 2
- Source IP: 127.0.0.1
- Authentication package: Negotiate
- Logon process: User32

Logon Type 2 indicated an interactive logon, while the loopback address `127.0.0.1` indicated that the activity originated locally on the endpoint.

---

## MITRE ATT&CK Mapping

Wazuh mapped the correlated activity to:

**Technique:** T1110 - Brute Force  
**Tactic:** Credential Access

The behavior demonstrated how repeated authentication failures can be correlated by a SIEM into a higher-severity security detection.

---

## Investigation Conclusion

The activity was determined to be an **authorized security test** performed within an isolated personal SOC lab.

No evidence indicated an actual external compromise.

The investigation demonstrated the complete detection workflow:

Windows authentication failure  
→ Event ID 4625  
→ Wazuh agent collection  
→ Rule 60122 authentication alerts  
→ Correlation analysis  
→ Rule 60204 Level 10 detection  
→ MITRE ATT&CK T1110 mapping  
→ Analyst investigation and disposition

---

## Skills Demonstrated

- SIEM monitoring with Wazuh
- Windows Security Event analysis
- Authentication log investigation
- Alert triage
- Event correlation
- Detection rule analysis
- Wazuh XML ruleset analysis
- MITRE ATT&CK mapping
- PowerShell log analysis
- SOC investigation methodology
- False-positive / benign-positive identification
- Technical documentation

---

## Key Takeaway

One of the most valuable parts of this lab occurred when the expected correlation alert did not appear in the Wazuh dashboard as an EVENT 60204.

By examining the Wazuh detection rules rather than simply repeating the test, I identified the configured frequency and timeframe requirements and adjusted the controlled simulation accordingly.

This demonstrated the importance of understanding detection logic and validating SIEM alerts against the underlying endpoint telemetry.
