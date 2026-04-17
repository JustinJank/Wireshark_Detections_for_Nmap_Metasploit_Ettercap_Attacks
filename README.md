# Network Attack Simulation & SOC Analysis Lab (ARP Spoofing / MITM)

## Overview

This project demonstrates a complete end-to-end local network attack simulation in an isolated virtual lab environment. The objective was to perform reconnaissance, simulate a Man-in-the-Middle (MITM) attack using ARP spoofing, and analyze network behavior using packet inspection tools.

The lab also includes SOC-style defensive analysis, highlighting detection signals and mitigation strategies.

## Lab Environment

- Attacker VM: Ubuntu Linux  
- Target VM: Windows 11 (Sysmon & Wireshark enabled)  
- Network: Isolated / Internal Virtual Network (VirtualBox)  
- Analysis Tools: Wireshark, Sysmon, IDS script  

Tools Used:
- Nmap (network reconnaissance)
- Metasploit Framework (port scanning simulation)
- Ettercap (ARP spoofing / MITM attack)
- Wireshark (packet analysis)
- Sysmon (endpoint telemetry)

## Objectives

- Perform network reconnaissance and host discovery  
- Simulate port scanning activity using Nmap and Metasploit  
- Execute ARP spoofing MITM attack using Ettercap  
- Capture and analyze network traffic using Wireshark  
- Validate attack impact via Windows ARP cache  
- Identify SOC-level detection opportunities  

## Phase 1 — Network Reconnaissance (Nmap)

Commands:
nmap -sn 192.168.56.0/24  
nmap -sS -sV -p 1-1000 <target-ip>  

Evidence:
![Nmap Discovery](https://github.com/user-attachments/assets/2af90d6a-73ae-437a-b48b-2c6319c4cdd4)  
![Nmap SYN Scan](https://github.com/user-attachments/assets/a59c82ef-521c-4f69-bdb0-a90cb0f9eda4)  
![Wireshark SYN Capture](https://github.com/user-attachments/assets/47a5d421-e825-4fe2-a2e5-6038c2e05e21)

Observations:
- Active hosts identified in subnet  
- Open ports discovered via SYN scan  
- TCP SYN patterns visible in capture  

## Phase 2 — Metasploit Port Scanning

Commands:
msfconsole  
use auxiliary/scanner/portscan/tcp  
set RHOSTS <target-ip>  
set PORTS 1-1000  
run  

Evidence:
![Metasploit Config](https://github.com/user-attachments/assets/9535af41-bdf8-49a6-8825-f5c35840dbd2)  
![Metasploit Scan](https://github.com/user-attachments/assets/4ffe1a54-fe2b-43d0-b330-47725b598ea5)  
![Wireshark Detection]<img width="1898" height="1046" alt="Wireshark_Filters_Detecting_Metasploit_Scan_6" src="https://github.com/user-attachments/assets/147a9729-0d56-4170-82b1-89a098e1ee01" />


Observations:
- High-speed SYN scan behavior observed  
- Different fingerprint compared to Nmap  
- Increased TCP SYN packet volume visible  

## Phase 3 — ARP Spoofing (MITM Attack)

Steps:
- Enabled IP forwarding  
- Scanned for hosts  
- Selected target and gateway  
- Started ARP poisoning using Ettercap  

Evidence:
![Ettercap MITM](https://github.com/user-attachments/assets/82c40e35-ed6b-42c4-9027-504c863de237)

Observations:
- Gateway MAC address was spoofed  
- ARP reply frequency increased  
- Traffic routed through attacker system  

## Phase 4 — Packet Capture & Analysis (Wireshark)

Filters:
arp  
arp.opcode == 2  
tcp.flags.syn == 1 && tcp.flags.ack == 0  

Evidence:
![Wireshark MITM](https://github.com/user-attachments/assets/73d5ae0b-35c6-4c1b-bf02-4266eaec25c4)

Observations:
- ARP reply spikes during MITM  
- SYN scan patterns visible  
- IP-to-MAC inconsistencies detected  

## Phase 5 — Endpoint Validation (Windows)

Command:
arp -a  

<img width="1132" height="905" alt="FinalImageARPA_9" src="https://github.com/user-attachments/assets/206102b1-0cdc-425b-9026-7fbb45e00727" />

Observations:
- Gateway MAC changed after attack  
- ARP cache reflects spoofed mapping  
- Confirms MITM success  

## SOC Detection Analysis

ARP Anomalies:
- Multiple ARP replies from single MAC  
- IP-to-MAC inconsistencies  
- High ARP broadcast activity  

Network Scanning:
- High SYN packet volume without completed handshakes  
- Sequential port probing behavior  

MITM Indicators:
- Gateway MAC changes  
- Unexpected ARP “is-at” responses  
- Traffic redirection through attacker  

## MITRE ATT&CK Mapping

Reconnaissance:
T1595 - Active Scanning (Nmap)  
T1046 - Network Service Scanning  

Adversary-in-the-Middle:
T1557 - MITM via ARP spoofing  
T1557.002 - ARP Poisoning  

Discovery:
T1016 - Network configuration discovery via arp -a  

## Mitigation Strategies

- Enable Dynamic ARP Inspection (DAI)  
- Use static ARP entries for critical systems  
- Implement VLAN segmentation  
- Deploy IDS/IPS for ARP anomaly detection  

## Conclusion

This lab demonstrates a full attack chain:

- Network reconnaissance (Nmap / Metasploit)  
- Traffic interception (Ettercap ARP spoofing)  
- Packet analysis (Wireshark)  
- Endpoint validation (ARP inspection)  

This demonstrates how Layer 2 attacks operate and how they can be detected in enterprise SOC environments.

## Disclaimer

This project was conducted in a controlled, isolated virtual environment for educational and research purposes only.
