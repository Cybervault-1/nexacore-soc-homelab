# Detection 02 — Nmap Reconnaissance (Network Port Scan)

## Detection Summary

| Field | Detail |
|---|---|
| Detection Name | Nmap Network Port Scan |
| MITRE ATT&CK Technique | T1046 — Network Service Discovery |
| Event Source | Windows Filtering Platform |
| Event ID | 5156 |
| Severity | Medium |
| Date Detected | 16 May 2026 |
| Affected Host | NEXACORE-WS01 (192.168.10.10) |
| Attacker Machine | Kali Linux (192.168.10.20) |

## What This Detection Catches

This detection identifies network port scanning activity where a single source IP generates an abnormally high number of connection attempts across multiple destination ports in a short time window. Port scanning is typically the reconnaissance phase of an attack where an adversary maps open services before deciding how to proceed.

## Detection Logic

The detection relies on Windows Filtering Platform connection logs captured via Event ID 5156. During the nmap scan, 37 events were generated from 192.168.10.20 within a matter of seconds. The following ports were hit:

| Destination Port | Service | Hit Count |
|---|---|---|
| 445 | SMB | 31 |
| 135 | RPC | 3 |
| 139 | NetBIOS | 3 |

A legitimate machine does not contact 1000 ports on a neighbour machine within seconds. That pattern is the indicator.

## Splunk Detection Query

```spl
index=main EventCode=5156 
| stats count by Source_Address, Dest_Port 
| where count > 5
| sort - count
```

**What each part does:**

- `EventCode=5156` filters to Windows Filtering Platform connection events only
- `stats count by Source_Address, Dest_Port` groups results by which IP hit which port and how many times
- `where count > 5` filters out normal low-frequency traffic and surfaces only repeated hits
- `sort - count` brings the highest hit counts to the top for faster triage

## Evidence from Splunk

Screenshot: `detection-02-nmap-splunk-results.png`

*The query results showing 192.168.10.20 hitting port 445 thirty-one times, confirming active port scanning behaviour.*

## Why This Matters

Reconnaissance is the step that happens before an attack. An adversary who has scanned your environment now knows which ports are open and which services are running. In this lab, the scan revealed SMB on port 445, RPC on port 135, and NetBIOS on port 139. These are exactly the services an attacker would target for lateral movement or exploitation. Detecting the scan early gives a defender the opportunity to investigate and respond before the next phase begins.

## Audit Policy Required

For Event ID 5156 to be captured, the following audit policy must be enabled on the target machine:

**Computer Configuration > Windows Settings > Security Settings > Advanced Audit Policy > System > Audit Filtering Platform Connection — set to Success**

Without this policy, the events will not appear in the Windows Security log and the detection will not work.

## MITRE ATT&CK Mapping

| Field | Value |
|---|---|
| Tactic | Discovery |
| Technique | T1046 — Network Service Discovery |
| Platform | Windows |
| Data Source | Windows Filtering Platform |
