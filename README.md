#  Network Attack Simulation & SOC Analysis Lab (ARP Spoofing / MITM)

##  Overview

This project demonstrates a complete end-to-end local network attack simulation in an isolated virtual lab environment. The objective was to perform reconnaissance, simulate a Man-in-the-Middle (MITM) attack using ARP spoofing, and analyze network behavior using packet inspection tools.

The lab also includes defensive SOC style analysis, highlighting detection signals and mitigation strategies.

---

##  Lab Environment

- **Attacker VM:** Ubuntu Linux
- **Target VM:** Windows 11 (Sysmon & Wireshark enabled)
- **Network:** Isolated / Internal Virtual Network
- **Analysis Tooling:** Windows VM running Wireshark, Sysmon, & IDS script from another past project

### Tools Used:
- Nmap (reconnaissance & port scanning)
- Metasploit Framework (further port scanning)
- Ettercap (ARP spoofing / MITM attack(s))
- Wireshark (packet analysis)
- Sysmon (endpoint telemetry)

---

## Objectives

- Perform network reconnaissance and host discovery
- Simulate port scanning activity (Nmap / Metasploit)
- Execute ARP spoofing MITM attack (Ettercap)
- Capture and analyze network traffic (Wireshark)
- Identify detection signals from SOC perspective

---

# Phase 1 — Network Reconnaissance (Nmap)

Network discovery was performed to identify active hosts in the subnet.

### Command:
```bash
nmap -sn 192.168.56.0/24
Detailed scan:
nmap -sS -sV -p 1-1000 <Windows VM / target ip>
<img width="1873" height="987" alt="Wireshark_Baseline_Packet_Capture_1" src="https://github.com/user-attachments/assets/2af90d6a-73ae-437a-b48b-2c6319c4cdd4" />

Observations
Active hosts identified in isolated subnet
Open ports enumerated via SYN scan
TCP SYN patterns visible in Wireshark

<img width="1876" height="1000" alt="Wireshark_NMAP_Detections_via_filters_2" src="https://github.com/user-attachments/assets/a59c82ef-521c-4f69-bdb0-a90cb0f9eda4" />
<img width="1907" height="1051" alt="Wireshark_Capturing_SYN_Port_Scans_3" src="https://github.com/user-attachments/assets/47a5d421-e825-4fe2-a2e5-6038c2e05e21" />

---

### Phase 2 — Metasploit Port Scanning

Metasploit was utilized to simulate structured TCP scanning behavior.

Commands:
msfconsole
use auxiliary/scanner/portscan/tcp
set RHOSTS <Windows VM / target ip>
set PORTS 1-1000
run

<img width="1897" height="1035" alt="Filters_Displaying_Targetted_Ports_and_IPs_4" src="https://github.com/user-attachments/assets/4ffe1a54-fe2b-43d0-b330-47725b598ea5" />
<img width="1009" height="920" alt="Metasploit_Basic_Attacking_Configurations_5" src="https://github.com/user-attachments/assets/9535af41-bdf8-49a6-8825-f5c35840dbd2" />

# Observations
High speed structured SYN scan behavior observed
Distinctive scanning pattern(s) compared to traditional Nmap scans
Increased TCP SYN packet volume evidently visible in Wireshark

<img width="1898" height="1046" alt="Wireshark_Filters_Detecting_Metasploit_Scan_6" src="https://github.com/user-attachments/assets/8cf47f96-78d8-4f00-a90c-5716acf82f3f" />

---

### Phase 3 — ARP Spoofing (MITM Attack)

## ARP poisoning was performed using Ettercap to position the attacker as a Man-in-the-Middle to simulate a real world attack.

Ettercap Steps Performed:
Enabled IP forwarding on attacker VM
Scanned network for hosts
Selected target (Windows VM) and gateway
Initiated ARP poisoning attack

<img width="1900" height="1050" alt="Wireshark_Detecting_Ettercap_MITM_Attack_7" src="https://github.com/user-attachments/assets/82c40e35-ed6b-42c4-9027-504c863de237" />


# Observations
Gateway MAC address was spoofed
ARP reply frequency increased significantly
Victim traffic routed through attacker system

### Phase 4 — Packet Capture & Analysis (Wireshark)

Network traffic was captured and analyzed using Wireshark
Filters Used:
arp
arp.opcode == 2
tcp.flags.syn == 1 && tcp.flags.ack == 0

<img width="1900" height="1054" alt="MITM_ARP_MAC_Spoofing_Wireshark_Capture_8" src="https://github.com/user-attachments/assets/73d5ae0b-35c6-4c1b-bf02-4266eaec25c4" />

# Observations
ARP responses increased during MITM phase
SYN scan patterns clearly visible
IP-to-MAC inconsistencies detected during spoofing
Phase 5 — Endpoint Validation (Windows)

ARP cache was inspected before and after the attack.

Command: arp -a
Observations: Gateway MAC address changed after ARP poisoning
ARP cache reflected attacker controlled mapping
Confirms successful MITM positioning

### SOC Detection Analysis

From a defensive SOC perspective, the following indicators were observed:

# ARP Anomalies -
Multiple ARP replies from a single MAC address
IP-to-MAC mapping inconsistencies
Rapid ARP broadcast activity
# Network Scanning Behavior
High volume SYN packets without full TCP handshakes
Sequential port probing patterns
# MITM Indicators
Gateway MAC address changes
Unexpected ARP “is-at” responses
Traffic redirection through attacker node
# Detection Opportunities
Monitor ARP table changes over time
Detect duplicate IP-to-MAC mappings
Alert on excessive ARP reply rates
Correlate with abnormal SYN scanning activity
# Mitigation Strategies
Enable Dynamic ARP Inspection (DAI)
Use static ARP entries for critical systems
Implement network segmentation (VLANs)
Deploy IDS/IPS for ARP anomaly detection

## Conclusion

This lab demonstrated a full attack chain:

Network reconnaissance (Nmap / Metasploit)
Traffic interception setup (Ettercap ARP spoofing)
Packet-level analysis (Wireshark)
Endpoint validation (ARP cache inspection)

The exercise highlights how Layer 2 attacks operate and how they can be detected in real world enterprise SOC environments.

### Forewarning and Disclaimer-

All activities documented in this project were performed in a controlled, isolated virtual environment for educational and research purposes only. The environment was fully segregated from production systems and external networks.
