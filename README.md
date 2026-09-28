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










