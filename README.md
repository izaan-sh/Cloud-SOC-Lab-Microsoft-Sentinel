# Cloud-SOC-Lab-Microsoft-Sentinel
<h2>Detection and Automated Response with Microsoft Sentinel </h2>
Prepared by: Izaan Shumaiz 
Environment: Microsoft Azure, Microsoft Sentinel

<img width="1090" height="496" alt="image" src="https://github.com/user-attachments/assets/fa55f5df-8f8c-4c18-80a1-900b56effae3" />



Logged into Azure and created a resource group and named it “RG-SOC-Lab”
Then created a virtual network inside:

<img width="1076" height="248" alt="image" src="https://github.com/user-attachments/assets/151e6fb7-e8db-48f6-8309-681934cae491" />



And then created a vm and named it with a not so suspicious name, should look normal to attackers, something like :  “CORP-NET-EAST-1”

<img width="1090" height="390" alt="image" src="https://github.com/user-attachments/assets/78599ffe-5cf6-4d7e-83e5-2c67cd6c8c55" />



This is how it looks like:

<img width="940" height="492" alt="image" src="https://github.com/user-attachments/assets/24018ec8-a92f-44b0-be46-201e1a48a216" />



Network Security Groups: 

<img width="940" height="493" alt="image" src="https://github.com/user-attachments/assets/f483bec7-219c-44a7-bf8d-22f08ed6aa4c" />



Removing the RDP inbound rule and creating a new inbound rule that will allow "any" traffic

<img width="940" height="493" alt="image" src="https://github.com/user-attachments/assets/5c17443f-73c5-43d8-8cb0-0bacc246902c" />



Adding the new inbound rule: 

<img width="940" height="1211" alt="image" src="https://github.com/user-attachments/assets/05475223-da2e-4ad9-9641-48c9e9ecdd9f" />



Next we use RDP to access our VM 

<img width="940" height="491" alt="image" src="https://github.com/user-attachments/assets/7ec4cfb2-c594-4321-ace5-41def16d3caa" />



Next turning off all firewalls

<img width="536" height="603" alt="image" src="https://github.com/user-attachments/assets/aad7ff46-a90b-45f2-b04e-4477e97158bf" />
<img width="545" height="455" alt="image" src="https://github.com/user-attachments/assets/3baf018e-cb2e-490e-925a-09779d9bf514" />



Pinging to see if its connected:

<img width="1090" height="571" alt="image" src="https://github.com/user-attachments/assets/47198685-a210-4018-9aba-6b1b685bedb0" />

This confirms the connection


Then logged out of the RDP session and tried logging back into it with different username and password (about 4 times)

This was done to see if the event viewer could show us the failed log in attempts under the security logs

<img width="1090" height="656" alt="image" src="https://github.com/user-attachments/assets/b3971eab-017e-4461-bc7c-5d6878a0169f" />

<img width="1090" height="1336" alt="image" src="https://github.com/user-attachments/assets/37062366-f089-43f9-a4f3-de53f32a3f06" />



Next we're going back to Sentinel, we are going to configure a log repository and then forward our vm logs - basically creating a Log Analytics Workspace
Created it and named it "LAW-soc-lab-0000"

<img width="1090" height="394" alt="image" src="https://github.com/user-attachments/assets/6a35b1a3-4493-4885-8991-e0cb4f06070f" />



Next up: Creating our sentinel instance

Just added sentinel to the workspace:

<img width="1090" height="532" alt="image" src="https://github.com/user-attachments/assets/9684d5e7-9316-4140-a1d7-90d12ebfe7bd" />



Next we need to connect the log analytics workspace with our VM. For that, we have to configure the Azure Monitoring Agent Security Event Connector

<img width="1090" height="496" alt="image" src="https://github.com/user-attachments/assets/fcc896c9-720e-4eac-b49d-88f91e550fbc" />

(red circled *connection is what were trying to get right now, as its not connected. We do so in the next step)



Added the security event connector

<img width="1090" height="524" alt="image" src="https://github.com/user-attachments/assets/f3c669d3-83ec-481e-b765-6d7a284a16e0" />



Now we're going to watch the logs, we will have to wait a few minutes for the logs to appear.

After a few minutes we have started to see some logs: 

<img width="1090" height="529" alt="image" src="https://github.com/user-attachments/assets/43b619bb-4274-41ad-88bb-4106b425b0a3" />



KQL to find 10 Security Events

<img width="1090" height="627" alt="image" src="https://github.com/user-attachments/assets/15c29408-e3f0-4509-86b9-f1deac1e9822" />



Next we are going to add this geoip spreadsheet into the watchlist

from here: https://raw.githubusercontent.com/joshmadakor1/lognpacific-public/refs/heads/main/misc/geoip-summarized.csv

<img width="1090" height="722" alt="image" src="https://github.com/user-attachments/assets/c44021e4-d502-42a8-9e9e-3cbf7b4d355e" />



Successfully added it, and when we check the logs, we can see the actual location too:

<img width="1090" height="516" alt="image" src="https://github.com/user-attachments/assets/571b2eb4-2fed-415b-b509-792a7b064bf7" />

KQL to find security events from IPAddress "82.114.228.224" where the eventID is 4625 (Failed Logons), ordering it by the time it occurred:

<img width="1090" height="546" alt="image" src="https://github.com/user-attachments/assets/2c316af0-0664-49d1-b3c9-f12fca687bdb" />

Another KQL entry to project necessary columns, and changing the names to be shown for the columns: 

<img width="1090" height="501" alt="image" src="https://github.com/user-attachments/assets/54d3c95f-11b1-474d-ae08-df3e17cb957c" />

Finally we check the map : 

<img width="1090" height="608" alt="image" src="https://github.com/user-attachments/assets/0cf15fcd-3a4d-4f27-846a-7f1edf2720f8" />

KQL Query to look for accounts that had failed login attempts of more than 10 times. 

<img width="1090" height="594" alt="image" src="https://github.com/user-attachments/assets/5c92f9ed-88ba-467f-a0ae-c6a1ced4bdbd" />

Next we are going to create a new rule:

<img width="1040" height="543" alt="image" src="https://github.com/user-attachments/assets/69e0b75d-b43a-46b2-a6b6-9d4e004b5f55" />
<img width="608" height="426" alt="image" src="https://github.com/user-attachments/assets/8b5cd83b-795a-4b27-84c6-6355c1835aa2" />


After a few minutes we could see alerts in the incidents page:

<img width="1090" height="540" alt="image" src="https://github.com/user-attachments/assets/acaba3ef-d655-4644-842f-14495863e881" />

Detection of repeated failed authentication:

<img width="1090" height="578" alt="image" src="https://github.com/user-attachments/assets/98790f0b-1c32-4a25-8e11-5185175086de" />


Number of failed attempts from this exact ip “80.94.95.83” :

<img width="1090" height="721" alt="image" src="https://github.com/user-attachments/assets/8639bb29-0b3c-4e4c-87bb-bc943250bae8" />


Checking weather the attacker actually got in or not:

<img width="1090" height="728" alt="image" src="https://github.com/user-attachments/assets/9b0b79d7-fe55-4919-b1a6-ea90394dc864" />

Next we are going to desing a Logic App - basically to autoblock IPAddresses that crosses the threshold for number of failed logins

<img width="1090" height="642" alt="image" src="https://github.com/user-attachments/assets/b926b47b-87fa-48fa-afc5-032cee4735ae" />

To test if the block works, we are going to wait for an IP to get to the threshold (10 failed login attempts) 

After a few minutes, we could see an automatic block for the IP ‘14.241.68.109’ 
This IP Address had more failed login attempts than the threshold, so now that IP Address has been autoblocked by the rule:

<img width="1090" height="651" alt="image" src="https://github.com/user-attachments/assets/d3daa372-b53e-476c-8088-50ca0cd99d7e" />

If we check the NSG Inbound Rules, we can a new inbound rule created to deny all traffic from this IP: 

<img width="1090" height="543" alt="image" src="https://github.com/user-attachments/assets/acbce2a4-a56f-47c3-9783-1f3e6feb0815" />

This means Azure NSG is configured to deny matching inbound traffic from that source IP.

New Issue Found:
Even after the blocking, the same IP traffic was still flowing. The same IP was able to attempt more failed logons. This was because in the beginning of the project I had created an inbound security rule to allow any any any traffic, and this had a priority of 100 (which is the highest.), so that’s why this inbound rule was letting that ip to attempt more, then I went on to change the rule priority so that the autoblock inbound rule has a higher priority than this one. 


After giving the autoblock rules a higher priority, this is how the inbound rules table looks like. 

<img width="1090" height="149" alt="image" src="https://github.com/user-attachments/assets/f985cae9-d27f-4b05-ab36-ca0bafd9741b" />


And now we can confirm that after the block, the login attempts has stopped and been blocked successfully

<img width="1090" height="673" alt="image" src="https://github.com/user-attachments/assets/8517fe9f-f8b3-40fe-a735-870ae5339111" />

The latest attempt was at UTC 8:04:03 and the block worked at UTC 8:05 and after that, no attempts were seen.

Finally, we will be making the dashboard

<img width="1090" height="515" alt="image" src="https://github.com/user-attachments/assets/6449a90f-3012-4409-9ca7-b7bef80654c3" />

<img width="1090" height="276" alt="image" src="https://github.com/user-attachments/assets/97745184-c316-4cac-82b0-f02762ee241a" />

<img width="1090" height="291" alt="image" src="https://github.com/user-attachments/assets/d00e2299-2804-416f-818d-73ccbc33908b" />

<img width="1090" height="282" alt="image" src="https://github.com/user-attachments/assets/51ed15b7-ccc7-430e-8b83-d626c7338553" />

<img width="1090" height="294" alt="image" src="https://github.com/user-attachments/assets/c7da926f-13fc-49fb-a3e1-09b23226a63d" />







