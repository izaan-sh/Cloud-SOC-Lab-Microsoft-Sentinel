# ☁️ Cloud SOC Lab: Microsoft Sentinel: Detection and Automated Response with Microsoft Sentinel


**Platform:** Microsoft Azure | Sentinel | KQL | Logic Apps  
**Author:** Izaan Shumaiz | Cybersecurity & AI Graduate  
**Build Period:** September 15–17, 2026 | **Report Date:** September 20, 2026  

---


## 📖 Overview

This lab builds a small cloud SOC. I deployed an intentionally exposed Windows VM (a honeypot), collected its security logs in Microsoft Sentinel, and mapped real attacker activity. I then wrote a detection rule for repeated failed logons and used a Logic App to block attacking IPs automatically.

<img width="1090" height="496" alt="Project overview" src="https://github.com/user-attachments/assets/fa55f5df-8f8c-4c18-80a1-900b56effae3" />

---

## 🎯 What This Project Shows

- Deploying Azure resources: resource group, virtual network, VM and NSG
- Forwarding Windows Security Events to a Log Analytics Workspace
- Enriching logs with GeoIP data through a Sentinel Watchlist
- Hunting with KQL and mapping attacks by location
- Creating analytics rules that raise incidents
- Automating response with a Logic App that blocks IPs in the NSG
- Finding and fixing an NSG rule priority issue
- Building a Sentinel dashboard

## 🧰 Tech Stack

| Component | Purpose |
|---|---|
| Azure Virtual Machine | Honeypot (`CORP-NET-EAST-1`) |
| Network Security Group | Inbound traffic control |
| Log Analytics Workspace | Log storage (`LAW-soc-lab-0000`) |
| Microsoft Sentinel | SIEM: detection, incidents, workbooks |
| Azure Monitor Agent | Sends Security Events from the VM |
| KQL | Log queries |
| Logic App | Automated IP blocking |

## 🗺️ Workflow

1. Build the environment
2. Generate failed logons and check the logs
3. Connect the logs to Sentinel
4. Enrich with GeoIP
5. Hunt with KQL
6. Create the detection rule
7. Automate the response
8. Fix the priority issue
9. Build the dashboard

---

## 🏗️ Step 1: Build the Environment

I logged into Azure and created a resource group named **`RG-SOC-Lab`**, then a virtual network inside it.

<img width="1076" height="248" alt="Virtual network" src="https://github.com/user-attachments/assets/151e6fb7-e8db-48f6-8309-681934cae491" />

Next I created a VM with a name that looks ordinary to attackers: **`CORP-NET-EAST-1`**.

<img width="1090" height="390" alt="VM creation" src="https://github.com/user-attachments/assets/78599ffe-5cf6-4d7e-83e5-2c67cd6c8c55" />

<img width="940" height="466" alt="VM overview" src="https://github.com/user-attachments/assets/dce3c0b9-bf08-4407-9323-431bb6ea2ed9" />

## 🔓 Step 2: Expose the VM (Lab Only)

> ⚠️ **Warning:** These settings are deliberately insecure so the honeypot attracts attacks. Never do this on a production system.

**Network Security Group**

<img width="1919" height="1006" alt="NSG" src="https://github.com/user-attachments/assets/6c1e9c07-ce36-4f1f-9935-4bc762cadfba" />

I removed the default RDP inbound rule and added a new inbound rule that allows **any** traffic.

<img width="940" height="1211" alt="Allow-any inbound rule" src="https://github.com/user-attachments/assets/05475223-da2e-4ad9-9641-48c9e9ecdd9f" />

**Connect over RDP**

<img width="1919" height="1003" alt="RDP login" src="https://github.com/user-attachments/assets/9939b531-7df3-4d89-a584-fbf003a99af8" />

**Turn off the Windows firewalls**

<img width="536" height="603" alt="Firewall settings" src="https://github.com/user-attachments/assets/aad7ff46-a90b-45f2-b04e-4477e97158bf" />
<img width="545" height="455" alt="Firewall off" src="https://github.com/user-attachments/assets/3baf018e-cb2e-490e-925a-09779d9bf514" />

**Ping test:** the reply confirms the VM is reachable.

<img width="1090" height="571" alt="Ping test" src="https://github.com/user-attachments/assets/47198685-a210-4018-9aba-6b1b685bedb0" />

## 🔑 Step 3: Generate Failed Logons

I logged out of RDP and tried to log back in with wrong credentials about 4 times. This checks that Event Viewer records failed logins (Event ID 4625) in the Security logs.

<img width="1090" height="656" alt="Event Viewer failed logons" src="https://github.com/user-attachments/assets/b3971eab-017e-4461-bc7c-5d6878a0169f" />

---

## 📡 Step 4: Connect Logs to Sentinel

**1. Create a Log Analytics Workspace** named `LAW-soc-lab-0000`. This is the log repository the VM logs are forwarded to.

<img width="1090" height="394" alt="Log Analytics Workspace" src="https://github.com/user-attachments/assets/6a35b1a3-4493-4885-8991-e0cb4f06070f" />

**2. Add Microsoft Sentinel to the workspace.**

<img width="1090" height="532" alt="Sentinel added" src="https://github.com/user-attachments/assets/9684d5e7-9316-4140-a1d7-90d12ebfe7bd" />

**3. Configure the Azure Monitor Agent Security Events connector.** The red circle marks the connection that was not working yet.

<img width="1090" height="496" alt="Connector not connected" src="https://github.com/user-attachments/assets/fcc896c9-720e-4eac-b49d-88f91e550fbc" />

**4. Add the Security Events connector.**

<img width="1090" height="524" alt="Connector added" src="https://github.com/user-attachments/assets/f3c669d3-83ec-481e-b765-6d7a284a16e0" />

**5. Wait a few minutes.** Logs then start to appear.

<img width="1090" height="529" alt="Logs arriving" src="https://github.com/user-attachments/assets/43b619bb-4274-41ad-88bb-4106b425b0a3" />

**KQL: show 10 security events**

<img width="1090" height="627" alt="KQL 10 events" src="https://github.com/user-attachments/assets/15c29408-e3f0-4509-86b9-f1deac1e9822" />

---

## 🌍 Step 5: Enrich with GeoIP

I added a [GeoIP CSV](https://raw.githubusercontent.com/joshmadakor1/lognpacific-public/refs/heads/main/misc/geoip-summarized.csv) to a Sentinel Watchlist.

<img width="1167" height="773" alt="GeoIP watchlist" src="https://github.com/user-attachments/assets/d07549c7-f489-4ed0-a025-c6e3435adf34" />

After it was added, the logs show the attacker's location.

<img width="1090" height="516" alt="Logs with location" src="https://github.com/user-attachments/assets/571b2eb4-2fed-415b-b509-792a7b064bf7" />

## 🔎 Step 6: Hunt with KQL

**Failed logons (Event ID 4625) from `82.114.228.224`, ordered by time**

<img width="1090" height="546" alt="KQL failed logons by IP" src="https://github.com/user-attachments/assets/2c316af0-0664-49d1-b3c9-f12fca687bdb" />

**Project only the needed columns and rename them**

<img width="1239" height="691" alt="KQL project and rename" src="https://github.com/user-attachments/assets/d5ebfb6d-80c5-447d-896a-a6b9dcb7fb26" />

**Attack map**

<img width="1090" height="608" alt="Attack map" src="https://github.com/user-attachments/assets/0cf15fcd-3a4d-4f27-846a-7f1edf2720f8" />

**Accounts with more than 10 failed logins**

<img width="1090" height="594" alt="KQL more than 10 failures" src="https://github.com/user-attachments/assets/5c92f9ed-88ba-467f-a0ae-c6a1ced4bdbd" />

---

## 🚨 Step 7: Detection Rule and Incidents

I created a new analytics rule for repeated failed logons.

<img width="1040" height="543" alt="Analytics rule 1" src="https://github.com/user-attachments/assets/69e0b75d-b43a-46b2-a6b6-9d4e004b5f55" />
<img width="608" height="426" alt="Analytics rule 2" src="https://github.com/user-attachments/assets/8b5cd83b-795a-4b27-84c6-6355c1835aa2" />

After a few minutes, alerts appeared on the **Incidents** page.

<img width="1090" height="540" alt="Incidents" src="https://github.com/user-attachments/assets/acaba3ef-d655-4644-842f-14495863e881" />

**Repeated failed authentication detected**

<img width="1090" height="578" alt="Repeated failed auth" src="https://github.com/user-attachments/assets/98790f0b-1c32-4a25-8e11-5185175086de" />

**Failed attempts from one IP (`80.94.95.83`)**

<img width="1090" height="721" alt="Attempts from one IP" src="https://github.com/user-attachments/assets/8639bb29-0b3c-4e4c-87bb-bc943250bae8" />

**Did the attacker get in?** I checked for a successful logon.

<img width="1090" height="728" alt="Successful logon check" src="https://github.com/user-attachments/assets/9b0b79d7-fe55-4919-b1a6-ea90394dc864" />

---

## 🤖 Step 8: Automated Response with a Logic App

I designed a Logic App that automatically blocks any IP that crosses the failed-login threshold (**10 attempts**).

<img width="1090" height="642" alt="Logic App design" src="https://github.com/user-attachments/assets/b926b47b-87fa-48fa-afc5-032cee4735ae" />

**Test result:** after a few minutes, IP `14.241.68.109` passed the threshold and was blocked automatically.

<img width="1090" height="651" alt="Auto block" src="https://github.com/user-attachments/assets/d3daa372-b53e-476c-8088-50ca0cd99d7e" />

The NSG inbound rules now show a new rule that denies all traffic from that IP.

<img width="1090" height="543" alt="NSG deny rule" src="https://github.com/user-attachments/assets/acbce2a4-a56f-47c3-9783-1f3e6feb0815" />

### 🐛 Issue Found and Fixed

| | |
|---|---|
| **Problem** | The blocked IP could still attempt failed logons. |
| **Cause** | My original allow-any rule had priority **100**, which is the highest. It was evaluated before the auto-block rule. |
| **Fix** | I changed the priorities so the auto-block rule is evaluated first. |

Inbound rules after the fix:

<img width="1090" height="149" alt="Fixed rule priorities" src="https://github.com/user-attachments/assets/f985cae9-d27f-4b05-ab36-ca0bafd9741b" />

**Result:** the block works. The last attempt was at **08:04:03 UTC**, the block took effect at **08:05 UTC**, and no attempts appeared after that.

<img width="1090" height="673" alt="Attempts stopped" src="https://github.com/user-attachments/assets/8517fe9f-f8b3-40fe-a735-870ae5339111" />

---

## 📊 Step 9: Dashboard

Finally, I built a dashboard to visualize the attack activity.

<img width="1090" height="515" alt="Dashboard 1" src="https://github.com/user-attachments/assets/6449a90f-3012-4409-9ca7-b7bef80654c3" />

<img width="1090" height="276" alt="Dashboard 2" src="https://github.com/user-attachments/assets/97745184-c316-4cac-82b0-f02762ee241a" />

<img width="1090" height="291" alt="Dashboard 3" src="https://github.com/user-attachments/assets/d00e2299-2804-416f-818d-73ccbc33908b" />

<img width="1090" height="282" alt="Dashboard 4" src="https://github.com/user-attachments/assets/51ed15b7-ccc7-430e-8b83-d626c7338553" />

<img width="1090" height="294" alt="Dashboard 5" src="https://github.com/user-attachments/assets/c7da926f-13fc-49fb-a3e1-09b23226a63d" />

---

## 💡 Key Takeaways

- NSG rules are evaluated by priority, and the lowest number wins. An automated deny rule must have a lower number than any broad allow rule.
- Enriching logs with GeoIP turns raw IPs into a usable attack map.
- Analytics rules plus Logic Apps give detection and response with no manual work.

## 👤 Author

**Izaan Shumaiz**  
Cybersecurity | Cloud | SOC
