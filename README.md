
## Download links for the files used in this lab:

VirtualBox
https://www.virtualbox.org/wiki/Downloads

Windows Server
https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022
 
Windows 10
https://www.microsoft.com/en-us/software-download/windows10

  

## Prepare the files:

Download and install VirtualBox first, then download and install VirtualBox extension Pack

Download and run the Windows 10 media installer, select the option to download the Windows 10 ISO, then save the ISO to a folder of your choice 

Download Windows Server 2019 ISO directly from the link above and save it to a folder of your choice

## Domain Controller

**Create the VM (virtual machine) for the Domain Controller**

Open VirtualBox and select New to create a new VM

Give the VM a name like DC 

Select the Windows 2019 server ISO

Make sure to check "Skip unattended install"

Under hardware you can select how much RAM and how many CPU cores you want to give the VM.  I have 8 cores / 16 threads and 32 gigs of RAM on my host machine so I gave my VM 4096mb RAM and 2 CPU cores.  The more RAM and cores you can give the VM the faster it will be.

Hit Finish

Under Settings go to expert mode

Go to network, select Adapter 1, make sure it is enabled and that it is attached to NAT network

Select Adapter 2, enable it, ensure it is attached to Internal network

Hit Start on the DC virtual machine

**Install Windows on the Domain Controller**

The DC virtual machine should boot up into the Windows Server installation

Install Windows Server 2019

Choose clean install

When choosing which disk to install to just select the unallocated space and hit next

Once you get through the installation process, give the initial account a password. For lab purposes make it easy like Password1

When you sign on you need to use the input menu at the top of the VM window to select CTRL-ALT-DELETE.  If you ever find your mouse gets imprisoned within the VM window you can press the right CTRL button on your keyboard to free it.

You will login as a local Administrator with Password1

Once logged in I would select "yes" to allow this PC to be discoverable by other PC’s and devices on this network.  This actually won't work at this point since Server 2019 has some services turned off by default.  We'll turn these services on later.

Install VM guest additions. This will improve performance while working with the VM's.  Go to devices at the top of the VM window and select Insert Guest Additions CD Image.

Go to file explorer (the folder icon at the bottom of the taskbar) and go to This PC and double click Virtual Box guest additions CD. Install the one ending in AMD64.

Restart Windows and login as Administrator again

**Configure the Network Adapters of the Domain Controller**

Login

Right click on the network icon in the lower right of the Windows Server

Change adapter settings

Right click on one of the adapters and click Status and then Details

The one that has an IP starting with 10 is your Internet adapter. Right click and rename the adapter to Internet.

The one that has an IP starting with 169 is your Internal adapter. Rename it to Internal. This internal adapter will connect to the internal adapters of the other VM's/computers later on in our lab.  We will need to give it a static IP and DNS address.

Right click properties select IPV4

Use the following IP:172.16.0.1

Subnet mask 255.255.255.0

Use the following DNS:

Preferred DNS: 172.16.0.1 (you can also use 127.0.0.1 but I have better luck using the same IP)

Alternate DNS: you can leave this blank

**Give the Domain Controller a name**

Even though the VM is named DC, its "Windows" name is still default.  We should rename it to DC.

Right click on the Start menu and select System and select Rename this PC and name it DC

Restart Windows Server and login again

**Install Domain Services**

At the Server Manager dashboard hit Add Roles and Features

Hit next on the Before you Begin screen

Check the Role-based or feature-based installation and hit Next

Check select a server from the server pool; it should be just the DC server

Install Active Directory Domain Services

Hit next until you get to Install and then Finish

Go up to the yellow flag on upper right of the Server Manager dashboard

Promote this server to a domain controller

Add new forest

Name the domain mydomain.com for lab purposes

Use the password you created earlier Password1

Leave DNS delegation unchecked

Let the NetBios domain name autofill to MYDOMAIN

Keep hitting Next until you get to Install

Restart Windows Server

**Create domain Admin account**

Login to Windows Server

You will notice the sign on name is MYDOMAIN\Administrator

Login with Password1

Hit the Start button

Go to Windows Administrative Tools

Go to Active Directory users and Computers

Right click on mydomain and create New Organizational Unit and name it _ADMINS

Once _ADMINS is created right click on it and create new user

Enter first name, last name and logon name. The typical naming convention for a logon name would be first initial of your first name and full last name. So for John Doe it would be jdoe. For an admin logon put an a- in front of it so it would be a-jdoe. My name is Angel Toquero so I use a-atoquero

Give the user an easy password for lab purposes, like Password1.

For lab purposes, Uncheck user must change password at logon

For lab purposes, Check password never expires

Once created, select the _ADMINS folder and right click the user you just made and select Properties

Select MemberOf, select Add. Where it says enter the object names to select type in Domain Admins and click Check Names then press OK

You now have a domain admin account created

Logout and login

At the sign on screen you can Select other user

Type in the domain admin logon name and password you just created

**Install NAT on the Domain Controller**

At the Server Manager dashboard hit Add Roles and Features

Hit next on the Before you Begin screen

Check the Role-based or feature-based installation and hit Next

Check select a server from the server pool; it should be just the DC server

Install Remote Access

Install Routing

Finish

In Windows Server manager, go to tools in the upper right corner, select Routing and Remote access

Right click DC and configure and enable

Select NAT and hit next

Under use the public interface to connect to the Internet you should see your internal and internet adapters

If you don’t see your internal and internet adapters hit cancel, close Routing and Remote access and open it again from the Tools menu. Right click DC, hit configure and enable and see if the network adapters pop up.

Once they pop up select the Internet adapter. 

Hit next and finish.

**Install DHCP server on the Domain Controller**

At the Server Manager dashboard hit Add Roles and Features

Hit next on the Before you Begin screen

Check the Role-based or feature-based installation and hit Next

Check select a server from the server pool; it should be just the DC server

Select DHCP, hit next and finish

At Server Manager dashboard go up to tools and select DHCP

Under dc.mydomain.com 

Right click IPv4 and select New scope

Name the scope 172.16.0.100-200

Hit next

For IP range:

Start IP address 172.16.0.100

End IP address 172.16.0.200

Length 24, subnet mask 255.255.255.0

Hit next

For add Exclusions and Delay leave it all blank and hit next

For lease duration I just leave it at 8 hours

Hit next

For configure DHCP options select Yes, I want to configure these options now

Hit next

The IP of our internal network adapter will be our gateway IP, so input 172.16.0.1 and hit add

Hit next

For Domain name and DNS servers:

parent domain is mydomain.com

The IP of our internal network adapter will be the IP of our DNS server

Enter 172.16.0.1 and hit add 

I remove the other IP addresses to be sure 172.16.0.1 is functioning as a valid IP for our DNS server

Leave WINS Servers blank and hit next

Activate scope now next and finish

Click on IPv4 to expand it.  See where it says server options.  Right click server options, select configure options, select Router, input 172.16.0.1, hit add and hit OK. Select DNS servers and input 172.16.0.1, hit add and hit OK.

Right click dc.mydomain.com and hit authorize

Right click dc.mydomain.com and hit Refresh and IPv4 should turn green

## Client 1

**Create the VM for Client 1**

Open VirtualBox and select New to create new VM

Give it a name like CL1 

Select the Windows 10 ISO

Skip unattended install

Under hardware give it 4096mb RAM and 2 CPU cores

Hit Finish

Under settings go to expert mode

Select Adapter 1, enable it, ensure it is attached to Internal network

**Install Windows 10 on CL1**

Hit Start on the CL1 virtual machine

Install Windows 10

Select I don’t have a product key

Select Windows 10 Pro

Hit next on unallocated space

For the next few screens I would select “I don’t have internet” and “Limited Setup”

Give the initial account a username and password.  For lab purposes type in “User” for the login and leave password blank.

I would opt out of all the options for the next few screens until setup is complete.

Once setup is complete, Install VM guest additions.  Go to devices at the top of the VM window and select Insert Guest Additions CD Image.

Go to file explorer, This PC, and double click Virtual Box guest additions CD.  Install the one ending in AMD64.

Restart Windows 10

**Change Windows name to CL1 and join domain**

Login as User

Right click Start Menu

Go to System

Rename this PC advanced

Change name

Rename it to CL1

Domain is mydomain.com

Use the domain admin account credentials (the user you created in the _ADMIN folder on the DC) to join mydomain.com

Restart Windows 10

Login as domain admin

Go to Network and Sharing center, turn discovery on and allow this computer to see and be seen by other computers on the domain


## Client 2

**Create the VM for Client 2**

Open VirtualBox and select New to create new VM

Give it a name like CL2

Select the Windows 10 ISO

Skip unattended install

Under hardware give it 4096mb RAM and 2 CPU cores

Hit Finish

Under settings go to expert mode

Select Adapter 1, enable it, ensure it is attached to Internal network

**Install Windows 10 on CL2**

Hit Start on the CL2 virtual machine

Install Windows 10

Select I don’t have a product key

Select Windows 10 Pro

Hit next on unallocated space

For the next few screens I would select “I don’t have internet” and “Limited Setup”

Give the initial account a username and password.  For lab purposes type in “User” for the login and leave password blank.

I would opt out of all the options for the next few screens until setup is complete.

Once setup is complete, Install VM guest additions.  Go to devices at the top of the VM window and select Insert Guest Additions CD Image.

Go to file explorer, This PC, and double click Virtual Box guest additions CD.  Install the one ending in AMD64.

Restart Windows 10

**Change Windows name to CL2 and join domain**

Login as User

Right click Start Menu

Go to System

Rename this PC advanced

Change name

Rename it to CL1

Domain is mydomain.com

Use the domain admin account credentials (the user you created in the _ADMIN folder on the DC) to join mydomain.com

Restart Windows 10

Login as domain admin

Go to Network and Sharing center, turn discovery on and allow this computer to see and be seen by other computers on the domain


## File Server

**Create VM for File Server**

Open VirtualBox and select New

Give it a name like FS1 

Select the Windows 2019 server iso

Skip unattended install

Under hardware give it 4096mb RAM and 2 CPU cores

Hit Finish

Under settings go to expert mode

Go to Network

Select Adapter 1, enable it, ensure it is attached to Internal network

**Install Windows Server on FS1**

Hit Start on the FS1 virtual machine

Install Windows Server 2019 Standard Edition (desktop edition)

Give the initial account a password. For lab purposes make it easy like “Password1”

Install VM guest additions. Go to devices at the top of the VM window and select Insert Guest Additions CD Image.

Go to file explorer, go to This PC, open the Virtual Box guest additions CD. Install the one ending in AMD64.

Restart Windows Server

**Change Windows name to FS1 and join domain**

Log into Windows Server

Right click Start Menu

Hit System

Hit System Info

Under computer name, domain and workgroup settings select change settings

Click change

Rename it to FS1

Domain is mydomain.com

Use the domain admin account for credentials

Shut down Windows server

**Add a second hard drive to FS1 VM**

Back to Virtualbox Manager 

FS1 should be powered off.

Select FSI and go to Settings.

Go to Storage

Next to Controller: SATA there is an icon to add a hard disk.  Click it.

At the Hard Disk selector screen hit Create

Use the default name.  Mine is called FS1_1

For size 10 gigs will be enough for the lab.

Hit finish 

In Hard Disk selector you will see it is listed under Not Attached.  

Just double click on the name and it will attach itself.  

You'll see it listed under Controller: SATA

**Add the second hard drive to Windows**

Start up the FS1 virtual machine again

Log into Windows Server with the domain account we created earlier

Right click on the start menu and hit Run

Type in diskmgmt.msc

The disk we just attached to the FS1 virtual machine will now initialize in Windows

Once initialized it will resemble a black bar

Right click on it and create a simple volume using all the default settings

Once finished you can go to file explorer and under This PC see that there is now a second hard drive.  Mine is setup as E: drive

**Create a network share in Windows**

For lab purposes we’re going to create a network share at the root of E: drive. 

Create a folder at the root of the E: Drive called SoftwareDeploy. 

Right click SoftwareDeploy

Hit properties

Go to the sharing tab

Click share

Type in Domain Computers and hit Add

Type in Domain Users and hit Add

Give both only Read only access.  

If there is a listing for "Everyone", remove it.  

Hit Share at the bottom.

Go to the Security tab

Edit permissions 

Add Domain Users 

Add Domain computers

Give both read access only 

Remove "Everyone" if that is listed

Hit OK and finish

**Download Firefox and Chrome**

At the server manager dashboard hit Configure this local server and turn off IE Enhanced Security Configuration

Open Internet Explorer
Go to https://www.mozilla.org/en-US/firefox/all/desktop-release/](https://www.mozilla.org/en-US/firefox/all/desktop-release/

Download the Windows 64-bit MSI of Firefox and place it in the SoftwareDeploy folder at the root of the E: drive. 

Go to https://chromeenterprise.google/download/

Download the Windows 64-bit MSI of Chrome and place it in the SoftwareDeploy folder at the root of the E: drive.  

Make sure to get the MSI versions of the installation files because we will need that version to deploy it later in the lab.

**Turn on services to make FS1 visible to domain computers**

By default Windows server has services disabled or off that prevents network discovery

Right click on start

Click Run

Type in Services.msc

Right click the following services and set to automatic and right click to start them:

Function Discovery Provider Host

Function Discovery Resource Publication

SSDP Discovery

UPnP Device Host

## Group Policy Edit

**Create new Organizational Unit**

An organizational unit is a way to put computers or users into specific groups

You can then apply a group policy to these organizational units that would affect only the users or computers inside the group

Login into DC

At the Server Manager dashboard go to Tools

Open Active Directory Users and Computers

Right click mydomain.com

Click New Organizational Unit and call it _mydomain_computers

Right click _mydomain_computers and create another organizational unit and call it Chrome

Right click _mydomain_computers and create another organizational unit and call it Firefox

Under mydomain.com you will see a built-in organizational unit called Computers

Inside Computers you will see all the computers connected to the domain such as FS1, CL1 and CL2

Drag CL1 to Chrome

Drag Cl2 to Firefox

**Create Group Policy to install Chrome to the computers located inside the Chrome organizational unit**

At the Server Manager dashboard go to Tools

Open Group Policy Management

Under mydomain.com open _mydomain_computers

Right Click Chrome

Click Create a GPO in this domain and link it here

Give the group policy object a new like Chrome_deploy

Right Click Chrome_deploy and click Edit

This should open the Group Policy Management Editor for Chrome_deploy

Under Chrome_deploy open Computer Configuration > Policies > Software Settings > Software Installation

Right click software installation and select new > Package

Go to the network 

Type in //FS1/SoftwareDeploy into the address bar or click on FS1 and get into SoftwareDeploy

Choose the Chrome msi 

Choose Assigned and hit OK

**Create Group Policy to install Firefox to the computers located inside the Firefox organizational unit**

At the Server Manager dashboard go to Tools

Open Group Policy Management

Under mydomain.com open _mydomain_computers

Right Click Firefox 

Click Create a GPO in this domain and link it here

Give the group policy object a new like Firefox _deploy

Right Click Firefox _deploy and click Edit

This should open the Group Policy Management Editor for Firefox _deploy

Under Firefox _deploy open Computer Configuration > Policies > Software Settings > Software Installation

Right click software installation and select new > Package

Go to the network 

Type in //FS1/SoftwareDeploy into the address bar or click on FS1 and get into SoftwareDeploy

Choose the Firefox  msi 

Choose Assigned and hit OK

**Test Group Policy on CL1**

CL1 should be in the Chrome organizational unit, which in turn has the Chrome_deploy group policy object

Login to CL1 with domain account

Right click Start menu and hit Run

Type in CMD and hit OK

Type in gpupdate /force

Restart Windows

Login to CL1 again with domain account

Cl1 should have Chrome installed

**Test Group Policy on CL2**

CL2 should be in the Firefox organizational unit, which in turn has the Firefox_deploy group policy object

Login to CL2 with domain account

Right click Start menu and hit Run

Type in CMD and hit OK

Type in gpupdate /force

Restart Windows

Login to CL2 again with domain account

CL2 should have Firefox installed