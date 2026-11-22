<div align="center">

# Red Team Cyber Security - Assignment 1
### Foundations, Reconnaissance, Service Enumeration & Risk Reporting

[![Course](https://img.shields.io/badge/Course-ITsimpleara%20Red%20Team-0A192F?style=for-the-badge&logo=shield&logoColor=00E5FF)](https://github.com/amplydrama)
[![Environment](https://img.shields.io/badge/Lab-Isolated%20VirtualBox%20Host--Only-112240?style=for-the-badge&logo=virtualbox&logoColor=00FF9C)](https://github.com/amplydrama)
[![Framework](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK%20%7C%20Kill%20Chain-1B263B?style=for-the-badge&logo=target&logoColor=FFD166)](https://attack.mitre.org/)
[![Ethics](https://img.shields.io/badge/Scope-Non--Intrusive%20%2F%20No%20Exploitation-0D1B2A?style=for-the-badge&logo=lock&logoColor=38BDF8)](#ethics--rules-of-engagement)

**Author:** KENFACK DJIOLEU Amply Gaulait  
**GitHub:** [@amplydrama](https://github.com/amplydrama) · **Date:** November 2026

</div>

---

## Overview

This repository documents the methodology, isolated lab architecture, reconnaissance and enumeration outputs, vulnerability analysis, MITRE ATT&CK mappings, and real-world case study completed for **ITsimpleara — Red Team Cyber Security (Assignment 1)**.

All active testing was executed strictly inside an isolated **Oracle VirtualBox Host-Only** network (`192.168.56.0/24`) with **zero active exploitation**—focusing exclusively on adversary surface mapping, risk classification, and actionable Blue Team defensive controls.

---

## Table of Contents

- [Lab Architecture & Network Isolation](#lab-architecture--network-isolation)
- [Lab Topology](#lab-topology)
- [Methodology & Toolchain](#methodology--toolchain)
- [Summary of Key Lab Findings](#summary-of-key-lab-findings)
- [Case Study: 2020 SolarWinds Orion Compromise](#case-study-2020-solarwinds-orion-compromise)
- [Repository Structure](#repository-structure)
- [Ethics & Rules of Engagement](#ethics--rules-of-engagement)
- [References](#references)

---

## Lab Architecture & Network Isolation

To prevent accidental scanning or traffic leakage onto external or local home networks, the attacker and target virtual machines communicate solely over an isolated virtual switch:

```text
+-----------------------------------------------------------------------+
|                       HOST WORKSTATION (Windows 11 Pro)              |
|               VirtualBox Host-Only Adapter: 192.168.56.1/24           |
|                  (Netmask: 255.255.255.0 | Mode: Host-Only)           |
+-----------------------------------+-----------------------------------+
                                    |
                 Isolated Virtual Switch (192.168.56.0/24)
              [No Bridged Adapter | No External/LAN Routing]
                                    |
            +-----------------------+-----------------------+
            |                                               |
            v                                               v
+-----------------------+                       +-----------------------+
|   KALI LINUX (Attacker)|                       | METASPLOITABLE 2 (Lab)|
|    Hostname: kali     |                       | Hostname: metasploitable|
|  IP: 192.168.56.10/24 |                       |  IP: 192.168.56.20/24 |
|  Mode: Host-Only      |                       |  Mode: Host-Only      |
+-----------------------+                       +-----------------------+
```
## Lab Topology

| Node | Role | Hostname | IP Address | Subnet Mask | Network Mode |
|------|------|----------|------------|-------------|--------------|
| Host OS | Virtualization Host | `ScriptZ` | `192.168.56.1` | `255.255.255.0` | Host-Only Adapter |
| Attacker VM | Reconnaissance & Enumeration | `kali` | `192.168.56.10` | `255.255.255.0` | Host-Only Adapter |
| Target VM | Intentionally Vulnerable Lab | `metasploitable` | `192.168.56.20` | `255.255.255.0` | Host-Only Adapter |

---

## Methodology & Toolchain

| Phase | Scope / Target | Tools & Commands | Objective |
|-------|----------------|------------------|-----------|
| 1. Passive OSINT | `scanme.nmap.org` (Authorised) | `whois`, `dig`, `nslookup`, `theHarvester` | Gather public DNS, registrar, and Certificate Transparency metadata without touching target infrastructure. |
| 2. Active Recon | `192.168.56.20` (Isolated Lab) | `nmap -sn`, `nmap -sV -sC` | Confirm host availability, identify 23 open TCP ports, and capture service/script banners. |
| 3. Service Enumeration | HTTP (`80/tcp`) & SMB (`139/445`) | `gobuster`, `nikto`, `smbclient`, `enum4linux` | Enumerate web directories, headers, SMB shares, null-session accounts, SIDs, and password policies. |
| 4. Risk & ATT&CK Analysis | Lab Findings & SolarWinds Case | CVSS v3.1, MITRE ATT&CK, Kill Chain | Correlate weaknesses with CVEs, ATT&CK techniques, and Blue Team detection/hardening controls. |

---

## Summary of Key Lab Findings

**Target:** `192.168.56.20` (metasploitable)

| ID | Finding | Port / Service | Severity | MITRE ATT&CK | Primary Remediation |
|----|---------|----------------|----------|--------------|---------------------|
| F-01 | Unauthenticated Root Bindshell Exposed | `1524/tcp` (bindshell) | 🔴 Critical | `T1059` / `T1543` | Terminate listener, remove backdoor entry, and block unauthorized ports via firewall. |
| F-02 | Anonymous SMB Null-Session & Legacy Samba 3.0.20 | `139, 445/tcp` (Samba) | 🟠 High | `T1087` · `T1135` | Upgrade Samba, disable SMBv1 & null sessions, enable SMB signing, and restrict `tmp` share. |
| F-03 | End-of-Life Apache 2.2.8 / PHP 5.2.4 & Exposed `/phpinfo.php` | `80/tcp` (http) | 🟠 High | `T1592` · `T1190` | Remove `/phpinfo.php`, restrict `/phpMyAdmin/`, disable directory indexing & HTTP TRACE, upgrade stack. |
| F-04 | Anonymous FTP Access & Outdated vsftpd 2.3.4 | `21/tcp` (ftp) | 🟠 High | `T1078` · `T1190` | Disable anonymous FTP (`anonymous_enable=NO`) and migrate file transfers to SFTP over SSH. |
| F-05 | Cleartext Remote Admin Protocols (telnetd, r-services) | `23, 512–514/tcp` | 🟡 Medium | `T1040` · `T1110` | Disable Telnet and `r*` daemons; enforce hardened SSH with key-based authentication. |

---

## Ethics & Rules of Engagement

> ⚠️ **Disclaimer**
>
> All practical reconnaissance and enumeration documented in this repository were conducted **strictly inside a private, isolated virtual laboratory** (`192.168.56.0/24`) and against an **explicitly permitted public test domain** (`scanme.nmap.org`) for educational purposes under ITsimpleara coursework guidelines.
>
> **No active exploitation was performed**, and none of the techniques documented here should ever be used against systems without **explicit written authorization**.

---

## References

- [MITRE ATT&CK® Enterprise Matrix](https://attack.mitre.org/) & Campaign `C0024` (SolarWinds Compromise)
- [Lockheed Martin - The Cyber Kill Chain® Framework](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)
- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [OWASP Web Security Testing Guide (WSTG)](https://owasp.org/www-project-web-security-testing-guide/)
