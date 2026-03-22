# 🏗️ Enterprise MSP Lab

> A production-grade business network built from scratch inside a virtual datacenter — simulating the infrastructure used by real Managed Service Providers to run and secure client businesses.

**Built by:** Adwaith Jayaraj · Holland College, Charlottetown PEI  
**Status:** Phase 3 in progress (OCI Cloud Migration)  
**Lab hostname:** `HC-lab-firewall.home.arpa`

---

## 📸 Lab Screenshots — Live & Running

### pfSense Firewall Dashboard
![pfSense Dashboard](.screenshots/pfsense-dashboard.png/pfsense-dashboard..jpeg)
> pfSense 2.8.1-RELEASE running on Hyper-V — WAN (10.0.0.78) and LAN (192.168.1.1) fully segmented. All core services live: DHCP, DNS Resolver, NTP, Gateway Monitoring.

### Wazuh SIEM — Vulnerability Detection
![Wazuh Vulnerability Detection](.screenshots/pfsense-dashboard.png/wazuh-vulnerability-detection.jpeg)
> Active vulnerability scan across all endpoints: **19 Critical**, **1,032 High**, **432 Medium** CVEs detected. Top agents: WinServer-DC (1,358 findings) and testuser01 (131 findings). CVE-2025-54100 and CVE-2025-59505/06/07 flagged for patch management.

---

## 🗺️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        WAN (10.0.0.78)                          │
│                     10Gbase-T · Full-Duplex                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                  ┌──────────▼──────────┐
                  │   pfSense Firewall   │
                  │   v2.8.1 · FreeBSD  │
                  │   Hyper-V VM        │
                  │                     │
                  │  WAN ─── 10.0.0.78  │
                  │  LAN ─ 192.168.1.1  │
                  └──────────┬──────────┘
                             │ LAN (192.168.1.0/24)
                             │ 10Gbase-T · Full-Duplex
          ┌──────────────────┼──────────────────┐
          │                  │                  │
┌─────────▼────────┐ ┌───────▼───────┐ ┌───────▼────────┐
│  Windows Server  │ │   Windows 10  │ │  Ubuntu Linux  │
│  2022 Datacenter │ │   Education   │ │   Endpoint     │
│                  │ │               │ │                │
│  • Domain Ctrl   │ │  • testuser01 │ │  • Wazuh Mgr   │
│  • AD DS         │ │  • Domain     │ │  • SIEM Engine │
│  • WinServer-DC  │ │    joined     │ │  • Log Collect │
│  • Wazuh Agent   │ │  • Wazuh Agent│ │                │
│  • 1,358 CVEs    │ │  • 131 CVEs   │ │                │
│    tracked       │ │    tracked    │ │                │
└──────────────────┘ └───────────────┘ └────────────────┘
          │                  │
          └──────────────────┘
              Wazuh telemetry → Linux Manager
```

---

## ✅ Phase 1 — Core Infrastructure

**Completed January 2026**

### What was built

| Component | Details | Status |
|---|---|---|
| **Hypervisor** | Hyper-V on Windows Server 2022 Datacenter | ✅ Live |
| **Firewall** | pfSense 2.8.1-RELEASE (FreeBSD 15.0) | ✅ Live |
| **WAN Interface** | 10.0.0.78 · 10Gbase-T · Full-duplex | ✅ Live |
| **LAN Interface** | 192.168.1.1 · 10Gbase-T · Full-duplex | ✅ Live |
| **DHCP Server** | ISC DHCP — serving 192.168.1.0/24 | ✅ Running |
| **DNS Resolver** | Unbound DNS — local resolution + forwarding | ✅ Running |
| **NTP** | ntpd clock sync across all endpoints | ✅ Running |
| **Gateway Monitor** | dpinger — WAN_DHCP online, RTT 0.7ms / 2.3ms sd | ✅ Running |
| **Domain Controller** | Active Directory DS — Primary DC | ✅ Live |
| **Endpoints** | Windows 10 Education + Ubuntu Linux, domain-joined | ✅ Live |
| **Security Policy** | GPOs — baseline hardening across all domain endpoints | ✅ Applied |

### pfSense Services Running

```
dhcpd   ● ISC DHCP Server          — serving LAN clients
dpinger ● Gateway Monitoring        — WAN: Online (0.7ms RTT)
iperf   ● Network Performance Test  — bandwidth benchmarking
ntpd    ● NTP Clock Sync            — time authority for domain
syslogd ● System Logger Daemon      — centralised log collection
unbound ● DNS Resolver              — local + upstream resolution
```

### Network Segmentation

```
WAN Zone  (10.0.0.0/24)     — External-facing, strict ingress rules
LAN Zone  (192.168.1.0/24)  — Internal private network, stateful firewall
```

---

## ✅ Phase 2 — Security Monitoring (Wazuh SIEM)

**Completed February 2026**

### What was built

| Component | Details | Status |
|---|---|---|
| **Wazuh Manager** | Linux-based SIEM manager | ✅ Live |
| **Agent: WinServer-DC** | Windows Server 2022 endpoint monitoring | ✅ Active |
| **Agent: testuser01** | Windows 10 Education endpoint monitoring | ✅ Active |
| **CVE Engine** | Vulnerability detection across all assets | ✅ Active |
| **FIM** | File Integrity Monitoring — real-time change detection | ✅ Active |
| **Log Ingestion** | alerts.log + alerts.json custom parsing | ✅ Active |
| **Adversary Emulation** | Nmap scan + directory traversal simulation | ✅ Tested |

### Live Vulnerability Summary

```
┌─────────────────────────────────────────────────────────┐
│           WAZUH VULNERABILITY DETECTION — LIVE          │
│                  Cluster: adwaith                        │
├──────────────┬───────────┬───────────┬──────────────────┤
│   CRITICAL   │   HIGH    │  MEDIUM   │      LOW         │
│      19      │   1,032   │    432    │       6          │
├──────────────┴───────────┴───────────┴──────────────────┤
│ Top CVEs Detected:                                       │
│  • CVE-2025-54100  (×2)  — Windows Server 2022          │
│  • CVE-2025-59505  (×2)  — Windows Server 2022          │
│  • CVE-2025-59506  (×2)  — Windows Server 2022          │
│  • CVE-2025-59507  (×2)  — Windows Server 2022          │
│  • CVE-2025-552    (×2)  — Windows Server 2022          │
├─────────────────────────────────────────────────────────┤
│ Top Agents:                                              │
│  • WinServer-DC   → 1,358 vulnerabilities tracked       │
│  • testuser01     → 131 vulnerabilities tracked         │
└─────────────────────────────────────────────────────────┘
```

### Adversary Emulation Results

```bash
# Simulated attacks — all successfully flagged by Wazuh:

[ALERT] Rule 100002  — Nmap network scan detected on 192.168.1.x
[ALERT] Rule 550     — File integrity change: /etc/passwd modified
[ALERT] Rule 510     — Unauthorized directory traversal attempt
[ALERT] CVE-2025-54100 — Critical vulnerability on WinServer-DC
```

### Log Intelligence Setup

Beyond default Wazuh configs, raw telemetry was ingested from:
- `alerts.log` — parsed for silent network reconnaissance
- `alerts.json` — decoded for structured event correlation
- **Result:** Nmap scans and directory traversal flagged that default rules miss

---

## 🔄 Phase 3 — OCI Cloud Migration + Hybrid Identity

**In Progress — 2026**

### Target Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                   Oracle Cloud Infrastructure                     │
│                                                                  │
│   ┌──────────┐      ┌──────────────┐      ┌──────────┐          │
│   │  VCN +   │      │ Site-to-Site │      │   NLB    │          │
│   │ Subnets  │◄────►│     VPN      │◄────►│ Load Bal.│          │
│   └──────────┘      └──────┬───────┘      └──────────┘          │
│                            │ IPSec Tunnel                        │
└────────────────────────────┼─────────────────────────────────────┘
                             │
┌────────────────────────────▼─────────────────────────────────────┐
│                    On-Premises Lab                                │
│                                                                  │
│   [pfSense] ◄──────► [Windows Server 2022 + AD DS]              │
│                                    │                             │
│                            [Entra Connect]                        │
│                                    │                             │
│                            [Azure Entra ID]                       │
│                         ← Hybrid Identity Sync →                 │
└──────────────────────────────────────────────────────────────────┘
```

### Phase 3 Goals

- [ ] Deploy OCI VCN with public/private subnets
- [ ] Establish Site-to-Site IPSec VPN (pfSense ↔ OCI)
- [ ] Configure Network Load Balancer (NLB)
- [ ] Deploy Microsoft Entra Connect
- [ ] Sync on-prem AD users → Azure Entra ID
- [ ] Unified cloud + on-prem Wazuh monitoring
- [ ] Document full hybrid identity model

---

## 🛠️ Full Tech Stack

```
NETWORKING     pfSense 2.8.1 · FreeBSD 15.0 · TCP/IP · DNS · DHCP · VLANs · VPN
HYPERVISOR     Microsoft Hyper-V · Windows Server 2022 Datacenter
IDENTITY       Active Directory DS · Group Policy Objects · Microsoft Entra Connect
ENDPOINTS      Windows Server 2022 · Windows 10 Education · Ubuntu Linux
SECURITY       Wazuh SIEM · FIM · CVE Engine · Log Ingestion · Adversary Emulation
CLOUD          Oracle Cloud Infrastructure (OCI) · Azure Entra ID · Hybrid Identity
SCRIPTING      PowerShell (AD automation) · Python (tooling)
```

---

## 📁 Repository Structure

```
enterprise-msp-lab/
│
├── README.md                    ← You are here
├── screenshots/
│   ├── pfsense-dashboard.png    ← Live pfSense dashboard
│   └── wazuh-vulnerability-detection.png  ← Live Wazuh SIEM
│
├── phase1-infrastructure/
│   ├── pfsense-config-notes.md  ← Firewall rules & segmentation
│   ├── active-directory-setup.md
│   └── gpo-hardening-guide.md
│
├── phase2-siem/
│   ├── wazuh-deployment.md      ← Manager + agent setup
│   ├── log-ingestion-config.md  ← alerts.log / alerts.json
│   ├── fim-setup.md             ← File Integrity Monitoring
│   └── adversary-emulation.md   ← Nmap + traversal simulation
│
└── phase3-cloud/                ← In progress
    ├── oci-vcn-setup.md
    ├── site-to-site-vpn.md
    └── entra-connect-hybrid.md
```

---

## 📫 Contact

**Adwaith Jayaraj**  
📧 adwaith.jayaraj03info@gmail.com  
📞 902-978-3375  
🔗 [linkedin.com/in/adwaith-jayaraj](https://linkedin.com/in/adwaith-jayaraj)  
🐙 [github.com/tensorTamer](https://github.com/tensorTamer)

> *Actively seeking summer IT opportunities in Charlottetown, PEI — MSP, helpdesk, sysadmin, cloud, or security.*
