# Cloud SOC Lab: Detection and Automated Response with Microsoft Sentinel

A small cloud SOC built in Microsoft Azure. A Windows honeypot VM is exposed to the
public internet, its logs are collected into Microsoft Sentinel, a custom KQL rule
detects brute-force login activity, and an Azure Logic App automatically blocks the
attacking IP address at the network layer.

**Prepared by:** Izaan Shumaiz
**Environment:** Microsoft Azure, Microsoft Sentinel

**Tools:** Azure Virtual Machines, Network Security Groups, Log Analytics, Microsoft
Sentinel, KQL, Azure Logic Apps

**Based on:** [Josh Madakor's SOC honeypot lab](https://www.youtube.com/@JoshMadakor) —
the honeypot VM, log collection pipeline, and GeoIP attack map follow his walkthrough.
Everything from the detection rule onward (analytics rule tuning, incident automation,
the Logic App auto-block, the NSG priority fix, and the dashboard) is my own extension
of the base project.

📄 **[Read the full report (PDF)](https://github.com/izaan-sh/Cloud-SOC-Lab-Microsoft-Sentinel/blob/main/SOC_Sentinel_Lab_Report.pdf)**

---

## Architecture

```
Windows VM → Azure Monitor Agent → Log Analytics Workspace → Microsoft Sentinel
   → KQL detection rule → Sentinel incident → Automation rule → Logic App
   → NSG deny rule → attacking IP blocked
```

![Honeypot architecture](screenshots/04-architecture-honeypot.png)

## Results

| Metric | Value |
|---|---|
| Failed Windows logons | 26,600 |
| Unique source IPs | 97 |
| Accounts targeted | 187 |
| Sentinel incidents generated | 154 |
| Successful Windows logons | 177 |
| Automated block verified | Yes — last failed attempt 8:04 UTC, blocked 8:05 UTC |

---

## 1. Setting Up the Environment

Logged into Azure and created a resource group named `RG-SOC-Lab`, then created a
virtual network inside it.

![Resource group and virtual network](screenshots/01-resource-group-vnet.png)

Created a VM and gave it an ordinary, business-sounding name instead of something
obviously suspicious, so it would look normal to attackers: `CORP-NET-EAST-1`.

![VM created](screenshots/02-vm-created.png)

This is what it looks like once deployed:

![VM overview](screenshots/03-vm-overview.png)

## 2. Exposing the Honeypot

This is the VM's Network Security Group (NSG) — the cloud firewall in front of it:

![NSG overview](screenshots/05-nsg-overview.png)

I removed the default RDP-only inbound rule and created a new inbound rule allowing
**any** traffic, so the VM would be discoverable by anyone scanning the internet:

![Adding an inbound rule to allow any traffic](screenshots/06-nsg-add-rule.png)

Connected to the VM over RDP:

![Connecting over RDP](screenshots/07-vm-connect-rdp.png)

Then turned off the Windows Firewall entirely:

![Turning off the firewall](screenshots/08-firewall-off-1.png)
![Firewall confirmed off](screenshots/09-firewall-off-2.png)

Pinged the VM from my own device to confirm it was reachable over the public internet:

![Ping test](screenshots/10-ping-test.png)

This confirmed the connection.

## 3. First Failed Logins

Logged out, then intentionally failed a login with a different username and the wrong
password a few times, to confirm Windows would log the activity. Logged back in with
the real credentials afterward and checked Event Viewer:

![Event Viewer, failed logon events highlighted](screenshots/11-event-viewer-list.png)
![Event 4625 detail](screenshots/12-event-4625-detail.png)

Event ID 4625 is a failed logon — this became the basis for everything that follows.

## 4. Building the Log Pipeline

### Log Analytics Workspace

Created a Log Analytics Workspace to act as the central log repository, named
`LAW-soc-lab-0000`.

![Log Analytics Workspace created](screenshots/13-law-created.png)

### Microsoft Sentinel

Added Microsoft Sentinel on top of the workspace.

![Sentinel added to the workspace](screenshots/14-sentinel-guides.png)

At this point the VM and the workspace weren't connected yet:

![Not yet connected](screenshots/15-architecture-sentinel-notconnected.png)

### Azure Monitor Agent

To connect them, I configured the Azure Monitor Agent (AMA) Windows Security Events
connector:

![AMA connector missing](screenshots/16-architecture-ama-missing.png)

Added the connector and the extension installed on the VM:

![Security events connector added](screenshots/17-ama-extension-installed.png)

After a few minutes, logs started arriving in the workspace:

![Logs appearing](screenshots/18-logs-appearing-1.png)
![Logs appearing, more detail](screenshots/19-logs-appearing-2.png)
![Sample of collected events](screenshots/20-logs-sample.png)
![Event ID reference](screenshots/21-event-id-reference.png)

## 5. GeoIP Enrichment

By default, `SecurityEvent` only gives you an IP address — no location. To fix that, I
imported a GeoIP CSV as a Sentinel watchlist, so KQL could resolve each attacker's IP to
a city and country:

Source: [`geoip-summarized.csv`](https://raw.githubusercontent.com/joshmadakor1/lognpacific-public/refs/heads/main/misc/geoip-summarized.csv)

![Importing the GeoIP watchlist](screenshots/22-geoip-watchlist-import.png)
![Watchlist query](screenshots/23-geoip-watchlist-query.png)

With the watchlist in place, the same failed-logon query now resolves a real location:

```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where IpAddress == "<attacker IP>"
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
```

![Failed logons enriched with location data](screenshots/24-kql-geoip-enrichment.png)

Renamed the columns to be clearer (e.g. `AttackerIP = IpAddress`):

![Renamed columns](screenshots/25-kql-renamed-columns.png)

I also checked which workstation names and IPs were generating failed logons — this is
where I noticed a few of the "attempts" were actually my own manual tests from earlier,
not external attackers, which mattered later when interpreting the results:

![Reviewing attacker workstation names](screenshots/26-kql-attacker-workstation.png)

Finally, built the attack map:

![Attack map](screenshots/27-attack-map.png)

## 6. Investigating the Activity

Before writing any detection logic, I spent time exploring the data manually — grouping
failed attempts by IP and account to see what real attack traffic actually looked like:

![Grouping failed attempts by IP](screenshots/28-kql-failcount-5.png)
![Checking for 4624/4625 together](screenshots/29-kql-4624-4625.png)
![IPs with 10+ failed attempts](screenshots/30-kql-10-failed.png)
![IPs and accounts, 5-minute buckets](screenshots/31-kql-5-failed-groups.png)

## 7. Building the Detection Rule

From this investigation, I built a repeatable Sentinel analytics rule rather than
searching for suspicious IPs by hand each time. The deployed rule flags any source IP
responsible for **20 or more failed network logons within a five-minute window**:

```kql
SecurityEvent
| where EventID == 4625
| where LogonType == 3
| where isnotempty(IpAddress)
| summarize FailedAttempts = count(),
            FirstSeen = min(TimeGenerated),
            LastSeen = max(TimeGenerated)
    by IpAddress, bin(TimeGenerated, 5m)
| where FailedAttempts >= 20
```

![Creating the analytics rule](screenshots/32-analytics-rule-review.png)
![Rule details: entity mapping, custom details, incident settings](screenshots/33-analytics-rule-details.png)

After a few minutes, alerts started appearing in the Incidents page:

![Incident generated](screenshots/34-incident-overview.png)

I then dug into one incident specifically to confirm it wasn't a single failed login,
but sustained, repeated authentication failure:

![Investigating a 10+ threshold pattern](screenshots/35-kql-10-threshold-investigation.png)
![All events from a single attacking IP](screenshots/36-kql-single-ip-events.png)
![Total attempts from that IP](screenshots/37-kql-ip-summary.png)
![Accounts targeted by that IP](screenshots/38-kql-account-summary.png)

### Did the attacker get in?

Checked whether any of the investigated IPs ever produced a successful logon
(Event ID 4624) after their failed attempts:

![Checking for a successful logon](screenshots/39-kql-check-successful-logon.png)
![Checking whether the attack was still active](screenshots/40-kql-active-check.png)

No successful logon was found from the IPs I checked this way.

## 8. Automated Response with a Logic App

Manually investigating and blocking every malicious IP wouldn't scale, so I built a
Logic App, `SOC-AutoBlock-Malicious-IP`, that responds automatically:

**Sentinel incident → For each entity → Condition (is it an IP?) → Compose (extract the
IP) → Create/update an NSG deny rule → comment on the incident**

![Logic App designer](screenshots/41-logic-app-designer.png)

### Troubleshooting

- **Wrong entity path** — the trigger looked for `Entities` at the top level, which
  returned nothing. Fixed by inspecting the actual payload and using the correct path:
  `object → properties → relatedEntities`.
- **Case-sensitive match** — the condition checked for `"ip"`, but Sentinel sends
  `"Ip"`. Fixed by matching the exact case.
- **Wrong nesting** — the Condition step was sitting outside the For Each loop instead
  of inside it. Fixed by moving it inside.
- **Missing permission on comments** — adding a comment to the incident returned a
  403 error. Fixed by granting the Logic App's identity a separate Sentinel
  incident-management permission, in addition to its NSG (Network Contributor)
  permission.

Once an IP crossed the rule's threshold, the Logic App ran successfully end to end and
extracted the attacking IP:

![Logic App run: IP extracted](screenshots/42-logic-app-run-success.png)

The block worked — IP `14.241.68.109` exceeded the threshold and was auto-blocked:

![Auto-block confirmation](screenshots/43-nsg-block-rule.png)

Checking the NSG confirms a new inbound rule was created denying all traffic from that
IP — Azure NSG is now configured to deny matching inbound traffic from that source.

### Troubleshooting: the block wasn't fully effective at first

Even after the block rule existed, the same IP kept getting through. The cause: back at
the start of the project I'd created an inbound rule allowing *any/any/any* traffic at
**priority 100** — the highest priority, meaning it was evaluated first and let the
traffic through before the deny rule ever got a chance to apply.

Fixed it by lowering that rule's priority so the auto-block rule (priority 4000) takes
precedence:

![NSG priority fixed](screenshots/44-nsg-priority-fix.png)

After the fix, I confirmed the login attempts stopped and stayed blocked. The last
failed attempt was at **08:04 UTC**; the block took effect at **08:05 UTC**, and no
further attempts were logged after that.

![Verifying the block held](screenshots/45-kql-verify-block.png)

## 9. Dashboard

Finally, built a Sentinel workbook to summarize the whole project's activity:

![Dashboard overview](screenshots/46-dashboard-overview.png)
![Top source IPs by failed logons](screenshots/47-dashboard-top-ips.png)
![Most targeted accounts](screenshots/48-dashboard-top-accounts.png)
![Source IP and account activity](screenshots/49-dashboard-ip-account-table.png)
![Recent Sentinel incidents](screenshots/50-dashboard-recent-incidents.png)

---

## Credits

- [Josh Madakor](https://www.youtube.com/@JoshMadakor) — original SOC honeypot lab
  concept, log collection setup, and GeoIP watchlist source.
- Detection rule, incident automation, Logic App auto-block, NSG priority fix, and
  dashboard extensions are my own work.
