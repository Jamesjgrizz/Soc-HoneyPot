# Soc-HoneyPot
This is my SIEM project created and hosted in Azure

 James Reeves SIEM (Security Information and Event Management) in Microsoft Azure. 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/99dcf58c-f902-4607-bec4-0b43702200f2)


This exercise will show you how to set up a SIEM and map where the attacks are coming from, using Microsoft Sentinel and Log Analytics Workspace (LAW).

Virtual Machine Setup/ SQL Server Setup

 Create an Azure account/subscription.
 Create a resource group and create a virtual machine to go into that resource group; be sure to make sure both are in the same region.
   Create 2 windows virtual machines: one "WindowsVM" and the other "AttackVM". The windowsVM will be the main VM we store our SQL server on and view event manager. We will use attackVM to attack windowsVM so we can monitor the attacks on event viewer. The attackVM will be in its own resource group for this exercise and make sure the resource group and attackVM are in a different region than the windowsVM to simulate an outside attack from another part of the world. 

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/d65f53ad-b901-4917-973d-ad95a16ac1bd)

 


       Notice the “AttackVM” and the “Cyberlab(windowsVM)” are in separate resource groups and the locations (regions) are different.

 Next, we will disable the NSG (Network Security Group) on WindowsVM so that it is accessible and easy to attack for outside threats. We will be viewing these attacks on “Event Viewer”.
  . First go to the Cyberlab (WindowsVM) resource group:

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/8dc5a339-3947-4fad-ad66-6241280debf8)

    


Next; Select “WindowsVM-nsg”, we will create an inbound rule to allow all traffic into our windowsVM, inbound rules dictate what comes into our VM and outbound dictates what goes out. NSG (network security groups) in general; are firewall-like security groups for network traffic in Azure. A network security group is like a superhero that protects a special computer place in the sky called Azure from bad guys who try to break in. Just like how superheroes keep their city safe, NSG’s keep the computer place safe.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/db735d57-14a7-4c4c-88f3-b3062e929424)




Next you will select “inbound security rules” under settings and then “Add” to create a new rule to allow all traffic into the windowsVM;

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/70a36af3-a2e2-49b2-b802-6abca210c8f9)
 



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
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/034888a2-2f44-44d0-9113-0ef86d69f50c)

 



![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/1afe066d-4720-40f4-a1cd-f0bc8cea5292)
 







You should now see your new rule.  Allowing all traffic is bad for obvious reasons, but for the sake of the lab, you will see how many attackers are anxious to hack into your vm, and we will be able to track the attacks and eventually map them.

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/d771ace2-2fb6-4e6e-aa90-5b2f001a5d5e)
 


4) Now we are ready to remote into our WindowsVM and disable the actual firewall in our virtual machine. To do this, you will need the IP address of your virtual Machine. 
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/285b18fc-a641-42b4-a4ef-be7fec9f82fb)


Copy the Public IP address and depending on whether you are using a windows pc or mac; that will determine how you remote into your vm. In this example, I am using windows. If you are using a mac you can use go to the app store and download “Windows Remote Desktop.” Start by opening the start menu and typing in “Remote Desktop”. When the application pops up, you will type the IP address in the “Computer” field. And click “connect”
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/ce4eadcb-9bdc-42cb-9f40-41e1eddbd93a)


Log into your WindowsVM with your username and password you created when creating your vm then click “ok”. The vm should load up like a normal PC coming on for the first time. 
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/6fc1d057-fb89-4d69-ba65-4288b58b748f)

 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/93ae44f6-e72d-47f8-9d59-fded03714bc6)


We will now disable the firewall in order to make our VM more enticing and susceptible to cyber-attacks. To disable the firewall in the vm; open the windows menu and type in wf.msc 
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/afdc56af-0418-47f0-9a55-ee7e814b60b4)


Next click “Windows Defender Firewall Properties”
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/c7cac1ec-0980-4ca8-bd1f-55dc0f98a850)


Turn off the firewall for the Domain Profile, Private Profile, and Public Profile, click “apply” and “ok” after all three are turned off.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/4d3a9765-eefb-46ac-9699-678972de0a96)

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/3b2da200-add8-4958-bb61-9234c209d0ee)

Click “ok” and “Apply” to finalize shutting down the firewall 

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/e14664b7-c545-4597-9492-eaadb965498f)


Now we install SQL Server Evaluation edition on our WindowsVM so that we have something to monitor. Open edge and download from SQL Server 2022 | Microsoft Evaluation Center
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/ce7432a8-32b2-401a-b4a3-24e49756cbe7)
 

Download the EXE 64-bit download and run the download
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/b3cb407c-5e7c-4e50-a9dd-288b8b78407e)
 


Choose the media option 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/80654b6f-de4a-4b5f-a19d-f4626ec32b65)

Select ISO and then download
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/295e4f9d-f405-44bc-9a84-70533b238278)


Go to your download location and right click on the ISO and mount the iso to your computer.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/198638ea-6d56-4b4f-bddf-2f1466017310)


Now that the ISO is mounted as a drive on your computer, run the setup.
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/d82c6f83-d7b0-4d6f-b877-8f79b0676d7b)
 

Follow the screenshots below, these will guide you through the SQL server installation process:
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/69dc5337-c9ea-463d-a5fe-4acc6121ce4c)
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/b9f4ff19-7a22-4b6b-af6a-044a4dbe63d7)

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/c82eb888-5831-4646-8f28-e45e22b342a5)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/76dccaba-2bf0-4ba1-bc3b-0db19207bcb6)
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/27aabcf1-f85f-4f3a-8829-f6aad0bf9039)

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/e56e7b5d-83b4-4eb4-ab37-40da6ada117b)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/df762b10-32b9-47af-b369-ad882c5ee060)



Within the Database Engine Configuration; this is where you will create your username and password for your SQL server. By selecting “Mixed Mode” you enable the server to login with either windows or server credentials. The username by default will be “sa” so you can create whichever password you please.  Select “Add Current User” to add your current windows profile as the server admin. 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/2a2702c8-a282-4725-ab20-aea9997155f2)
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/c49685a6-28a9-4c99-80b7-57480650aa56)

Finally Done install, you now have a SQL SERVER!!! 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/6620a276-c1ac-40db-aee6-c719e4c19ad5)


Now that your server is downloaded, go back to the SQL Server Installation and install the SQL Server Management Tools
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/55ea60d0-c734-47c1-a51b-bd6f204f9de3)
 

This will open a link to Microsoft.com, download the SSMS 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/fad28b65-fbd8-41e6-96ad-38c221e8c2df)


Run the SSMS setup
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/da8e7609-e713-4123-afe2-96363ca7813c)
 

INSTALL 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/9776bc18-98f9-4a85-ab19-9ec78b061045)


Now that you installed SQL Server Management Studio, open the windows menu, type “SSMS” and open the SQL Server Management Studio
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/e97dc760-5784-445c-8d50-9660a82868fc)
 

Connect to your server with the username and password you created during the installation of the SQL Server.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/b907a90e-cc96-4005-a9e6-67b9d9a3995e)


Next, we need to grant permission and enable the SQL server to be able to write to the windows security log and audit any failed or successful logins into our server. Right click on your WindowsVM server and select “Server Properties” and “Security” then check to have both failed and successful logins logged in windows security log. 

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/ee1ebe09-e90c-4229-beb8-867297cc74c1)



We will now enable auditing from the SQL server by running a script in the command prompt, this way the SQL server will start tracking the activty going on in the server. You can find the script at  Write SQL Server Audit events to the Security log - SQL Server | Microsoft Learn and then copy the script below.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/63305450-2bd4-4a3d-afac-1aeda307d232)

Open the command prompt, run as an admin, and paste the script and run it. 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/484d3206-5ce3-44a9-b87f-96f8541e1f26)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/e13e6024-463a-4dae-8101-38c483145790)


The last step to getting the SQL server to audit is granting permission to the server to write to the windows security log in event viewer. To do this we will need to add “Network Services” to the security folder in the registry and grant full access. Follow the screenshots below
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/a7cbb2e2-11e0-45b1-a7e3-df30179e3ee6)

Open windows menu and open “Registry Editor” 

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/72b6e615-0a4d-4bc9-99f6-7549a513649f)



Open the folders that corresponds to the highlighted path until you get to “Security” Right click and select “Permissions” 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/3fe777a7-7c71-4e6b-8cb2-1263804a96f4)


Add “Network Services” to the “Group or Usernames” 

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/685bf0b8-ef6c-49b4-9474-b2c097d93464)







Allow “Full Control”, apply and ok. 
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/1a0b02bd-3684-44db-a266-e8fefc079655)


Restart server to finalize changes, SQL server can now audit events and log them into windows security log.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/fc985542-0173-4ade-ad3d-8d8f10f73a15)



Now let’s test the log by creating some failed attempts at getting into the server. Disconnect from the server and then enter a bad login when attempting to reconnect. Open the “Event Viewer” open the “Windows Log”, select “Security” and the failed attempt should be logged, proving that the server is logging failed and successful attempts.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/bc910416-eb53-4896-9b4e-399fce8c343d)
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f1679ba3-bba3-4f83-b0c8-e8d4a43a8f74)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/6d402898-95cf-4d88-9089-ee2f1aac63c9)
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/58dcd79d-eb37-47ae-8b5a-30672de7e520)


Now log into the AttackVM and download the SSMS (SQL Server Management Studio) just like we did on the WindowsVM. And we are going to generate some failed attempts and successful attempts against the WindowsVM Server.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/746db634-95e7-46c5-80d9-7898fdbf2f93)


We will open SQL Server Management tool and generate failed login attempts and a successful login. To login to WindowsVM server; copy the IP address of the WindowsVM and paste it into the “Server Name” field and then enter a bad password and username in order to generate a bad login attempt
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/18e19c39-afd3-4b25-ba66-64258075ec2e)

Now enter the correct username and password to generate a successful login.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/88f44b31-3d03-42d1-8f09-1ca2df30d117)


Now let’s  check the WindowsVM Event viewer and we should see both failed and successful logins from the Attack VM with the source coming from the Attack VM   IP address
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f190217c-322d-44e4-b944-5447abc2dd72)
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/2e557931-ad65-4bf7-8e61-a92cc285dd82)


Now that we know our SQL Server is auditing and successfully logging into windows security log, we can proceed to the next step, which is setting up Sentinel and Log Analytics Workspace.

Sentinel & LAW (Log Analytics Workspace) Setup

In order to map attacks from the SQL Server and see where they are coming from (geographical location) we need to ingest IP Geo-data. IP Geo-data refers to information about the geographical location of an IP address.  This info can also include Country, City, ISP, Time zone, Domain name, Proxy or VPN, Latitude and Longitude.  Essentially, the LAW (log analytics workspace) will read the read IP addresses of the attacks against the SQL server (as seen in event viewer) and use the IP Geodatabase to map it in Sentinel so we can see the physical location of the IP address.  

First you must download an IP-Geodatabase, you find them anywhere online. For the sake of this tutorial, I will be using a database provided to me from my course (I don’t know where they came from, so don’t ask me.)
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/d216375e-f0cc-4553-ab4c-e6968fd2110c)

Next, we must create a storage account in Azure to store our IP Geo-database, make sure to put it in your cyberlab resource group and keep it in the same region as your resource group. Name your storage account whatever you want.
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/26211c9e-15f5-4e88-b840-d3cd67d32329)
 
Now we will create a container named “IP-Geodatabase” inside of the storage account to store our Databases. Inside of the storage, click “containers” then “+Container” then create the container. The purpose of creating this container is so when we ingest our geodata, we have point of reference to ingest from.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/c3811304-dbfa-465c-8628-f35d0a8df809)

Next upload your IP-Geodatabases that you downloaded into the container.
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/9c3f5a96-eea3-4688-9706-c3672c870b5e)
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/b00fcc30-6d48-4e9b-a0bb-e313a8c17c20)

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/209a1422-f0b1-4f8c-bacc-979250fe10ae)

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/19a7023e-840c-49e9-9e31-05610407db42)

Next, we will generate a SAS (shared access signature) URL. Essentially what this does is grant access, for a specific time, to the contents inside the storage blob (container) without you having to share your storage account credentials.  Think of it as a smart keypad lock; you have the master code (your account information) to unlock the door of the house (container). But you’ve also programed a secondary code (SAS URL) because you’re expecting the maintenance technician to come to your house and fix something while you’re at work. So, you give the technician the temporary code to access the house, but his code is only good for today, after today the code will no longer work which means no more access to the home. We are going to need the SAS to grant permission to the LAW (log analytics workspace) so that it can use IP Geo-database in the storage blob. 

Right click, set expiration, generate SAS
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/0fa50e87-dd77-463a-b96e-41a3f465287a)
 

Copy SAS URL
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/d65ee6db-496d-4a70-bf88-222c834c617b)
 

Do the same for “City Locations” or whatever the name of your GEO-Data is
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/0d3950fd-23d2-4d7d-a888-e7769247b59e)
 

I suggest pasting those URLs somewhere safe because we will use them for our Log Analytics Workspace, which we will create now.  Go back to portal.azure.com and search for “log analytics workspace “and create a LAW. (You should know what LAW means by now)
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/fb196c07-cab6-4645-aff5-1da620df9356)



Make sure you put it in your cyberlab resource group, make it the same region as that resource group, and name it whatever you like. Then review and create.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/85ae600e-7f49-41d5-88a3-f3f0e2d320b8)


Now we will create the Sentinel, which is used to map the attacks. Once that is created, we will link it to our LAW, create 2 watchlist in the Sentinel and ingest the geo-data from our container in storage into LAW. The SQL server will audit the attacks against it, record those attacks in the windows security log along with the IP addresses, LAW will read those IP addresses and pinpoint the geographical location of those IP address using the geo-data ingested from the container using our SAS URLs, and Sentinel will put those locations on a map and create a table of those malicious IP addresses using the watchlist’s, so that if they are detected in network traffic, Sentinel will trigger a security response or investigation.

Create Microsoft Sentinel
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/ba83860a-9a62-4464-9949-f26ca217af33)

Connect Sentinel to LAW
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/d433b149-36b1-41bb-8269-aa1f137c7827)


Create the Watchlist in Sentinel 
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/0f43e47f-7d30-4190-b873-640d6cc2da7b)


Create the Watchlist based off the Geo Data in the container.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/5d481fee-e85d-4dd2-8d46-8191fd8f80a8)



Source type is Azure Storage, paste “city blocks” SAS URL in the “BLOB SAS URL (PREVIEW)” field and the search key is “NETWORK”. You know it’s right when the file preview populates on the right. “Review and create “and then create on the next page.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/7d5a65f8-4c53-4340-b1f5-3b0268ed1f20)

Do the same for “City Locations”. The name will be different; name it “geo_ipv4_cities, paste the “City location” SAS URL blob, and the search key will be “geoname¬_id.” After that, create the watchlist
  
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/cd5f710c-f7ff-492d-9239-f1c81a27f856)


It will take at least 3 days for the watchlist “city blocks” to upload. Do not move forward until both say succeeded.
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/236eb6a6-aae5-4daf-bd8f-ae1c05b5b134)

 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/dd650bf3-f7ae-4938-9e09-d836d7dfaaf1)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/9c5287c7-7800-43cb-89dd-ddde04ed0728)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/c72d5a67-5ebb-4d92-aa54-0a1d9589871e)


After the Geo-Ip database is completely ingested, create a Linux-VM. We will be connecting the Linux machine syslog to the LAW to monitor anyone that fails or succeeds to SSH into our Linux machine. Make sure the VM is in the same resource group as your other resources and the region is the same as your resource group.
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/e35ccff2-c78f-4154-adcd-42829ed74d96)


Logging to LAW (Log Analytics Workspace)
Now that all the geo-data is collected in the sentinel and your resources, it’s time to connect everything to the LAW so that it can monitor all the traffic coming through our NSG’s (Network Security Groups), our Virtual Machines, our Resources, etc. Think of the LAW as the brain in this SIEM. The sentinel would be the eyes; it helps us visually see the attacks on a map (hence is why we needed to ingest the Ip-geodata so it can make a map of Ip addresses correlated with locations) and you can’t see without a brain. Notice in the exercise that you cannot create a sentinel without first having a LAW to assign it too. Your Virtual Machines, NSG’, Resources, etc... can be thought of as limbs. If someone were to touch your arm or cut your arm, regardless of what’s going on, it’s still stimuli; information that your arm would send back to your brain to let you know what’s going on either positive or negative. Your LAW is the same way. If someone tries to sign into your server or VM or resource (whatever you are logging), the attempt will be logged into the LAW whether its positive or negative so that you know what’s going on in that environment. 

Let’s connect our Windows and Linux VMs to the LAW so it can log the activities from them, starting with the WindowsVM. We will also log the NSG (network security groups) associated with each VM to monitor the traffic coming in and out of our “firewall” in the cloud. First, we must enable “Microsoft Defender for Cloud” in the portal. These plans allow us to ingest logs from our virtual machines and SQL instances into our LAW. It’s kind of like enabling the logging feature in our LAW like how we did in our SQL server when we ran that script in command prompt and added Network Services to security in the registry.
Follow the screenshots below:
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/d2658461-c39c-4810-a60d-a829c3f852c9)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/481b1544-a8ae-44e8-be9e-413074a7d76f)

We will start with enabling all the plans for the Azure subscription. Click the three dots and click “edit Settings” and that will take you to the settings were we can enable the plans.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/a94ce65f-94ed-4cfb-a896-bc3013ae8864)

Enable all the settings, making sure that the status of everything is switched to “on” and then “save”. Once we are done, we go to the “Continuous Export” tab. It is here we will tell the defender to send everything logged from our virtual machines to our LAW. Click save once you enable the export and select the export data types.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/3cc87e02-6df3-42c1-9ad6-266e9d481532)




Now we will manually add NIST SP 800-53 R4 and Azure CIS 1.4.0 to our security policies. This provides a set of guidelines or rules that Microsoft can use to ensure that Cloud Defender is doing its job at properly keeping data safe.
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/03e82501-4308-4b4e-b9a9-49271be1b7e4)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/720349ad-2a50-42d7-8f61-afcfa7b8e620)

Now we will go back to Microsoft Defender for Cloud environment settings and enable both plans for the “LAWCyberlab”. We will also go to “Data Collection” and select “All Events.” Remember to save once making all your selections. 
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/06d2b73e-9fd5-4088-af75-91ab5f17af2c)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/3748cc55-7973-430f-8e7c-0549883490ab)

LOGGING NSG (Network Security Group)
The NSG is the “firewall” of the cloud. It controls/monitors what traffic comes in and out of our virtual machines. We are going to log this information in our LAW through “Diagnostic Settings” “NSG Flow Log” and “Data Connectors.”
 First, go to your virtual machine and go to “Networking.”
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/93071e02-88b4-4e79-8137-e166870ed177)
 

Next, we will click on the NSG associated with that network.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/e6222482-93f6-4a67-ab53-8b0d1b711e12)


From here we will add a “Diagnostic Setting.” This tells the NSG to send the logs to our LAW from that virtual machine.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/2a1ee963-3ef7-4471-a0c0-55e08caee98f)


Name your diagnostic setting to whatever name you choose and send all logs to the LAW “DON’T FORGET TO SAVE!!!!!!!!!” 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/38240e28-5885-4db2-adc8-7ef38ddce383)

Next, we will create a “NSG Flow Log” which sends all the logged NSG traffic to the LAW to eventually be monitored on the Sentinel (the eyes).
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/b773cc31-ef1c-4fa8-a045-14c7e1e46b90)

Next create a Flow Log and follow the screenshots below. (Select your WindowsVM-nsg as your resource)
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/7cfbd3f9-8ccc-49f4-a849-e957069ffee7)

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f1941c21-51fc-4399-90ba-e1ee7658147e)


“Review+create” your Flow log. Repeat these same steps for your Linux Machine

Next step is to finally connect the Windows VM Directly to our LAW using “Data Connectors” Think of the data connector as a nerve in the analogy I gave about the SIEM being compared to the human body earlier in this tutorial. Any stimuli (Successful or failed login activity in the VM) will be reported back to the brain (the LAW). We will go to Microsoft Sentinel and then “Data Connector.”
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/cc38bb3b-4d02-42d0-93b4-1441669959b8)



In the search box we will type “Windows Security Events via AMA (Azure Monitor Agent)” and open connecter page. This connector provides security-related activities on “Windows Machines” in azure, which is very important because we use a different method to log our Linux machine.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/e1f61be3-f37c-490b-b19c-7ec10c75ef23)


Next we will create a data collection rule, follow the steps below 

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/7fe53ceb-8b64-4bfc-9b2d-2974102ef204)

 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/a11727fa-d3dc-4222-b012-3e0dc315f02a)
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/92d4299f-23e8-47eb-a1f0-9d57298f8c51)
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/7f106377-6401-4f38-b951-83ab5637b6b5)

Review Create, and we will connect our Linux Machine


In order to log the Linux VM, we will connect the data connector through our LAW and click the “agents” tab, follow the screenshots below to connect the data connector for linux.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/4aac2df3-7486-4596-9c61-ecb238d9228b)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/85839689-543f-41a5-ae31-4acec2bb6173)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/a46aa71c-ce59-4f23-8c8a-7e6a34b7a1ad)

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f0390b8d-2b24-4c5a-a41f-b27f928222bb) 


In this step, make sure everything says “none” except the LOG_AUTH
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/38d6807f-8073-489a-bd59-64c297d0c943)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/4cf012f8-8890-4e32-89fa-b223c0595da3)

 

After a few minutes, you should see that there is 1 server found for Linux 
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/72e52387-ba45-4938-a928-4e48699c8c1a)

You have now successfully logged both VMs to LAW!


AZURE AD LOGGING 
Azure AD (Active Directory) essentially contains the log in history of the tenant (your account) and any changes made such as creating users, assigning a user to a group, assigning a role to a user, and deleting a user. There are instances where people will try to hack into your account, so we log the Active Directory events in the LAW so it can eventually be mapped by sentinel. 

First, we will enable audit logging through “Diagnostic Settings” (connecting to LAW)
  
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/2b3a2dec-4822-487d-a524-f52e2c0b889d)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/a20b7a07-7c99-4ac8-8ee4-bbc49c9076c0)

Name your setting whatever you’d like to name it and make sure to send the logs to your LAW and be sure to highlight in the “Categories “section, the first two options.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/b4660e5c-363e-4640-8c47-8324da746028)

Once completed, click save and when you refresh the page; you should see your setting. 
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/19ba8931-83a3-4af9-82eb-7ad0c88de0bc)


Now that we have our Diagnostic Setting (our nerve from the limb connecting to the brain (LAW)), let’s create a new user in Active Directory, assign a role, and check the LAW to see if this process gets logged.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/13d55cb8-8073-424e-aef6-858d0ecae00d)

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/1d7de1b5-f2bf-4386-a1ed-9c9de210cc41)

You can choose to have a password autogenerated or you can uncheck the box and choose to create a unique password for the user.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/8e45a84d-a61e-413a-bcf7-d627952f3e2f)

Assign Global role, which is total control (something you shouldn’t do essentially)
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/e31d6883-16d8-42ac-a4b3-b402dc7040ec)

Review+Create and when you refresh, you should see your new user 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/a977a2da-31e6-4448-b172-55f4396c9bdc)

In LAW you can see that creating a new user and assigning a role generated log.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/8f335afa-ce66-4fd1-8356-f82e250f4472)




CREATING SIEM MAP
We are finally here! We are now at the stage where we create our world attack map and can visually see where attacks are coming from. We will create this map in Sentinel in the Azure portal. We will build 4 maps to map attacks coming from our Windows VM, Linux VM (Syslog), Azure SQL Server, and our Network Security Groups (cloud firewall).

First, we go to Sentinel and then select “Workbooks” and then “My Workbooks
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/5ea7a708-203a-4674-bc4b-f7d8c1b16522)


Now Select “Add workbook” and click “edit” at the top left. Click the 3 dots and delete the prebuilt elements on the page
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/993d8ab1-4360-47f6-8385-63b4b02fc2dd)
 
Once done; it should be blank.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f19819ff-0ec7-4d66-9768-0de85ac7138b)


We will now create our first map, starting with the Linux (Syslog) failed authentication. Click “Add” and “Add query”
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/825d28ce-2e29-457f-8438-f3c8fa550053)

Then click "Advanced Editor” Delete the query that is prewritten inside. 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/6c55b7b0-fae5-4c18-968e-7e2c1944a8a5)


Copy and paste the corresponding query and click “Done Editing.” MAKE SURE TO CLICK THE ONE AT THE BOTTOM!!!
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/5247e5ed-e30c-4b77-ad13-82f7aa8ea082)
 

Your map should generate and look like this 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/43e18add-39d7-4d3a-b225-d6124234c088)


These are all the failed attempts to ssh inside of our Linux machine and where they come from. Next, we will click “Done Editing” again at the top and save our workbook. Title the workbook after what is being logged and reference the correct resource group and apply.
We will repeat these steps for all 4 maps.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/78ed96a8-420b-479e-aa9d-30d2cae57eee)


After creating the 4 maps, you should have 4 workbooks.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/e033b6e7-e105-42df-86d3-b64e28f90704)


INCIDENT/ALERTS
Now we will set up alerts for our attacks. We will also generate some attacks on our environments so that we will see how these alerts respond to the incidents.

We will open the sentinel and go to Analytics.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/2b458e31-9f8f-42a0-926d-9278b3cf1c00)


Next, we will create our first analytic alert rule for our Windows VM

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/3afa4856-eb53-4e05-90a7-3336026e918f)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/73325d14-0528-41c9-8ed6-a95f81e0e260)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/b5b13727-c8f5-4dc6-8942-158720aee9ef)




 

 

Next, we input the query that the alert rule will use. The query is as follows.
// Brute Force Attempt
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(60m)
| summarize FailureCount = count() by SourceIP = IpAddress, EventID, Activity
| where FailureCount >= 10
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/395e54e5-f54a-4c36-844b-4edc969906d5)

After this step, the alert enrichment we will use is “Entity mapping.” Essentially what this does is correlate the attacks by the identifiers you selected. For example, if an alert gets triggered by an attacker with an IP address of 1.1.1.1 and then another attack goes off with same corresponding IP address, sentinel can correlate that this is the same attacker, linking similar entities with different data sources.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/1f82d276-0563-4966-bc54-55216f791e71)



With query scheduling, we will run this query every 10 minutes and lookup queries from the last 24 hours to stay current and up to date on attacks.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f8d949c5-dd93-47a1-8c79-07df30f0fc11)

For incident settings, we are going to keep “create incidents” enabled so each event will create an alert, and we will enable alert grouping so that we group related attacks together.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/b2a9ea68-ab98-4bbd-8a8a-5c01213bb876)


After that, review and create.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/8d4c9ea7-a160-4b57-af86-df76e30e5e3f)


You will see in analytics our rule is created and if you go to the “incidents” section, you can see the incidents triggered by this rule.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/482cb55e-ea52-47a6-8916-fd3fcc71fc59)


 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/33e081e2-1e19-40f2-aa07-430e5b09da3f)


By clicking on the incident, you can see how many events occurred.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/a2660773-f6fb-4743-838c-0e6575098a1d)

After 24 hours
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/15593239-302d-4d59-ae27-dc52601321af)


By scrolling down and clicking on the “Actions” tab and selecting “Investigate” you can see a mini map of the attackers and IP addresses involved in the incident
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f1b77c22-7451-4a60-9053-c7d47f322d96)

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/879ef23a-a651-4962-ba2f-da4d559aa331)


SECURING THE CLOUD

So now let’s recap, we created our insecure virtual machines, enabled logging, created our LAW (the brain), created our Sentinel (the eyes), connected our virtual machines to the LAW (data connectors), connected our NSGs to the LAW (NSG Flow logs and Diagnostic Settings), ingested our geodata, created our workbooks (maps) to view the attacks, created alerts using query to alert us of an attack, we will now secure our environments using NIST compliance.

Microsoft Defender for Cloud is what you will use to check the level of how secure your environment is and how to improve upon that security level. For example, my environment is at 60%.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/7be2b656-3089-4d22-8a0e-4b5d2e3df951)


By clicking “Regulatory Compliance” it will show me what parts of my environment are not secure by having a red x next to the compliance control.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/71eca821-4787-4f94-9242-90782e12ba30)


By clicking on one of the controls, it will expand that control and show you in detail what parts of the control are out of compliance and how to get that control in compliance.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/97568c83-7a8e-401b-b9b6-5f303749ac2c)


We build up SC-7 Boundary Protection to NIST-800 53 standards to mitigate attacks and malicious traffic to our VMs.
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/be92d2a7-e4cd-4a8b-be54-7a750893d578)
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f708124a-fa7a-4558-8b33-a805f414d494)


We will start by creating private endpoints on our network. Endpoints provide a secure, direct connection for accessing our services in Azure without public exposure. Think of it as a secret road to a special place; a special road being a special IP address and the special place being our virtual machines, storage account, and any other service in our environment.

Let’s start with creating an endpoint for our storage accounts where the blobs are stored.
Go to your storage account and choose “Configuration” to remove our blob from any internet access.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/08b854d4-dc14-4680-ae78-39e17fcba69a)


 After that we will go to “Networking.” From here we will disable any public access so that our resources cannot be reached from the open internet; then we will create private endpoints.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f0cacf30-abfd-4cd3-bc15-9aba29d50258)

 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/13d64e82-8be8-4ca0-907e-b8afa31a7465)



Make sure everything is in the correct resource group and name the endpoint.  

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f181c0e2-905a-40da-8fd4-7f10f963b6bc)

We choose “Blob” as the target source.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/a066a4b5-4066-4265-84db-aa909c692984)


Choose the correct Vnet 

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/4372ba70-e4ed-4701-a11c-3339ff8c44d7)

  ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/90d9275f-68a5-49f5-a027-8bc9000610a9)


Create then done! 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/84faf68a-b7f7-4d25-b88c-e9432b6e5c08)

Now the only way to access our storage account is through our portal or through the endpoint from the created blob service link. 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/823bc9c6-fd58-498b-937d-267743abbed8)


Last thing we need to lockdown are the NSG’s to stop the flow malicious attacks on our virutal machines. Lets go to our windows machine and select “Networking.” Then we will add another inbound rule. In the picture I already have it done so ignore the top rule, we will be creating yours.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/ad1ddbb8-33a2-491b-8e38-265e3b060b3b)

Select “My IP address” as the source to allow traffic from our  IP address only.  Anything else will not be allowed to enter our network. 

![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/3c2fbda6-767a-4cc4-8249-8c9db658d897)

Name your rule and set your priority at the highest so this rule is first and add. Mine is already created but it will allow you to add on your profile
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/f37d491a-2340-4000-9213-726107ded344)


Repeat the same process for the linux machine.
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/0da29f47-6d24-4bd2-98b7-c70170bdc454)


We will create one last NSG and attach it to our subnet, adding an extra layer of protection and filtering network traffic at the subnet level. I’ve created one already but still follow the screenshots to create your own.
 
 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/24e6768d-6bd2-4dc7-89ad-5e63c895bfa9)


 ![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/2753570e-31e0-46d4-a181-743a1405d802)



Now open your resource group and create an inbound rule allowing  traffic from only our IP address just like the previous NSG’s we configured for our virtual machines.
 
![image](https://github.com/Jamesjgrizz/Soc-HoneyPot/assets/116228475/83229954-78b0-4efe-8918-5909a5fc1015)


 

 

Now we go to “Virtual Networks” in Azure and attach the created NSG to our subnet. Follow the screenshots. (I created another NSG before these screenshots so mine will be under a different name).
Select your Vnet  


 Go to “Subnets”, select the default subnet and add your created NSG to the  subnet and save.  

Let your virtual machines run for 24 hours and check your maps in sentinal and you will notice how the attacks are drastically dropped, if there are any to log at all. These are before and after photos after securing the VM’s. (Edit the query in the Map to 24 hours to see results)

 
 


 
 


 
 
After securing my enviornment, I have no attacks. 
 

Compliance raised to 83%
 


We have now offically completed our SIEM project!!!






































