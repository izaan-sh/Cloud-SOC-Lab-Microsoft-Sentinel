# 🛡️ Cloud SOC Lab: Microsoft Sentinel

**Detection and Automated Incident Response in Azure**  
**Author:** Izaan Shumaiz  
**Environment:** Microsoft Azure, Microsoft Sentinel, Log Analytics, Azure Logic Apps  

---

## 📌 Project Overview
This project demonstrates the setup of a Cloud Security Operations Center (SOC) environment using **Microsoft Sentinel**. The primary objective is to simulate brute-force attacks against an exposed Windows Virtual Machine, collect and analyze security logs using **Kusto Query Language (KQL)**, geolocate attack sources, trigger alerts, and implement **automated incident response (Logic Apps)** to automatically block malicious IP addresses.

<p center>
  <img width="1090" height="496" alt="Lab Topology Overview" src="https://github.com/user-attachments/assets/fa55f5df-8f8c-4c18-80a1-900b56effae3" />
</p>

---

## ⚙️ Phase 1: Environment Setup & Exposure

### 1. Resource Group & Virtual Network Setup
Created a dedicated Resource Group named `RG-SOC-Lab` and deployed a Virtual Network inside it.

<img width="1076" height="248" alt="Resource Group and VNet Setup" src="https://github.com/user-attachments/assets/151e6fb7-e8db-48f6-8309-681934cae491" />

### 2. Virtual Machine Deployment (Honeypot)
Deployed a Windows VM designed to look like a standard corporate asset: `CORP-NET-EAST-1`.

<img width="1090" height="390" alt="Virtual Machine Configuration" src="https://github.com/user-attachments/assets/78599ffe-5cf6-4d7e-83e5-2c67cd6c8c55" />  
<img width="940" height="466" alt="VM Overview" src="https://github.com/user-attachments/assets/dce3c0b9-bf08-4407-9323-431bb6ea2ed9" />

### 3. Network Security Group Configuration
To entice external attacks, the default RDP rule was removed and replaced with an **Any-to-Any** inbound traffic rule.

<img width="1919" height="1006" alt="Network Security Group Rules" src="https://github.com/user-attachments/assets/6c1e9c07-ce36-4f1f-9935-4bc762cadfba" />  
<img width="940" height="1211" alt="Adding Open Inbound Rule" src="https://github.com/user-attachments/assets/05475223-da2e-4ad9-9641-48c9e9ecdd9f" />

### 4. VM Configuration & Initial Test
Connected via RDP, disabled all Windows Firewalls to allow incoming traffic, and verified connectivity using ICMP `ping`.

<p align="center">
  <img width="536" height="603" alt="Disable Windows Firewall 1" src="https://github.com/user-attachments/assets/aad7ff46-a90b-45f2-b04e-4477e97158bf" />
  <img width="545" height="455" alt="Disable Windows Firewall 2" src="https://github.com/user-attachments/assets/3baf018e-cb2e-490e-925a-09779d9bf514" />
</p>

<img width="1090" height="571" alt="Ping Test" src="https://github.com/user-attachments/assets/47198685-a210-4018-9aba-6b1b685bedb0" />

---

## 📊 Phase 2: Logging & SIEM Integration

### 1. Simulated Attack Verification
Logged out of RDP and deliberately attempted multiple failed logons with invalid credentials to generate **Event ID 4625** security events in the Windows Event Viewer.

<img width="1090" height="656" alt="Failed Login Attempts" src="https://github.com/user-attachments/assets/b3971eab-017e-4461-bc7c-5d6878a0169f" />

### 2. Log Analytics Workspace & Sentinel Deployment
- Configured a Log Analytics Workspace: `LAW-soc-lab-0000`.
- Enabled Microsoft Sentinel on top of the workspace.

<img width="1090" height="394" alt="Log Analytics Workspace" src="https://github.com/user-attachments/assets/6a35b1a3-4493-4885-8991-e0cb4f06070f" />  
<img width="1090" height="532" alt="Microsoft Sentinel Activation" src="https://github.com/user-attachments/assets/9684d5e7-9316-4140-a1d7-90d12ebfe7bd" />

### 3. Azure Monitor Agent Configuration
Configured the **Azure Monitor Agent (AMA) Security Event Connector** to ingest Windows Event Logs into the Log Analytics Workspace.

<img width="1090" height="496" alt="AMA Security Connector Status" src="https://github.com/user-attachments/assets/fcc896c9-720e-4eac-b49d-88f91e550fbc" />  
<img width="1090" height="524" alt="Adding Security Events Connector" src="https://github.com/user-attachments/assets/f3c669d3-83ec-481e-b765-6d7a284a16e0" />

---

## 🔍 Phase 3: Log Analysis & Geolocation Enrichment

### 1. Verifying Ingested Logs
After a few minutes, Security Events started streaming into Sentinel.

<img width="1090" height="529" alt="Ingested Logs Stream" src="https://github.com/user-attachments/assets/43b619bb-4274-41ad-88bb-4106b425b0a3" />  
<img width="1090" height="627" alt="KQL Query Top 10 Logs" src="https://github.com/user-attachments/assets/15c29408-e3f0-4509-86b9-f1deac1e9822" />

### 2. GeoIP Watchlist Integration
Imported a GeoIP dataset into Sentinel Watchlists ([Data Source](https://raw.githubusercontent.com/joshmadakor1/lognpacific-public/refs/heads/main/misc/geoip-summarized.csv)) to map incoming source IP addresses to physical locations.

<img width="1167" height="773" alt="Importing GeoIP Watchlist" src="https://github.com/user-attachments/assets/d07549c7-f489-4ed0-a025-c6e3435adf34" />  
<img width="1090" height="516" alt="GeoIP Log Mapping" src="https://github.com/user-attachments/assets/571b2eb4-2fed-415b-b509-792a7b064bf7" />

### 3. KQL Queries for Detection
Queried for specific failed logon attempts (`EventID: 4625`) from attacker IPs, projecting custom field names and visualizing attacks globally:

```kql
SecurityEvent
| where EventID == 4625
| where IPAddress == "82.114.228.224"
| sort by TimeGenerated desc
