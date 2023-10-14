# Soc-HoneyPot
This is my SIEM project created and hosted in Azure

 James Reeves SIEM (Security Information and Event Management) in Microsoft Azure. 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/99dcf58c-f902-4607-bec4-0b43702200f2)


This exercise will show you how to set up a SIEM and map where the attacks are coming from, using Microsoft Sentinel and Log Analytics Workspace (LAW).

Virtual Machine Setup/ SQL Server Setup
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/611ad268-1afc-4da4-a52e-3951f712db57)

 Create an Azure account/subscription.
 Create a resource group and create a virtual machine to go into that resource group; be sure to make sure both are in the same region.
   Create 2 windows virtual machines: one "WindowsVM" and the other "AttackVM". The windowsVM will be the main VM we store our SQL server on and view event manager. We will use attackVM to attack windowsVM so we can monitor the attacks on event viewer. The attackVM will be in its own resource group for this exercise and make sure the resource group and attackVM are in a different region than the windowsVM to simulate an outside attack from another part of the world. 

 


       Notice the “AttackVM” and the “Cyberlab(windowsVM)” are in separate resource groups and the locations (regions) are different.

 Next, we will disable the NSG (Network Security Group) on WindowsVM so that it is accessible and easy to attack for outside threats. We will be viewing these attacks on “Event Viewer”.
  . First go to the Cyberlab (WindowsVM) resource group:

 
    


Next; Select “WindowsVM-nsg”, we will create an inbound rule to allow all traffic into our windowsVM, inbound rules dictate what comes into our VM and outbound dictates what goes out. NSG (network security groups) in general; are firewall-like security groups for network traffic in Azure. A network security group is like a superhero that protects a special computer place in the sky called Azure from bad guys who try to break in. Just like how superheroes keep their city safe, NSG’s keep the computer place safe.
 




Next you will select “inbound security rules” under settings and then “Add” to create a new rule to allow all traffic into the windowsVM;

 



To all allow all traffic you are going to create the rule as follows
. Source =Any
. Source Port Ranges=Any or *
. Destination=Any
. Service=Custom
. Destination Port Ranges= * (which means any)
. Protocol=Any
. Action=Allow
. Priority=290. You want this priority at the highest level so it can get evaluated first. The lower the number; the higher priority
 . Name=AllowAllTraffic (this is optional, choose whatever name you want)
Then click add;

 



 







You should now see your new rule.  Allowing all traffic is bad for obvious reasons, but for the sake of the lab, you will see how many attackers are anxious to hack into your vm, and we will be able to track the attacks and eventually map them.

 


4) Now we are ready to remote into our WindowsVM and disable the actual firewall in our virtual machine. To do this, you will need the IP address of your virtual Machine. 
 


Copy the Public IP address and depending on whether you are using a windows pc or mac; that will determine how you remote into your vm. In this example, I am using windows. If you are using a mac you can use go to the app store and download “Windows Remote Desktop.” Start by opening the start menu and typing in “Remote Desktop”. When the application pops up, you will type the IP address in the “Computer” field. And click “connect”
 


Log into your WindowsVM with your username and password you created when creating your vm then click “ok”. The vm should load up like a normal PC coming on for the first time. 
 

 


We will now disable the firewall in order to make our VM more enticing and susceptible to cyber-attacks. To disable the firewall in the vm; open the windows menu and type in wf.msc 
 


Next click “Windows Defender Firewall Properties”
 

Turn off the firewall for the Domain Profile, Private Profile, and Public Profile, click “apply” and “ok” after all three are turned off.
 
 
Click “ok” and “Apply” to finalize shutting down the firewall 



Now we install SQL Server Evaluation edition on our WindowsVM so that we have something to monitor. Open edge and download from SQL Server 2022 | Microsoft Evaluation Center
 

Download the EXE 64-bit download and run the download
 


Choose the media option 

Select ISO and then download
 


Go to your download location and right click on the ISO and mount the iso to your computer.
 


Now that the ISO is mounted as a drive on your computer, run the setup.
 

Follow the screenshots below, these will guide you through the SQL server installation process:
 
 
 
 
 
 
 



Within the Database Engine Configuration; this is where you will create your username and password for your SQL server. By selecting “Mixed Mode” you enable the server to login with either windows or server credentials. The username by default will be “sa” so you can create whichever password you please.  Select “Add Current User” to add your current windows profile as the server admin. 
 
Finally Done install, you now have a SQL SERVER!!! 


Now that your server is downloaded, go back to the SQL Server Installation and install the SQL Server Management Tools
 

This will open a link to Microsoft.com, download the SSMS 


Run the SSMS setup
 

INSTALL 


Now that you installed SQL Server Management Studio, open the windows menu, type “SSMS” and open the SQL Server Management Studio
 

Connect to your server with the username and password you created during the installation of the SQL Server.
 


Next, we need to grant permission and enable the SQL server to be able to write to the windows security log and audit any failed or successful logins into our server. Right click on your WindowsVM server and select “Server Properties” and “Security” then check to have both failed and successful logins logged in windows security log. 




We will now enable auditing from the SQL server by running a script in the command prompt, this way the SQL server will start tracking the activty going on in the server. You can find the script at  Write SQL Server Audit events to the Security log - SQL Server | Microsoft Learn and then copy the script below.
 
Open the command prompt, run as an admin, and paste the script and run it. 
 


The last step to getting the SQL server to audit is granting permission to the server to write to the windows security log in event viewer. To do this we will need to add “Network Services” to the security folder in the registry and grant full access. Follow the screenshots below
 
Open windows menu and open “Registry Editor” 




Open the folders that corresponds to the highlighted path until you get to “Security” Right click and select “Permissions” 
Add “Network Services” to the “Group or Usernames” 







Allow “Full Control”, apply and ok. 
 


Restart server to finalize changes, SQL server can now audit events and log them into windows security log.
 



Now let’s test the log by creating some failed attempts at getting into the server. Disconnect from the server and then enter a bad login when attempting to reconnect. Open the “Event Viewer” open the “Windows Log”, select “Security” and the failed attempt should be logged, proving that the server is logging failed and successful attempts.
 
 

 


Now log into the AttackVM and download the SSMS (SQL Server Management Studio) just like we did on the WindowsVM. And we are going to generate some failed attempts and successful attempts against the WindowsVM Server.
 


We will open SQL Server Management tool and generate failed login attempts and a successful login. To login to WindowsVM server; copy the IP address of the WindowsVM and paste it into the “Server Name” field and then enter a bad password and username in order to generate a bad login attempt
 

Now enter the correct username and password to generate a successful login.
 


Now let’s  check the WindowsVM Event viewer and we should see both failed and successful logins from the Attack VM with the source coming from the Attack VM   IP address
 
 


Now that we know our SQL Server is auditing and successfully logging into windows security log, we can proceed to the next step, which is setting up Sentinel and Log Analytics Workspace.

Sentinel & LAW (Log Analytics Workspace) Setup

In order to map attacks from the SQL Server and see where they are coming from (geographical location) we need to ingest IP Geo-data. IP Geo-data refers to information about the geographical location of an IP address.  This info can also include Country, City, ISP, Time zone, Domain name, Proxy or VPN, Latitude and Longitude.  Essentially, the LAW (log analytics workspace) will read the read IP addresses of the attacks against the SQL server (as seen in event viewer) and use the IP Geodatabase to map it in Sentinel so we can see the physical location of the IP address.  

First you must download an IP-Geodatabase, you find them anywhere online. For the sake of this tutorial, I will be using a database provided to me from my course (I don’t know where they came from, so don’t ask me.)
 

Next, we must create a storage account in Azure to store our IP Geo-database, make sure to put it in your cyberlab resource group and keep it in the same region as your resource group. Name your storage account whatever you want.
 
Now we will create a container named “IP-Geodatabase” inside of the storage account to store our Databases. Inside of the storage, click “containers” then “+Container” then create the container. The purpose of creating this container is so when we ingest our geodata, we have point of reference to ingest from.
 

Next upload your IP-Geodatabases that you downloaded into the container.
 
 
 
 
Next, we will generate a SAS (shared access signature) URL. Essentially what this does is grant access, for a specific time, to the contents inside the storage blob (container) without you having to share your storage account credentials.  Think of it as a smart keypad lock; you have the master code (your account information) to unlock the door of the house (container). But you’ve also programed a secondary code (SAS URL) because you’re expecting the maintenance technician to come to your house and fix something while you’re at work. So, you give the technician the temporary code to access the house, but his code is only good for today, after today the code will no longer work which means no more access to the home. We are going to need the SAS to grant permission to the LAW (log analytics workspace) so that it can use IP Geo-database in the storage blob. 

Right click, set expiration, generate SAS
 

Copy SAS URL
 

Do the same for “City Locations” or whatever the name of your GEO-Data is
 

I suggest pasting those URLs somewhere safe because we will use them for our Log Analytics Workspace, which we will create now.  Go back to portal.azure.com and search for “log analytics workspace “and create a LAW. (You should know what LAW means by now)
 



Make sure you put it in your cyberlab resource group, make it the same region as that resource group, and name it whatever you like. Then review and create.
 


Now we will create the Sentinel, which is used to map the attacks. Once that is created, we will link it to our LAW, create 2 watchlist in the Sentinel and ingest the geo-data from our container in storage into LAW. The SQL server will audit the attacks against it, record those attacks in the windows security log along with the IP addresses, LAW will read those IP addresses and pinpoint the geographical location of those IP address using the geo-data ingested from the container using our SAS URLs, and Sentinel will put those locations on a map and create a table of those malicious IP addresses using the watchlist’s, so that if they are detected in network traffic, Sentinel will trigger a security response or investigation.

Create Microsoft Sentinel
 
Connect Sentinel to LAW
 


Create the Watchlist in Sentinel 
 


Create the Watchlist based off the Geo Data in the container.
 


Source type is Azure Storage, paste “city blocks” SAS URL in the “BLOB SAS URL (PREVIEW)” field and the search key is “NETWORK”. You know it’s right when the file preview populates on the right. “Review and create “and then create on the next page.
 

Do the same for “City Locations”. The name will be different; name it “geo_ipv4_cities, paste the “City location” SAS URL blob, and the search key will be “geoname¬_id.” After that, create the watchlist
  


It will take at least 3 days for the watchlist “city blocks” to upload. Do not move forward until both say succeeded.
 
 
 
 



After the Geo-Ip database is completely ingested, create a Linux-VM. We will be connecting the Linux machine syslog to the LAW to monitor anyone that fails or succeeds to SSH into our Linux machine. Make sure the VM is in the same resource group as your other resources and the region is the same as your resource group.
 
 

Logging to LAW (Log Analytics Workspace)
Now that all the geo-data is collected in the sentinel and your resources, it’s time to connect everything to the LAW so that it can monitor all the traffic coming through our NSG’s (Network Security Groups), our Virtual Machines, our Resources, etc. Think of the LAW as the brain in this SIEM. The sentinel would be the eyes; it helps us visually see the attacks on a map (hence is why we needed to ingest the Ip-geodata so it can make a map of Ip addresses correlated with locations) and you can’t see without a brain. Notice in the exercise that you cannot create a sentinel without first having a LAW to assign it too. Your Virtual Machines, NSG’, Resources, etc... can be thought of as limbs. If someone were to touch your arm or cut your arm, regardless of what’s going on, it’s still stimuli; information that your arm would send back to your brain to let you know what’s going on either positive or negative. Your LAW is the same way. If someone tries to sign into your server or VM or resource (whatever you are logging), the attempt will be logged into the LAW whether its positive or negative so that you know what’s going on in that environment. 

Let’s connect our Windows and Linux VMs to the LAW so it can log the activities from them, starting with the WindowsVM. We will also log the NSG (network security groups) associated with each VM to monitor the traffic coming in and out of our “firewall” in the cloud. First, we must enable “Microsoft Defender for Cloud” in the portal. These plans allow us to ingest logs from our virtual machines and SQL instances into our LAW. It’s kind of like enabling the logging feature in our LAW like how we did in our SQL server when we ran that script in command prompt and added Network Services to security in the registry.
Follow the screenshots below:
 
 

We will start with enabling all the plans for the Azure subscription. Click the three dots and click “edit Settings” and that will take you to the settings were we can enable the plans.
 

Enable all the settings, making sure that the status of everything is switched to “on” and then “save”. Once we are done, we go to the “Continuous Export” tab. It is here we will tell the defender to send everything logged from our virtual machines to our LAW. Click save once you enable the export and select the export data types.
 




Now we will manually add NIST SP 800-53 R4 and Azure CIS 1.4.0 to our security policies. This provides a set of guidelines or rules that Microsoft can use to ensure that Cloud Defender is doing its job at properly keeping data safe.
 
 

Now we will go back to Microsoft Defender for Cloud environment settings and enable both plans for the “LAWCyberlab”. We will also go to “Data Collection” and select “All Events.” Remember to save once making all your selections. 
 
 

LOGGING NSG (Network Security Group)
The NSG is the “firewall” of the cloud. It controls/monitors what traffic comes in and out of our virtual machines. We are going to log this information in our LAW through “Diagnostic Settings” “NSG Flow Log” and “Data Connectors.”
 First, go to your virtual machine and go to “Networking.”
  

Next, we will click on the NSG associated with that network.
 

From here we will add a “Diagnostic Setting.” This tells the NSG to send the logs to our LAW from that virtual machine.
 

Name your diagnostic setting to whatever name you choose and send all logs to the LAW “DON’T FORGET TO SAVE!!!!!!!!!” 

Next, we will create a “NSG Flow Log” which sends all the logged NSG traffic to the LAW to eventually be monitored on the Sentinel (the eyes).
 

Next create a Flow Log and follow the screenshots below. (Select your WindowsVM-nsg as your resource)
 
 

“Review+create” your Flow log. Repeat these same steps for your Linux Machine

Next step is to finally connect the Windows VM Directly to our LAW using “Data Connectors” Think of the data connector as a nerve in the analogy I gave about the SIEM being compared to the human body earlier in this tutorial. Any stimuli (Successful or failed login activity in the VM) will be reported back to the brain (the LAW). We will go to Microsoft Sentinel and then “Data Connector.”
 



In the search box we will type “Windows Security Events via AMA (Azure Monitor Agent)” and open connecter page. This connector provides security-related activities on “Windows Machines” in azure, which is very important because we use a different method to log our Linux machine.
 


Next we will create a data collection rule, follow the steps below 

 
 
 
Review Create, and we will connect our Linux Machine


In order to log the Linux VM, we will connect the data connector through our LAW and click the “agents” tab, follow the screenshots below to connect the data connector for linux.
 


 

 

 


In this step, make sure everything says “none” except the LOG_AUTH
 


 

After a few minutes, you should see that there is 1 server found for Linux 
 

You have now successfully logged both VMs to LAW!


AZURE AD LOGGING 
Azure AD (Active Directory) essentially contains the log in history of the tenant (your account) and any changes made such as creating users, assigning a user to a group, assigning a role to a user, and deleting a user. There are instances where people will try to hack into your account, so we log the Active Directory events in the LAW so it can eventually be mapped by sentinel. 

First, we will enable audit logging through “Diagnostic Settings” (connecting to LAW)
  
 

Name your setting whatever you’d like to name it and make sure to send the logs to your LAW and be sure to highlight in the “Categories “section, the first two options.
 

Once completed, click save and when you refresh the page; you should see your setting. 
 


Now that we have our Diagnostic Setting (our nerve from the limb connecting to the brain (LAW)), let’s create a new user in Active Directory, assign a role, and check the LAW to see if this process gets logged.
 

 
You can choose to have a password autogenerated or you can uncheck the box and choose to create a unique password for the user.
 
Assign Global role, which is total control (something you shouldn’t do essentially)
 
Review+Create and when you refresh, you should see your new user 

In LAW you can see that creating a new user and assigning a role generated log.
 



CREATING SIEM MAP
We are finally here! We are now at the stage where we create our world attack map and can visually see where attacks are coming from. We will create this map in Sentinel in the Azure portal. We will build 4 maps to map attacks coming from our Windows VM, Linux VM (Syslog), Azure SQL Server, and our Network Security Groups (cloud firewall).

First, we go to Sentinel and then select “Workbooks” and then “My Workbooks
 

Now Select “Add workbook” and click “edit” at the top left. Click the 3 dots and delete the prebuilt elements on the page
 
Once done; it should be blank.
 

We will now create our first map, starting with the Linux (Syslog) failed authentication. Click “Add” and “Add query”
 
Then click "Advanced Editor” Delete the query that is prewritten inside. 
 

Copy and paste the corresponding query and click “Done Editing.” MAKE SURE TO CLICK THE ONE AT THE BOTTOM!!!
 

Your map should generate and look like this 
 

These are all the failed attempts to ssh inside of our Linux machine and where they come from. Next, we will click “Done Editing” again at the top and save our workbook. Title the workbook after what is being logged and reference the correct resource group and apply.
We will repeat these steps for all 4 maps.
 

After creating the 4 maps, you should have 4 workbooks.
 


INCIDENT/ALERTS
Now we will set up alerts for our attacks. We will also generate some attacks on our environments so that we will see how these alerts respond to the incidents.

We will open the sentinel and go to Analytics.
 

Next, we will create our first analytic alert rule for our Windows VM

 





 

 

Next, we input the query that the alert rule will use. The query is as follows.
// Brute Force Attempt
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(60m)
| summarize FailureCount = count() by SourceIP = IpAddress, EventID, Activity
| where FailureCount >= 10
 

After this step, the alert enrichment we will use is “Entity mapping.” Essentially what this does is correlate the attacks by the identifiers you selected. For example, if an alert gets triggered by an attacker with an IP address of 1.1.1.1 and then another attack goes off with same corresponding IP address, sentinel can correlate that this is the same attacker, linking similar entities with different data sources.
 


With query scheduling, we will run this query every 10 minutes and lookup queries from the last 24 hours to stay current and up to date on attacks.
 

For incident settings, we are going to keep “create incidents” enabled so each event will create an alert, and we will enable alert grouping so that we group related attacks together.
 


After that, review and create.
 

You will see in analytics our rule is created and if you go to the “incidents” section, you can see the incidents triggered by this rule.
 


 

By clicking on the incident, you can see how many events occurred.
 

After 24 hours
 

By scrolling down and clicking on the “Actions” tab and selecting “Investigate” you can see a mini map of the attackers and IP addresses involved in the incident
 
 


SECURING THE CLOUD

So now let’s recap, we created our insecure virtual machines, enabled logging, created our LAW (the brain), created our Sentinel (the eyes), connected our virtual machines to the LAW (data connectors), connected our NSGs to the LAW (NSG Flow logs and Diagnostic Settings), ingested our geodata, created our workbooks (maps) to view the attacks, created alerts using query to alert us of an attack, we will now secure our environments using NIST compliance.

Microsoft Defender for Cloud is what you will use to check the level of how secure your environment is and how to improve upon that security level. For example, my environment is at 60%.
 

By clicking “Regulatory Compliance” it will show me what parts of my environment are not secure by having a red x next to the compliance control.
 

By clicking on one of the controls, it will expand that control and show you in detail what parts of the control are out of compliance and how to get that control in compliance.
 

We build up SC-7 Boundary Protection to NIST-800 53 standards to mitigate attacks and malicious traffic to our VMs.
 
 

We will start by creating private endpoints on our network. Endpoints provide a secure, direct connection for accessing our services in Azure without public exposure. Think of it as a secret road to a special place; a special road being a special IP address and the special place being our virtual machines, storage account, and any other service in our environment.

Let’s start with creating an endpoint for our storage accounts where the blobs are stored.
Go to your storage account and choose “Configuration” to remove our blob from any internet access.
 


 After that we will go to “Networking.” From here we will disable any public access so that our resources cannot be reached from the open internet; then we will create private endpoints.
 

 


Make sure everything is in the correct resource group and name the endpoint.  


We choose “Blob” as the target source.
 


Choose the correct Vnet 

  

Create then done! 

Now the only way to access our storage account is through our portal or through the endpoint from the created blob service link. 


Last thing we need to lockdown are the NSG’s to stop the flow malicious attacks on our virutal machines. Lets go to our windows machine and select “Networking.” Then we will add another inbound rule. In the picture I already have it done so ignore the top rule, we will be creating yours.
 

Select “My IP address” as the source to allow traffic from our  IP address only.  Anything else will not be allowed to enter our network. 


Name your rule and set your priority at the highest so this rule is first and add. Mine is already created but it will allow you to add on your profile
 

Repeat the same process for the linux machine.
 

We will create one last NSG and attach it to our subnet, adding an extra layer of protection and filtering network traffic at the subnet level. I’ve created one already but still follow the screenshots to create your own.
 
 

 


Now open your resource group and create an inbound rule allowing  traffic from only our IP address just like the previous NSG’s we configured for our virtual machines.
 


 

 

Now we go to “Virtual Networks” in Azure and attach the created NSG to our subnet. Follow the screenshots. (I created another NSG before these screenshots so mine will be under a different name).
Select your Vnet  


 Go to “Subnets”, select the default subnet and add your created NSG to the  subnet and save.  

Let your virtual machines run for 24 hours and check your maps in sentinal and you will notice how the attacks are drastically dropped, if there are any to log at all. These are before and after photos after securing the VM’s. (Edit the query in the Map to 24 hours to see results)

 
 


 
 


 
 
After securing my enviornment, I have no attacks. 
 

Compliance raised to 83%
 


We have now offically completed our SIEM project!!!






































