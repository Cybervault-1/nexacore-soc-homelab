# Detection 01 — SMB Brute Force (Event ID 4625)

## Overview

This detection identifies brute force authentication attempts against Windows machines by monitoring for a high volume of failed login events (Event ID 4625) originating from the same source IP within a short timeframe. It was built and validated against the SMB brute force simulation documented in [Attack Simulation 01](../../03-attack-simulations/sim-01-smb-brute-force/README.md).

## MITRE ATT&CK Mapping

| Field | Detail |
|---|---|
| Tactic | Credential Access |
| Technique | Brute Force |
| Sub-technique | T1110.001 — Password Guessing |
| Reference | https://attack.mitre.org/techniques/T1110/001/ |

## Detection Logic

A single failed login is normal. A user mistyping their password generates one or two Event ID 4625 entries and then successfully logs in. What is not normal is seeing the same account targeted repeatedly from the same external IP address within seconds, with no successful login at any point.

That pattern is what this detection looks for. The moment multiple 4625 events appear from one source IP in a short window, a SOC analyst should treat it as a brute force attempt until proven otherwise.

## SPL Detection Query

This query searches for failed login events on NEXACORE-WS01 in the last 15 minutes. In a production SOC environment this query would be saved as a scheduled search, running silently in the background every 5 minutes and only triggering an alert when more than 5 failed logins are detected from the same source IP within that window.

    index=main host=NEXACORE-WS01 EventCode=4625 earliest=-15m

**What each part does:**

| Part | Meaning |
|---|---|
| index=main | Opens the main log storage bucket where all endpoint logs are kept |
| host=NEXACORE-WS01 | Filters results to only show events from the target endpoint |
| EventCode=4625 | Windows Security event for a failed logon attempt |
| earliest=-15m | Only shows events from the last 15 minutes to focus on recent activity |

## Investigation Query

Once the detection fires, the following query builds a clean table showing all relevant fields across every failed login event from the attacker IP. This gives the analyst a complete picture of the attack in one view.

    index=main EventCode=4625 Source_Network_Address=192.168.10.20 | table _time, Account_Name, Logon_Type, Failure_Reason, Source_Network_Address, Workstation_Name

## Key Fields To Examine

| Field | What To Look For |
|---|---|
| Account_Name | Is the attacker targeting a privileged account like administrator? |
| Logon_Type | Type 3 means the attempt came remotely over the network |
| Failure_Reason | The same failure reason repeated many times confirms automated activity |
| Source_Network_Address | The attacker IP — use this to block and trace the source |
| Workstation_Name | The attacker machine name — provides a second piece of identifying evidence |

## Detection Results

When this query was run during the SMB brute force simulation it returned 8 Event ID 4625 events, all originating from Kali Linux at 192.168.10.20, all targeting the administrator account, all within a 2 second window. No successful login was recorded at any point during the attack.

![Detection Results](../../03-attack-simulations/sim-01-smb-brute-force/screenshots/01-brute-force-4625-splunk-detection.png)

## Alert Recommendation

In a production SOC environment this detection should be configured as an automated Splunk alert with the following thresholds:

**Trigger condition:** More than 5 Event ID 4625 events from the same source IP within 5 minutes

**Severity:** High

**Recommended response:** Immediately block the source IP, check for any successful logins from the same IP across all machines, and open an incident ticket for investigation.

## Limitations

This detection only covers failed logins on NEXACORE-WS01. A more complete detection would cover all hosts in the environment and would also alert on successful logins following a series of failures, which could indicate the brute force succeeded.
