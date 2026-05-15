# Attack Simulation 01 — SMB Brute Force

## Objective

The objective of this simulation was to perform a brute force attack against the SMB service on NEXACORE-WS01 using Kali Linux, and to verify that the attack generates detectable evidence in Splunk through Windows Event ID 4625 failed login events.

## Attack Background

SMB (Server Message Block) is the protocol Windows uses to share files and printers across a network. It runs on port 445 and is one of the most commonly targeted services in real world attacks because it is open by default on most Windows machines and requires valid credentials to access. A brute force attack works by repeatedly trying different passwords against a target account until one succeeds. Every failed attempt generates a Windows Security Event ID 4625 which is a key detection signal for SOC analysts.

## MITRE ATT&CK Mapping

| Field | Detail |
|---|---|
| Tactic | Credential Access |
| Technique | Brute Force |
| Sub-technique | T1110.001 — Password Guessing |
| Reference | https://attack.mitre.org/techniques/T1110/001/ |

## Environment

| Role | Machine | IP Address |
|---|---|---|
| Attacker | Kali Linux | 192.168.10.20 |
| Target | NEXACORE-WS01 | 192.168.10.10 |
| SIEM | Splunk Enterprise | 192.168.56.1 |

## Tools Used

- smbclient — a Linux command line tool for interacting with SMB shares. Used here to repeatedly attempt authentication against NEXACORE-WS01 using a custom password list.

## Attack Steps

**Step 1 — Confirm SMB port is open on the target:**

Before launching the attack, netcat was used to verify port 445 was reachable from Kali. An open port confirms the attack surface exists.

```
nc -zv 192.168.10.10 445
```

![SMB Port Open](screenshots/03-smb-port-open.png)

**Step 2 — Create a password list:**

A small custom password list was created on Kali containing common weak passwords. In a real attack, an attacker would use a much larger wordlist such as rockyou.txt.

```
cat ~/passwords.txt
```

![Password List](screenshots/02-password-list-created.png)

**Step 3 — Launch the brute force attack:**

A bash loop was used to pass each password from the list to smbclient, attempting to authenticate as the administrator account on NEXACORE-WS01. Each failed attempt returned NT_STATUS_LOGON_FAILURE confirming the credentials were rejected by Windows.

```
for pass in $(cat ~/passwords.txt); do echo "Trying password: $pass"; smbclient -L //192.168.10.10 -U administrator%$pass 2>&1 | grep -E "NT_STATUS|session setup"; done
```

![Brute Force Attack Running](screenshots/00-brute-force-attack-smbclient.png)

## Detection

The attack was detected in Splunk using the following SPL query which searches for failed login events on NEXACORE-WS01 in the last 15 minutes:

```
index=main host=NEXACORE-WS01 EventCode=4625 earliest=-15m
```

The search returned 8 events corresponding exactly to the 8 password attempts made by the attacker.

![4625 Events Detected in Splunk](screenshots/01-brute-force-4625-splunk-detection.png)

## Investigation Findings

Each of the 8 Event ID 4625 events was expanded in Splunk and the following fields were examined to build a complete picture of the attack:

| Field | Value | Significance |
|---|---|---|
| Account_Name | administrator | The attacker targeted the most privileged account on the machine |
| Logon_Type | 3 | Network logon confirming the attempt came from a remote machine |
| Failure_Reason | Unknown user name or bad password | Wrong password on every attempt confirming brute force pattern |
| Source_Network_Address | 192.168.10.20 | Kali Linux IP identifying the attacker machine |
| Workstation_Name | KALI | Confirms the attack originated from the Kali VM |

The Account_Name field confirms the attacker was targeting the administrator account, which is the most privileged account on the machine. Targeting administrator is a common pattern in brute force attacks because gaining access to it gives full control of the system.

![Account Name](screenshots/02-4625-account-name.png)

The Logon_Type field shows a value of 3 which means the authentication attempt came over the network from a remote machine, not from someone sitting at the keyboard. This confirms the attack was launched remotely from Kali Linux.

![Logon Type](screenshots/03-4625-logon-type.png)

The Failure_Reason field shows Unknown user name or bad password on every single attempt. Seeing this message repeated 8 times from the same source IP is a clear indicator of automated brute force activity rather than a genuine user mistyping their password.

![Failure Reason](screenshots/04-4625-failure-reason.png)

The Source_Network_Address field shows 192.168.10.20 which is the IP address of the Kali Linux attacker machine. This field is critical during investigation because it tells the analyst exactly where the attack originated from.

![Source IP](screenshots/05-4625-source-ip.png)

The Workstation_Name field shows KALI, confirming the machine name of the attacker. Combined with the source IP, this gives the analyst two independent pieces of evidence pointing to the same attacker machine.

![Workstation Name](screenshots/06-4625-workstation-name.png)
The complete detection table showing all five fields across all events was generated using the following query:

```
index=main EventCode=4625 Source_Network_Address=192.168.10.20 | table _time, Account_Name, Logon_Type, Failure_Reason, Source_Network_Address, Workstation_Name
```

![Detection Table](screenshots/07-brute-force-detection-table.png)

## Remediation and Prevention

**Account lockout policy** — Configure Windows to lock an account after a set number of failed login attempts. This directly stops brute force attacks by making repeated guessing impractical.

**Disable SMB where not needed** — If SMB file sharing is not required on a machine, disable it entirely to remove the attack surface.

**Restrict SMB access** — Use firewall rules to limit which machines can reach port 445. Only machines that legitimately need SMB access should be allowed.

**Use strong passwords** — All accounts especially administrator should use long complex passwords that are not in common wordlists.

**Monitor Event ID 4625** — Create a Splunk alert that fires when more than 5 failed logins occur from the same source IP within 5 minutes. This gives the SOC team real time visibility of brute force activity.

**Disable the built-in Administrator account** — Rename or disable the default Administrator account and create a separate admin account with a non-obvious name. This makes credential guessing significantly harder.
