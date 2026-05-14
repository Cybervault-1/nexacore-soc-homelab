# Lab Architecture

## Overview

The NexaCore SOC homelab runs on a Windows laptop using VirtualBox, with Splunk Enterprise installed directly on the host machine at 192.168.56.1 to act as the central SIEM for the entire environment.

## Virtual Machines and Roles

The lab contains three virtual machines. Windows 10 at 192.168.56.30 on the Host-Only network and 192.168.10.10 on the Internal Network serves as the primary target endpoint, representing an employee workstation in a corporate environment. The Windows Server Domain Controller at 192.168.56.10 and 192.168.10.1 manages user authentication and Active Directory services for the NexaCore domain. Kali Linux at 192.168.10.20 serves as the attacker machine, used to simulate real world attacks against the Windows environment.

## Network Design

Network segmentation was achieved using two VirtualBox adapter types. The Host-Only adapter connects WIN10 and DC01 directly to the host machine, allowing the Splunk Universal Forwarder on each machine to ship logs to Splunk Enterprise. The Internal Network adapter connects all three VMs together in an isolated environment named NexaCoreNet, allowing attack simulation while preventing exposure to the real network. Kali uses a NAT adapter for internet access to download attack tools.

## Log Forwarding

The Splunk Universal Forwarder is installed on both WIN10 and DC01 because both machines generate critical security events including authentication logs, Sysmon activity and Active Directory events that are essential for detection and investigation.

## Network Diagram

| Machine | Role | Host-Only IP | Internal Network IP |
|---|---|---|---|
| Host Laptop | Splunk SIEM | 192.168.56.1 | N/A |
| WIN10 | Primary endpoint | 192.168.56.30 | 192.168.10.10 |
| DC01 | Domain Controller | 192.168.56.10 | 192.168.10.1 |
| Kali Linux | Attacker | N/A | 192.168.10.20 |
