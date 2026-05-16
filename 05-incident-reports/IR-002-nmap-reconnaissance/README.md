# IR-002 — Nmap Reconnaissance

## Incident Summary

| Field | Detail |
|---|---|
| Incident ID | IR-002 |
| Incident Title | Network Port Scan via Nmap |
| Date | 16 May 2026 |
| Severity | Medium |
| Status | Resolved |
| Affected Host | NEXACORE-WS01 (192.168.10.10) |
| Attacker Machine | Kali Linux (192.168.10.20) |
| MITRE ATT&CK | T1046 — Network Service Discovery |

## What Happened

On 16 May 2026, an nmap port scan was executed from the attacker machine at 192.168.10.20 against the target endpoint NEXACORE-WS01 at 192.168.10.10. The scan targeted ports 1 through 1000 using service version detection. The activity was captured in Splunk via Windows Filtering Platform connection logs and flagged due to the high volume of connection attempts from a single source IP within a short time window.

## Timeline

| Time | Event |
|---|---|
| Scan initiated | Kali Linux ran nmap -sV -p 1-1000 192.168.10.10 |
| During scan | 37 Event ID 5156 entries generated on NEXACORE-WS01 |
| Post scan | Splunk query confirmed scanning pattern from 192.168.10.20 |
| Detection confirmed | Analyst reviewed results and identified port scan behaviour |

## What Was Discovered

The scan revealed three open ports on NEXACORE-WS01:

| Port | Service | Significance |
|---|---|---|
| 445 | SMB | Primary target for lateral movement and exploitation |
| 135 | RPC | Used for remote procedure calls and service enumeration |
| 139 | NetBIOS | Legacy Windows networking service |

Port 445 received 31 of the 37 total connection attempts, indicating the attacker was particularly interested in the SMB service. This aligns with the previous incident IR-001 where SMB was used to conduct a brute force attack against the administrator account.

## Impact Assessment

No data was accessed, no credentials were compromised, and no changes were made to the system during this incident. The nmap scan is a passive reconnaissance activity. Its impact is informational — the attacker now has a map of open services on NEXACORE-WS01 which could inform future attack attempts.

## How It Was Detected

Splunk detected the scanning activity through Windows Filtering Platform logs using the following query:

```spl
index=main EventCode=5156 
| stats count by Source_Address, Dest_Port 
| where count > 5
| sort - count
```

The query surfaced 37 connection attempts from 192.168.10.20 across ports 445, 135, and 139 within seconds of each other, confirming the pattern of a port scan.

## Evidence

Screenshot: `ir-002-nmap-splunk-detection.png`

*Splunk results showing 37 Event ID 5156 entries from 192.168.10.20 against NEXACORE-WS01 across three ports.*

## Response Actions

| Action | Detail |
|---|---|
| Detection confirmed | Reviewed Event ID 5156 logs in Splunk and confirmed scanning pattern |
| Source identified | Attacker machine confirmed as Kali Linux at 192.168.10.20 |
| Scope assessed | Scan limited to ports 1 through 1000 on a single host |
| No escalation required | Lab environment, incident contained and documented |

## Recommendations

In a production environment the following steps would follow this detection. First, block or alert on repeated outbound connection attempts from unrecognised internal IPs. Second, review whether SMB port 445 needs to be exposed on the network or whether access can be restricted to authorised hosts only. Third, correlate this reconnaissance event with any subsequent authentication attempts since a scan followed by a login attempt is a strong indicator of targeted attack activity.

## Lessons Learned

Reconnaissance does not cause immediate damage but it should never be ignored. The pattern of an internal machine scanning another internal machine is abnormal and warrants investigation in any environment. In this case the scan directly preceded the SMB brute force documented in IR-001, which is a classic attack progression from discovery to exploitation.
