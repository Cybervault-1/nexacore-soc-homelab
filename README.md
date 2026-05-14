# NexaCore SOC Homelab

## About Me

My name is Adedeji Adetayo and I am an aspiring SOC analyst currently building hands-on experience by designing and operating a cybersecurity homelab from scratch, simulating real attack scenarios and detecting them using industry standard tools. My goal is to develop the practical skills needed to work in a SOC environment and contribute meaningfully from day one.

## Lab Overview

The NexaCore SOC Homelab is a fully functional security operations environment built from scratch on VirtualBox. It runs three virtual machines: Kali Linux as the attacker, Windows 10 as the primary target endpoint, and a Windows Server Domain Controller that manages user credentials and the domain. Windows 10 is the main focus of attack simulations because in real enterprises attackers target employee workstations first before attempting to reach critical infrastructure like domain controllers. The Splunk Universal Forwarder is installed on both Windows 10 and the Domain Controller to collect security events and ship them to Splunk Enterprise running on the host machine, where logs are aggregated, searched and analysed. The purpose of this lab is to simulate realistic attack scenarios and demonstrate how a SOC analyst detects, investigates and responds to them.

## Tools and Technologies

- Splunk Enterprise
- Splunk Universal Forwarder
- Sysmon
- Kali Linux
- Windows 10
- Windows Server 2019
- VirtualBox

## Skills Demonstrated

- Attack simulation and threat detection
- Windows event log analysis
- SIEM log ingestion and SPL querying
- Incident investigation and reporting
- Network segmentation and lab architecture design

## Attack and Detection Coverage

| Attack Simulation | Detection Method | Status |
|---|---|---|
| SMB Brute Force | Event ID 4625 via Splunk | Completed |

## Certifications

- Google Cybersecurity Certificate

## Status

Actively building. New attack simulations and detection cases are added regularly.
