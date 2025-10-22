
# 🔒 Creating a Honeypot in Microsoft Sentinel SOC

## 📌 Overview  
This project simulates a real-world Security Operations Center (SOC) workflow by deploying a honeypot in Microsoft Azure, forwarding logs into Microsoft Sentinel, and detecting brute force login attempts. The environment demonstrates end-to-end threat monitoring, log analysis with KQL, and real-time attack visualization on a global scale.

---

## 🛠️ Tools & Technologies  
- Microsoft Azure
- Windows 11 Pro VM
- Microsoft Sentinel (Cloud-based Security Information and Event Management tool)
- Microsoft Log Analytics Workspace  
- Windows Event Viewer
- Kusto Query Language (KQL)
- JSON Attack Map
- Geolocation Data CSV 

---

## Step 1: Create a Free Microsoft Azure Account
1. Navigate to the <a href="https://azure.microsoft.com/">Microsoft Azure web portal</a> to create an account (TIP: Do not use your work/school email for account creation)
2. Azure should provide you with a $200 credit to begin; if it doesn't, sign up for a paid subscription, just make sure to cancel it once you are finished so you don't rack up a bill.

## Step 2: Create New Resource Group
An Azure resource group is a logical container in Microsoft Azure that holds related resources for an application or project.
1. In the search bar of the Microsoft Azure portal, type in <b>Resource groups</b> and select the resource groups option
2. Select <b>Create</b> in the top left corner.
   - The subscription should already be filled in with your active Azure subscription
   - Under <b>Resource Group Name</b>, title it whatever you want your project to be named. I titled mine "ET-SOC-LAB"
   - For <b>Region</b>, I selected (US) East US 2

## Step 3: Create New Virtual Network
An Azure Virtual Network (VNet) is similar to a traditional physical network in your own datacenter. It is essential for isolating and securing your Azure resources.

1. In the search bar of the Microsoft Azure portal, type in <b>Virtual Networks</b> and select the Virtual Networks option
2. Select <b>Create</b> in the top left corner.
   - The subscription should already be filled in with your active Azure subscription
   - Under <b>Resource Group</b>, select the one we created in Step 2. (If you aren't seeing it, it likely hasn't finished deploying. Check the progress on the Resource Groups page)
   - For <b>Virtual Network Name</b>, you can name it whatever you want. I named mine "Vnet-soc-lab"
   - Region should stay as (US) East US 2
3. Select <b>Review + Create</b> (you don't need to change any other options)

## Step 4: Create the Honeypot VM
A honeypot VM (virtual machine) is a decoy computer system on a network, hosted within a virtual environment, that is designed to attract and trap cyber attackers. By mimicking real systems and appearing vulnerable, the honeypot VM diverts attackers from production systems, while simultaneously allowing security teams to monitor their activities and gather intelligence on their methods, motivations, and tools. 

1. In the search bar of the Microsoft Azure portal, type in <b>Virtual Machines</b> and select the Virtual Machines option
2. Select <b>Create</b> in the top left corner.
   - The subscription should already be filled in with your active Azure subscription
   - Under <b>Resource Group</b>, select the one we created in Step 2.
   - For <b>Virtual Machine Name</b>, you can name it whatever you want. I named mine "rushvm" after my last name.
   - Region should stay as (US) East US 2
   - For <b>Image</b>, select <b>Windows 10 Pro, version 22H2 - x64 Gen2</b> for the purposes of this lab.
   - <b>Size</b> should be <b>Standard_D2s v6</b> (if you pick any lower than this, your VM will run incredibly slow)
   - Under <b>Administrator Account</b>, create a username and password combination you will remember (I won't tell you mine here, hehehe)
   - Check the box next to <b>I confirm I have an eligible Windows 10/11 license with multi-tenant hosting rights.</b>
3. On the <b>Networking</b> tab of the creation screen, check the box next to <b>Delete NIC when VM is deleted</b>
4. Select <b>Review + Create</b> (you don't need to change any other options)

## Step 5: Configure Network Security Group to Allow Attackers
An Azure Network Security Group (NSG) is most comparable to a basic, stateful packet-filtering firewall or an Access Control List (ACL). It allows you to control network traffic by defining rules that permit or deny inbound and outbound connections for network resources. For this step, we need to configure the Network Security Group to allow ANY inbound traffic into our VM, intentionally exposing it to attackers. 

1. In the search bar of the Microsoft Azure portal, type in <b>Network Security Groups</b> and select the Network security groups option
2. Select the item that has your VM name followed by nsg.
3. The first rule under <b>Inbound Security Rules</b> allows inbound communication over Port 3389, the Remote Desktop Protocol. We need to change this to allow access from any port.
  - On the far right, select the <b>trash can</b> next to the RDP rule to delete it.
  - On the left-side menu, click <b>settings > inbound security rules </b>
  - On the top left, select <b>Add</b>
  - Change <b>Destination port ranges</b> to the wildcard character, <b>*</b>
  - For the <b>Name</b>, enter whatever you want. I named mine "DANGER_AllowAnyCustomAnyInbound"
  - Save the rule by pressing <b>Add</b>

## Step 6: Connect to Our Virtual Machine
The following instructions are for <b>MacOS systems</b> only. To access your virtual machine via Windows, see <a href="https://learn.microsoft.com/en-us/azure/virtual-machines/windows/connect-rdp">these instructions.</a>

1. You will first need the <b>Public IP address</b> of your virtual machine.
   - In the Azure dashboard, search for <b>Virtual Machines</b> and navigate to it
   - You should see the VM you created listed. On the right, there should be a column titled <b>Public IP address</b>. Copy that address.
2. Open up the <b>Windows App</b> application. (If you don't have this, navigate to the App Store and install it)
3. In Windows App, click the <b>Plus</b> icon and select <b>Add PC</b>.
4. Paste the Public IP Address in the <b>PC Name</b> and <b>Friendly Name</b> fields. Then click <b>Add</b>
5. Windows App should have added your machine. Now when you double click it, it should prompt you to log in with the username and password you created. Go ahead and do so. (If it prompts you to change any settings, just click OK, you don't need to)
6. You should now be inside your Windows VM!

## Step 7: Configure Windows Defender Firewall
We've configured the Network Security Group to enable inbound traffic on any port; however, our Virtual Machine is running Windows 10, meaning it also has its own Firewall configured. We need to disable this to completely open our system and make it vulnerable. 
1. In the search bar, type in <b>Firewall</b> and select <b>Windows Defender Firewall with Advanced Security</b>
2. Select <b>Windows Defender Firewall Properties</b> and change the <b>Firewall state</b> to <b>off</b> on each tab. (Tabs include Domain Profile, Private Profile, and Public Profile)
3. Hit <b>Apply</b> and then <b>OK</b>
4. You can test if the VM's Firewall has been disabled by running a ping test on it.
     - If you are on a Mac host machine, open up <b>Terminal</b>. For Windows, open <b>Command Line</b>
     - Type <b>ping <Your VMs Public IP Address</b> and then hit enter
     - If the Firewall has been disabled correctly, you should see pings coming through on the terminal every couple of seconds. (See example below)

## Step 8: Create Log Repository
In your Windows VM, you can navigate to Event Viewer and view security events that show unauthorized and failed login attempts on your machine. These should start coming through anytime between now and when you decide to shut the VM down (I recommend 12-24 hours for good results). You are looking for events with the EventID <b>4625</b>. What we need to do now is create a repository in Azure that can ingest these logs from our virtual machine for further analysis in our SIEM. 

1. In the Azure dashboard, search for <b>Log Analytics Workspace</b>
2. Select <b>Create</b> in the top left corner.
   - The subscription should already be filled in with your active Azure subscription
   - Under <b>Resource Group</b>, select the one we created in Step 2. 
   - For <b>Name</b>, you can name it whatever you want. I named mine "LAW-soc-lab-0000"
   - Region should stay as (US) East US 2
3. Select <b>Review + Create</b> (you don't need to change any other options)

## Step 9: Create Microsoft Sentinel Instance
Microsoft Sentinel is a cloud-native, Security Information and Event Management (SIEM) solution from Microsoft that provides intelligent security analytics and threat intelligence to collect, detect, respond to, and investigate security threats across an organization's entire IT estate.
1. In the Azure dashboard, search for <b>Microsoft Sentinel</b>
2. Select <b>Create</b> in the top left corner.
   - Select the Workspace you created in the previous step and <b>Add</b> (If you don't see your Workspace, it is likely still deploying. Try refreshing in a bit)
  
3. Return to the <b>Microsoft Sentinel</b> dashboard (you can click the breadcrumb or just search for it again)
4. Select your Sentinel Instance
5. Navigate to <b>Content Hub</b> on the left menu
6. Search for <b>Windows Security Events</b> and select the checkbox next to it.
7. With Windows Security Events selected, hit the <b>Install</b> button on the right.
8. Once it is done installing, you should see a <b>Manage</b> button where the Install button was. Click that.
9. Select the checkbox next to <b>Windows Security Events via AMA</b>
10. On the bottom right, select <b>Open Connector Page</b>
11. Select <b>Create Data Collection Rule</b> (This is what will forward the logs from your VM to the LAW)
    - For <b>Rule Name</b> you can name it whatever. I named mine "DCR-Windows"
    - The subscription should already be filled in with your active Azure subscription
    - Under <b>Resource Group</b>, select the one we created in Step 2.
    - Click <b>Next</b>
    - Check the box next to our Virtual Machine
    - Hit <b>Create</b>

## Step 10: Test Our Log Ingestion
At this point, our security logs should be ingesting from our VM into the Log Analytics Workspace since we have added the collection rule. Let's test it by querying some of the data with Kusto Query Language (KQL)
1. In the Azure dashboard, search for <b>Log Analytics Workspace</b>
2. Select our LAW instance
3. On the left menu panel, select <b>Logs</b> (If the Queries hub page opens, click the "X" to close it)
4. On the top right, change the box that says <b>Simple Mode</b> to <b>KQL Mode</b> (This allows you to use the Kusto Query Language to query results from the logs that are being ingested. Similar to SQL functionality with different syntax)
5. Type in the following in the Query box and hit <b>Run</b>
```
SecurityEvent
```
  - You should see data in the Results pane once the query has finished. If you don't, give the system about twenty minutes. Then, come back and try again. If you still are not seeing any logs, ensure your VM is running and review the previous steps.

6. The security events of interest in this lab are those that have the <b>EventID of 4625</b> - which are failed logon attempts. These could be indicators of brute force attacks, password spraying, rainbow tables, etc..
  - Update the query to the following and hit <b>Run</b>:


```
SecurityEvent
| where EventID == 4625
```

7. Now, you should see a list of all the instances with failed login attempts. You can also view their IP addresses in the results table. (If you see a large volume, don't be alarmed, this is normal!)
8. You can search these IP addresses on https://ipgeolocation.io/ to see where in the world they are originating from.

## Step 11: Create IP Address Watchlist
In Microsoft Azure, a watchlist is a customizable list of data that you can upload as a CSV file to Azure Sentinel. It's used to enrich events within your security data, allowing you to correlate information like high-value assets, terminated employees, or specific IP addresses with your security logs.

1. In the Azure dashboard, search for <b>Microsoft Sentinel</b>
2. Select your Sentinel instance
3. Under <b>Configuration</b> on the left menu, navigate to <b>Watchlist</b> (Azure may have you navigate to the Defender Portal. This is fine. Instructions are the same)
4. <a href="https://drive.google.com/file/d/13EfjM_4BohrmaxqXZLB5VUBIz2sv9Siz/view">Download this CSV file </a> containing a list of IP address ranges and their geolocations.
5. In the <b>Watchlist Dashboard</b>, select <b>New</b>
6. For <b>Name</b> and <b>Alias</b>, you can name it whatever you want. I named mine "geoip". Then click <b>Next</b>
7. Ensure the <b>Type</b> is "Local File" and <b>SearchKey</b> is "Network". Then upload the downloaded CSV.
8. Select <b>Review + Create</b> (you don't need to change any other options)
9. This step can take a while to upload as there are over 50,000 rows being uploaded in the CSV. Go get some coffee and come back! 
10. Go back to step 10 and repeat the steps to query for data where the EventID is 4625.
11. Find an attacker's IP address and copy it. 
12. Then, replace your code with the following, replacing the <b><Attacker IP></b> with the IP address you copied.

```
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where IpAddress == "<Attacker IP>"
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
```

## Step 12: Create Attack Map
Now, we will use a JSON-format global map and integrate it with our logs to see a global view of hotspots where malicious IPs are attempting to intrude on our VM. 

1. In the Azure dashboard, search for <b>Microsoft Sentinel</b>
2. Select your Sentinel instance
3. In the left menu pane, under <b>Threat Management</b>, select <b>Workbooks</b>
4. From here, select <b>Add Workbook</b>, then click the <b>Edit</b> button at the top. (You can delete anything already in the workbook if it isn't blank)
5. Click <b>Add > Add data source + visualization</b>
6. Open <a href="https://drive.google.com/file/d/1ErlVEK5cQjpGyOcu4T02xYy7F31dWuir/view">this file</a> and copy the JSON code
7. In the Workbook editor, navigate to the <b>Advanced Editor</b> tab and paste the JSON code. (You can delete any code currently in there)
8. Select <b>Done Editing</b> and then the <b>Save</b> button at the top. You can name your workbook anything. I named mine "Windows VM Attack Map".
9. Now, if you view the workbook, you can see a map of the world, with circles indicating where your attackers are located!

Leave your VM running for 12 to 24 hours and review the attack map again to see many more attackers. Then, use your analysis skills to query your logs and find more information! 


## 🚀 Skills Demonstrated  
- <b>Provisioned and secured cloud infrastructure</b> by deploying an Azure subscription with virtual machine resources configured for monitoring.
- <b>Implemented centralized logging</b> through the creation and configuration of an Azure Log Analytics Workspace to collect and normalize security events.
- <b>Integrated data pipelines with Microsoft Sentinel</b> by forwarding VM security logs to enable advanced correlation and threat detection.
- <b>Leveraged Kusto Query Language (KQL)</b> to identify repeated authentication failures and extract patterns of malicious activity.
- <b>Developed real-time attack visualizations</b> by building an interactive map that geolocates and tracks brute force attempts from global adversaries.

## Acknowledgement
- Huge shout out to thank Josh Madakor for creating this initial lab tutorial. Check out his YouTube <a href="https://www.youtube.com/@JoshMadakor">here</a>
