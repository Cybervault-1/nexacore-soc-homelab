# Infrastructure

## Host Machine

The homelab was built on a Windows laptop running an Intel Core i7 processor with 16GB of RAM, using VirtualBox to host all virtual machines. RAM was allocated based on the role of each machine: Splunk Enterprise on the host uses 4GB, Windows 10 was allocated 3GB to run stably with Sysmon and the forwarder running simultaneously, and Kali Linux was allocated 2GB since Linux requires fewer resources.

## Virtual Machines

The lab contains three virtual machines. Windows 10 serves as the primary target endpoint, Windows Server acts as the Domain Controller managing authentication and Active Directory services for the NexaCore domain, and Kali Linux serves as the attacker machine for simulating real world attacks.

| Machine | Role | Host-Only IP | Internal Network IP |
|---|---|---|---|
| WIN10 | Primary target endpoint | 192.168.56.30 | 192.168.10.10 |
| DC01 | Domain Controller | 192.168.56.10 | 192.168.10.1 |
| Kali Linux | Attacker machine | N/A | 192.168.10.20 |
| Host Laptop | Splunk SIEM | 192.168.56.1 | N/A |

## Network Configuration

Network segmentation was configured using three VirtualBox adapter types. Windows 10 and the Domain Controller each use a Host-Only adapter to communicate with Splunk Enterprise on the host laptop at 192.168.56.1, and an Internal Network adapter named NexaCoreNet for isolated communication between lab machines. Kali Linux uses a NAT adapter for internet access to download attack tools and an Internal Network adapter to reach the Windows machines during attack simulations. The Host-Only network uses the 192.168.56.0/24 range while the Internal Network uses 192.168.10.0/24.

## Sysmon

Sysmon was installed on the Windows 10 endpoint to enrich Windows event logs with detailed telemetry including process creation, network connections, PowerShell execution and file system changes. Without Sysmon, Windows logs alone would not provide enough detail for realistic SOC investigation.

## Splunk Universal Forwarder

The Splunk Universal Forwarder was installed on both Windows 10 and the Domain Controller to collect and ship security events to Splunk Enterprise. Logs travel from the endpoints through the forwarder over port 9997 into Splunk where they are indexed, searched and analysed for threat detection and incident investigation.
