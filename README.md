# ☁️ Cloud SOC Lab: Microsoft Sentinel

**Detection, Threat Hunting, and Automated Response in Azure**
**Platform:** Microsoft Azure | Microsoft Sentinel | KQL | Logic Apps
**Author:** Izaan Shumaiz | Cybersecurity & AI Graduate
**Build Period:** September 2026

![Azure](https://img.shields.io/badge/Microsoft-Azure-0078D4?style=for-the-badge\&logo=microsoftazure\&logoColor=white)
![Sentinel](https://img.shields.io/badge/Microsoft-Sentinel-0078D4?style=for-the-badge\&logo=microsoft\&logoColor=white)
![KQL](https://img.shields.io/badge/Query-KQL-5C2D91?style=for-the-badge)
![Logic Apps](https://img.shields.io/badge/Automation-Logic%20Apps-0062AD?style=for-the-badge)
![Status](https://img.shields.io/badge/status-completed-brightgreen?style=for-the-badge)

---

## 📌 Project Overview

This project demonstrates the design, deployment, and practical validation of a Cloud Security Operations Center (SOC) environment built on **Microsoft Sentinel**.

The lab focuses on deploying an exposed Windows Virtual Machine honeypot, ingesting security telemetry into a **Log Analytics Workspace**, mapping attacker geolocation through Sentinel Watchlists, performing **KQL-based threat hunting**, creating detection rules, investigating incidents, and automating containment through **Azure Logic Apps (SOAR)**.

<p align="center">
  <img width="1090" height="496" alt="Project Overview Architecture" src="https://github.com/user-attachments/assets/fa55f5df-8f8c-4c18-80a1-900b56effae3" />
  <br><em>Figure 1.1 — Cloud SOC Architecture & Automated Response Pipeline.</em>
</p>

---

## 📐 Architecture & Environment

| Component                  | Resource / Name                              | Purpose & Configuration Details                 |
| :------------------------- | :------------------------------------------- | :---------------------------------------------- |
| **Cloud Provider**         | Microsoft Azure                              | Infrastructure host environment                 |
| **Resource Group**         | `RG-SOC-Lab`                                 | Central container for lab assets                |
| **Target Host (Honeypot)** | `CORP-NET-EAST-1`                            | Windows Server VM exposed to public traffic     |
| **Security Control**       | Network Security Group (NSG)                 | Inbound traffic filtering & dynamic blocking    |
| **Log Repository**         | Log Analytics Workspace (`LAW-soc-lab-0000`) | Centralized log collection & storage            |
| **SIEM / XDR**             | Microsoft Sentinel                           | Analytics, incident triage, and threat hunting  |
| **Telemetry Agent**        | Azure Monitor Agent (AMA)                    | Ingests Windows Security Events                 |
| **SOAR Automation**        | Azure Logic App                              | Automated NSG rule generation for offending IPs |

---

# 🚀 Implementation Phases

## Phase 1: Environment Provisioning & Exposure

### 1. Resource Group & Network Setup

Created a dedicated Resource Group named `RG-SOC-Lab` along with a Virtual Network to host the target assets.

<p align="center">
  <img width="1076" height="248" alt="Virtual Network Provisioning" src="https://github.com/user-attachments/assets/151e6fb7-e8db-48f6-8309-681934cae491" />
  <br><em>Figure 2.1 — Resource Group and Virtual Network deployment in Azure.</em>
</p>

### 2. Virtual Machine Deployment

Deployed a standard Windows VM named `CORP-NET-EAST-1` to mirror a corporate endpoint and expose the environment to authentic external traffic.

<p align="center">
  <img width="1090" height="390" alt="Virtual Machine Setup" src="https://github.com/user-attachments/assets/78599ffe-5cf6-4d7e-83e5-2c67cd6c8c55" />
</p>

<p align="center">
  <img width="1919" height="1005" alt="Screenshot 2026-09-22 155043" src="https://github.com/user-attachments/assets/14216185-2fed-40c5-b9fb-4ca3bed2d914" />
  <br><em>Figure 2.2 — Windows target VM configuration details.</em>
</p>

### 3. Network Security Group Exposure

> ⚠️ **Note:** Environment settings were intentionally configured to allow open inbound access for honeypot data collection.

Removed the default RDP restrictions and configured a permissive **Any-to-Any** inbound traffic rule.

<p align="center">
  <img width="1919" height="1006" alt="Network Security Group Rules" src="https://github.com/user-attachments/assets/6c1e9c07-ce36-4f1f-9935-4bc762cadfba" />
</p>

<p align="center">
  <img width="940" height="1211" alt="Configuring Open Inbound Rule" src="https://github.com/user-attachments/assets/05475223-da2e-4ad9-9641-48c9e9ecdd9f" />
  <br><em>Figure 2.3 — NSG rule configured to allow unrestricted inbound traffic.</em>
</p>

### 4. Host Configuration & Connectivity

Connected to the VM via RDP, disabled the Windows Defender Firewall profiles for the controlled lab environment, and verified network accessibility using ICMP `ping`.

<p align="center">
  <img width="536" height="603" alt="Disabling Host Firewall 1" src="https://github.com/user-attachments/assets/aad7ff46-a90b-45f2-b04e-4477e97158bf" />
  <img width="545" height="455" alt="Disabling Host Firewall 2" src="https://github.com/user-attachments/assets/3baf018e-cb2e-490e-925a-09779d9bf514" />
</p>

<p align="center">
  <img width="1090" height="571" alt="Ping Response Verification" src="https://github.com/user-attachments/assets/47198685-a210-4018-9aba-6b1b685bedb0" />
  <br><em>Figure 2.4 — Disabling Windows Defender Firewall and confirming external connectivity.</em>
</p>

---

## Phase 2: Logging Baseline & SIEM Deployment

### 1. Telemetry Generation Verification

Generated baseline authentication failures locally to ensure Windows Event Viewer recorded Event ID `4625` under Security logs.

<p align="center">
  <img width="1090" height="656" alt="Event Viewer Event ID 4625" src="https://github.com/user-attachments/assets/b3971eab-017e-4461-bc7c-5d6878a0169f" />
  <br><em>Figure 3.1 — Local Event Viewer confirming failed logon telemetry.</em>
</p>

### 2. Workspace & Sentinel Configuration

* Provisioned Log Analytics Workspace: `LAW-soc-lab-0000`
* Enabled Microsoft Sentinel on the workspace

<p align="center">
  <img width="1381" height="499" alt="Screenshot 2026-09-22 163741" src="https://github.com/user-attachments/assets/e14169c6-de19-40c2-ab9f-e59dbea0b3df" />
</p>

<p align="center">
  <img width="1090" height="532" alt="Sentinel Workspace Enablement" src="https://github.com/user-attachments/assets/9684d5e7-9316-4140-a1d7-90d12ebfe7bd" />
  <br><em>Figure 3.2 — Microsoft Sentinel initialization.</em>
</p>

### 3. Data Connector Ingestion

Configured the **Azure Monitor Agent (AMA) Security Events Connector** to stream endpoint security logs into Sentinel.

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

## Phase 3: Threat Hunting & Geolocation Enrichment

### 1. GeoIP Watchlist Integration

Imported a custom GeoIP dataset into Sentinel Watchlists to enrich authentication events with geographical information.

**Data Source:**
https://raw.githubusercontent.com/joshmadakor1/lognpacific-public/refs/heads/main/misc/geoip-summarized.csv

<p align="center">
  <img width="1167" height="773" alt="Sentinel Watchlist Import" src="https://github.com/user-attachments/assets/d07549c7-f489-4ed0-a025-c6e3435adf34" />
</p>

<p align="center">
  <img width="1090" height="516" alt="GeoIP Mapping in Logs" src="https://github.com/user-attachments/assets/571b2eb4-2fed-415b-b509-792a7b064bf7" />
  <br><em>Figure 4.1 — Enriched log table displaying country and coordinates.</em>
</p>

### 2. KQL Analysis Queries

Executed targeted **Kusto Query Language (KQL)** queries to inspect failed authentication patterns, isolate source IP addresses, and support threat hunting.

#### Failed Logon Investigation by Source IP

```kql
// Filtering Event ID 4625 (Failed Logons) for a specific source IP
SecurityEvent
| where EventID == 4625
| where IPAddress == "82.114.228.224"
| sort by TimeGenerated desc
```

The query was used to isolate failed authentication activity associated with a specific source IP and review the events chronologically.

#### Failed Logons by Account

```kql
SecurityEvent
| where EventID == 4625
| summarize FailedLogons = count() by Account
| where FailedLogons > 10
| sort by FailedLogons desc
```

This query helped identify accounts receiving a high volume of failed authentication attempts.

#### Failed Logons by Source IP

```kql
SecurityEvent
| where EventID == 4625
| summarize FailedLogons = count() by IPAddress
| sort by FailedLogons desc
```

This provided a broader view of source IP addresses generating failed authentication activity.

---

## 🚨 Phase 4: Alert Rules & Incident Management

### 1. Analytics Rule Setup

Configured a Microsoft Sentinel Analytics Rule to detect accounts experiencing more than **10 failed logon attempts**.

The rule was designed to convert repeated failed authentication activity into a Sentinel incident for investigation.

<p align="center">
  <img width="1835" height="818" alt="Screenshot 2026-09-23 152122" src="https://github.com/user-attachments/assets/828cfd70-3d10-4bf2-a7b6-b0cd7ec50df2" />
  <img width="1838" height="651" alt="Screenshot 2026-09-23 152148" src="https://github.com/user-attachments/assets/13bd64fc-b7fa-4822-a203-78fac4dd4775" />
  <br><em>Figure 5.1 — Sentinel rule setup wizard.</em>
</p>

### 2. Incident Investigation

Once the threshold conditions were met, Microsoft Sentinel generated incidents containing the associated authentication activity.

The investigation process included reviewing:

* Source IP addresses
* Targeted accounts
* Number of failed authentication attempts
* Event timestamps
* Authentication activity surrounding the incident
* Whether successful authentication occurred after repeated failures

<p align="center">
  <img width="1855" height="916" alt="Screenshot 2026-09-23 152432" src="https://github.com/user-attachments/assets/c5afed7d-1dc5-4c3b-9ed0-14cb2ff9bd14" />
  <img width="1904" height="943" alt="Screenshot 2026-09-23 171931" src="https://github.com/user-attachments/assets/6dbe9ea2-f70f-4e43-8fa7-763385332aef" />
  <br><em>Figure 5.2 — Sentinel incident investigation and authentication activity.</em>
</p>

---

## 🤖 Phase 5: Automated Response — SOAR with Logic Apps

### 1. Logic App Design

Designed an automated response workflow using **Azure Logic Apps** to create a **Deny rule** on the Network Security Group when suspicious authentication activity crossed the configured threshold.

<p align="center">
  <img width="559" height="706" alt="Screenshot 2026-09-23 203808" src="https://github.com/user-attachments/assets/3bcd1c62-a798-450e-b30f-49e38b4aa353" />
  <br><em>Figure 6.1 — Logic app designer</em>
</p>

The response pipeline followed this flow:

```text
Sentinel Detection
       ↓
Sentinel Incident
       ↓
Azure Logic App
       ↓
Extract Source IP
       ↓
Identify Target NSG
       ↓
Create NSG Deny Rule
       ↓
Block Source IP
```

### 2. Testing Automated Enforcement

During controlled testing, source IP `14.241.68.109` triggered the configured threshold and was automatically added to the Network Security Group deny list.

<p align="center">
  <img width="1894" height="943" alt="Screenshot 2026-09-23 235805" src="https://github.com/user-attachments/assets/53268df3-e611-4712-b071-49e1be20a4db" />
  <br><em>Figure 6.1 — Automated NSG deny rule generated through the Logic App.</em>
</p>

### ⚠️ Troubleshooting & Key Finding

During initial testing, traffic from blocked IP addresses was still being observed.

Investigation revealed that the initial **Allow Any** inbound NSG rule had a priority of `100`, causing it to take precedence over the newly created deny rule.

### Resolution

Adjusted the NSG rule priorities so that the automatically generated deny rule was evaluated before the permissive inbound rule.

This demonstrated an important practical aspect of cloud security controls: **creating a deny rule does not guarantee enforcement if a higher-priority allow rule takes precedence.**

### 3. Verification of Block

After correcting the NSG rule priority, testing confirmed that subsequent connection attempts from the blocked source were no longer observed.

<p align="center">
  <img width="1544" height="926" alt="Screenshot 2026-09-24 001054" src="https://github.com/user-attachments/assets/d700a01f-b710-41c4-9bc6-d31fc26e7602" />
  <br><em>Figure 6.2 — Validation of automated source IP containment.</em>
</p>

<p align="center">
  <img width="1650" height="226" alt="Screenshot 2026-09-24 001158" src="https://github.com/user-attachments/assets/3531da85-9a2b-479f-bdc3-2a7e1e2fa313" />
  <br><em>Figure 6.3 — Autoblock rule now has a higher priority.</em>
</p>

Now we can confirm that there were no more failed login attempts from the source IP after the autoblock, the autoblock was made at UTC Time 08:05 PM, and the latest logged event was at 08:04 PM - this shows that there were no logs made after the autoblock
<p align="center">
  <img width="1186" height="732" alt="Screenshot 2026-09-24 002049" src="https://github.com/user-attachments/assets/c9d100dd-81e7-4817-9132-ddd552c9aeb8" />
  <br><em>Figure 6.4 — KQL Query Output - Latest failed login attempts logs from IPAddress: '14:241:68:109'.</em>
</p>

---

## 📈 Phase 6: Sentinel Workbook & Dashboards

Finalized the deployment by configuring Microsoft Sentinel Workbooks to visualize security activity and investigation data.

The dashboard provided visibility into:

* Failed authentication activity
* Source IP addresses
* Targeted accounts
* Attack trends
* Geographic distribution of authentication attempts
* Incident activity

<p align="center">
  <img width="1090" height="600" alt="Sentinel SOC Dashboard" src="https://github.com/user-attachments/assets/REPLACE_WITH_DASHBOARD_SCREENSHOT" />
  <br><em>Figure 7.1 — Microsoft Sentinel SOC dashboard and security monitoring visualizations.</em>
</p>

---

# 🔎 Investigation Workflow

The overall SOC workflow implemented in the lab was:

```text
External Authentication Attempts
            ↓
Windows Security Event 4625
            ↓
Azure Monitor Agent
            ↓
Log Analytics Workspace
            ↓
Microsoft Sentinel
            ↓
KQL Threat Hunting
            ↓
Analytics Rule
            ↓
Sentinel Incident
            ↓
Logic App Automation
            ↓
NSG Deny Rule
            ↓
Source IP Containment
            ↓
Validation
```

This workflow demonstrates the transition from raw endpoint telemetry to detection, investigation, automated response, and validation.

---

# 🧪 Attack & Detection Validation

| Activity                      | Telemetry                           | Detection / Response        | Result                                         |
| :---------------------------- | :---------------------------------- | :-------------------------- | :--------------------------------------------- |
| Failed Windows authentication | Event ID `4625`                     | KQL threat hunting          | Source IP and targeted accounts identified     |
| Repeated failed logons        | Multiple `4625` events              | Sentinel Analytics Rule     | Incident generated                             |
| Suspicious source IP          | Source IP entity                    | Logic App automation        | NSG deny rule created                          |
| Initial block failure         | NSG rule priority conflict          | Rule priority investigation | Root cause identified                          |
| Corrected containment         | NSG deny rule with correct priority | Automated enforcement       | Further connection attempts no longer observed |
| GeoIP enrichment              | Source IP                           | Sentinel Watchlist          | Geographic context added                       |
| Security monitoring           | Sentinel telemetry                  | Workbook visualization      | Attack activity visualized                     |

---

# 🛠️ Technologies & Skills Demonstrated

### Cloud & Infrastructure

* Microsoft Azure
* Azure Virtual Machines
* Virtual Networks
* Network Security Groups
* Log Analytics Workspace

### SIEM & SOC

* Microsoft Sentinel
* Security Event Monitoring
* Incident Investigation
* Threat Hunting
* Security Telemetry Analysis

### Detection Engineering

* KQL
* Windows Event ID `4625`
* Analytics Rules
* Authentication Failure Detection
* Source IP Analysis
* GeoIP Enrichment

### Automation & Response

* Azure Logic Apps
* SOAR Workflows
* Dynamic NSG Rule Creation
* Automated IP Containment
* NSG Priority Management

### Operating Systems

* Windows Server
* Windows Security Event Logs

---

# 📊 Project Results

The completed lab demonstrated an end-to-end cloud SOC workflow:

**1. Telemetry Collection**
Windows authentication events were successfully collected through Azure Monitor Agent and ingested into Log Analytics.

**2. Threat Hunting**
KQL queries were used to investigate failed authentication activity and identify suspicious source IP addresses and targeted accounts.

**3. Detection**
A Sentinel Analytics Rule converted repeated failed authentication activity into security incidents.

**4. Investigation**
Sentinel incidents provided the context required to investigate source IPs, targeted accounts, timestamps, and authentication patterns.

**5. Automated Response**
Azure Logic Apps automatically created NSG deny rules for source IPs associated with suspicious activity.

**6. Troubleshooting**
An NSG rule-priority conflict was identified during testing and corrected to ensure the automated deny rule was evaluated correctly.

**7. Validation**
The final configuration successfully demonstrated automated source IP containment and subsequent validation of the response.

---

# 📁 Suggested Repository Structure

```text
microsoft-sentinel-soc-lab/
│
├── README.md
│
├── KQL/
│   ├── failed-logons.kql
│   ├── failed-logons-by-ip.kql
│   ├── targeted-accounts.kql
│   ├── successful-logons.kql
│   └── multiple-failed-logons-detection.kql
│
├── Detection-Rules/
│   └── multiple-failed-logons.md
│
├── Automation/
│   └── SOC-AutoBlock-Malicious-IP.md
│
└── screenshots/
    ├── 01-azure-resource-group.png
    ├── 02-virtual-machine.png
    ├── 03-network-security-group.png
    ├── 04-rdp-access.png
    ├── 05-log-analytics.png
    ├── 06-microsoft-sentinel.png
    ├── 07-ama-connector.png
    ├── 08-security-events.png
    ├── 09-geoip.png
    ├── 10-kql-investigation.png
    ├── 11-analytics-rule.png
    ├── 12-sentinel-incident.png
    ├── 13-incident-investigation.png
    ├── 14-logic-app.png
    ├── 15-nsg-autoblock.png
    ├── 16-nsg-priority-fix.png
    └── 17-block-confirmed.png
```

---

# 🎯 Future Improvements

Potential extensions to the lab include:

* Add additional Windows security event detections
* Develop more advanced KQL detection rules
* Integrate MITRE ATT&CK mappings
* Expand automated response workflows
* Add additional endpoint telemetry
* Integrate additional threat intelligence sources
* Introduce more structured incident investigation playbooks
* Simulate additional attack techniques

---

# 🏁 Conclusion

This project provided hands-on experience building and operating a cloud-based SOC using **Microsoft Sentinel**.

The lab covered the complete security monitoring lifecycle:

> **Collect → Hunt → Detect → Investigate → Respond → Validate**

Rather than only configuring a SIEM, the project focused on demonstrating how security telemetry can be transformed into actionable detections and automated containment through Azure-native security services.

The final environment combined **Microsoft Sentinel, KQL, Azure Monitor Agent, Log Analytics, Watchlists, Logic Apps, and Network Security Groups** into a practical cloud SOC workflow.
