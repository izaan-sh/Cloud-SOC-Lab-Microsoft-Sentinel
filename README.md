# ☁️ Cloud SOC Lab: Microsoft Sentinel

**Detection, Threat Hunting, and Automated Response in Azure**  
**Platform:** Microsoft Azure | Microsoft Sentinel | KQL | Logic Apps  
**Author:** Izaan Shumaiz | Cybersecurity & AI Graduate  
**Build Period:** September 2026  


![Azure](https://img.shields.io/badge/Microsoft-Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Sentinel](https://img.shields.io/badge/Microsoft-Sentinel-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![KQL](https://img.shields.io/badge/Query-KQL-5C2D91?style=for-the-badge)
![Logic Apps](https://img.shields.io/badge/Automation-Logic%20Apps-0062AD?style=for-the-badge)
![Status](https://img.shields.io/badge/status-completed-brightgreen?style=for-the-badge)

---

## 📌 Project Overview

This project demonstrates the design, deployment, and practical validation of a Cloud Security Operations Center (SOC) environment built on **Microsoft Sentinel**. The lab focuses on deploying an exposed Windows Virtual Machine honeypot, ingesting security telemetry into a **Log Analytics Workspace**, mapping attacker geolocation via Watchlists, executing **KQL threat hunting**, and automating incident containment via **Azure Logic Apps (SOAR)**.

<p align="center">
  <img width="1090" height="496" alt="Project Overview Architecture" src="https://github.com/user-attachments/assets/fa55f5df-8f8c-4c18-80a1-900b56effae3" />
  <br><em>Figure 1.1 — Cloud SOC Architecture & Automated Response Pipeline.</em>
</p>

---

## 📐 Architecture & Environment

| Component | Resource / Name | Purpose & Configuration Details |
| :--- | :--- | :--- |
| **Cloud Provider** | Microsoft Azure | Infrastructure host environment |
| **Resource Group** | `RG-SOC-Lab` | Central container for lab assets |
| **Target Host (Honeypot)**| `CORP-NET-EAST-1` | Windows Server VM exposed to public traffic |
| **Security Control** | Network Security Group (NSG) | Inbound traffic filtering & dynamic blocking |
| **Log Repository** | Log Analytics Workspace (`LAW-soc-lab-0000`) | Centralized log collection & storage |
| **SIEM / XDR** | Microsoft Sentinel | Analytics, incident triage, and threat hunting |
| **Telemetry Agent** | Azure Monitor Agent (AMA) | Ingests Windows Security Events (Event ID 4625) |
| **SOAR Automation** | Azure Logic App | Automated NSG rule generation for offending IPs |

---

## 🚀 Implementation Phases

### Phase 1: Environment Provisioning & Exposure

#### 1. Resource Group & Network Setup
Created a dedicated Resource Group named `RG-SOC-Lab` along with a Virtual Network to host target assets.

<p align="center">
  <img width="1076" height="248" alt="Virtual Network Provisioning" src="https://github.com/user-attachments/assets/151e6fb7-e8db-48f6-8309-681934cae491" />
  <br><em>Figure 2.1 — Resource Group and Virtual Network deployment in Azure.</em>
</p>

#### 2. Virtual Machine Deployment
Deployed a standard Windows VM named `CORP-NET-EAST-1` to mirror authentic corporate infrastructure and entice external threat actors.

<p align="center">
  <img width="1090" height="390" alt="Virtual Machine Setup" src="https://github.com/user-attachments/assets/78599ffe-5cf6-4d7e-83e5-2c67cd6c8c55" />
</p>

<p align="center">
  <img width="940" height="466" alt="VM Overview Panel" src="https://github.com/user-attachments/assets/dce3c0b9-bf08-4407-9323-431bb6ea2ed9" />
  <br><em>Figure 2.2 — Windows target VM configuration details.</em>
</p>

#### 3. Network Security Group Exposure
> ⚠️ **Note:** Environment settings were intentionally configured to allow open inbound access for honeypot data collection.

Removed default RDP restrictions and configured a permissive **Any-to-Any** inbound traffic rule.

<p align="center">
  <img width="1919" height="1006" alt="Network Security Group Rules" src="https://github.com/user-attachments/assets/6c1e9c07-ce36-4f1f-9935-4bc762cadfba" />
</p>

<p align="center">
  <img width="940" height="1211" alt="Configuring Open Inbound Rule" src="https://github.com/user-attachments/assets/05475223-da2e-4ad9-9641-48c9e9ecdd9f" />
  <br><em>Figure 2.3 — NSG rule configured to allow unrestricted inbound traffic.</em>
</p>

#### 4. Host Configuration & Connectivity
Connected via RDP, disabled all profiles under Windows Defender Firewall, and verified network accessibility via ICMP `ping`.

<p align="center">
  <img width="536" height="603" alt="Disabling Host Firewall 1" src="https://github.com/user-attachments/assets/aad7ff46-a90b-45f2-b04e-4477e97158bf" />
  <img width="545" height="455" alt="Disabling Host Firewall 2" src="https://github.com/user-attachments/assets/3baf018e-cb2e-490e-925a-09779d9bf514" />
</p>

<p align="center">
  <img width="1090" height="571" alt="Ping Response Verification" src="https://github.com/user-attachments/assets/47198685-a210-4018-9aba-6b1b685bedb0" />
  <br><em>Figure 2.4 — Disabling Windows Defender Firewall and confirming external connectivity.</em>
</p>

---

### Phase 2: Logging Baseline & SIEM Deployment

#### 1. Telemetry Generation Verification
Generated baseline authentication failures locally to ensure Windows Event Viewer recorded Event ID `4625` under Security logs.

<p align="center">
  <img width="1090" height="656" alt="Event Viewer Event ID 4625" src="https://github.com/user-attachments/assets/b3971eab-017e-4461-bc7c-5d6878a0169f" />
  <br><em>Figure 3.1 — Local Event Viewer confirming failed logon telemetry.</em>
</p>

#### 2. Workspace & Sentinel Configuration
- Provisioned Log Analytics Workspace: `LAW-soc-lab-0000`.
- Enabled Microsoft Sentinel on top of the workspace repository.

<p align="center">
  <img width="1090" height="394" alt="Log Analytics Workspace Setup" src="https://github.com/user-attachments/assets/6a35b1a3-4493-4885-8991-e0cb4f06070f" />
</p>

<p align="center">
  <img width="1090" height="532" alt="Sentinel Workspace Enablement" src="https://github.com/user-attachments/assets/9684d5e7-9316-4140-a1d7-90d12ebfe7bd" />
  <br><em>Figure 3.2 — Microsoft Sentinel initialization.</em>
</p>

#### 3. Data Connector Ingestion
Configured the **Azure Monitor Agent (AMA) Security Events Connector** to stream endpoint logs directly to Sentinel.

<p align="center">
  <img width="1090" height="496" alt="Connector Unbound Status" src="https://github.com/user-attachments/assets/fcc896c9-720e-4eac-b49d-88f91e550fbc" />
</p>

<p align="center">
  <img width="1090" height="524" alt="Connector Bound Successfully" src="https://github.com/user-attachments/assets/f3c669d3-83ec-481e-b765-6d7a284a16e0" />
</p>

<p align="center">
  <img width="1090" height="529" alt="Incoming Log Stream" src="https://github.com/user-attachments/assets/43b619bb-4274-41ad-88bb-4106b425b0a3" />
  <br><em>Figure 3.3 — Security Event data pipeline validation.</em>
</p>

---

### Phase 3: Threat Hunting & Geolocation Enrichment

#### 1. GeoIP Watchlist Integration
Imported a custom GeoIP dataset into Sentinel Watchlists ([Data Source](https://raw.githubusercontent.com/joshmadakor1/lognpacific-public/refs/heads/main/misc/geoip-summarized.csv)) to map incoming authentication requests to exact geographical locations.

<p align="center">
  <img width="1167" height="773" alt="Sentinel Watchlist Import" src="https://github.com/user-attachments/assets/d07549c7-f489-4ed0-a025-c6e3435adf34" />
</p>

<p align="center">
  <img width="1090" height="516" alt="GeoIP Mapping in Logs" src="https://github.com/user-attachments/assets/571b2eb4-2fed-415b-b509-792a7b064bf7" />
  <br><em>Figure 4.1 — Enriched log table displaying country and coordinates.</em>
</p>

#### 2. KQL Analysis Queries
Executed targeted Kusto Query Language (KQL) queries to inspect brute-force patterns, isolate threat actor IPs, and project attack maps:

```kql
// Filtering Event ID 4625 (Failed Logons) for specific attacker IP
SecurityEvent
| where EventID == 4625
| where IPAddress == "82.114.228.224"
| sort by TimeGenerated desc
