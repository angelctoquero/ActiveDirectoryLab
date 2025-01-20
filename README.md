# Intro
This lab is inspired primarily from Josh Madakor's Active Directory Home Lab.

I'm gradually building on top of it to simulate an environment I might encounter in a corporate setting:

- Additional Windows clients (computers)
- A file server
- Organizational units to separate and group the clients
- Unique Group Policy Object for each organizational unit
- PowerShell commands to speed up routine tasks

I have some other ideas I will add as soon as I can.

# Handy Notes
For lab purposes
 - On the Windows Server domain controller I use Password1 when creating both the local admin and Domain Admin.
 - On the Windows 10 client I leave password blank on the local admin.  I name the local admin User in the lab.
 - Once I get Windows 10 clients renamed and connected to the domain I use the Domain Admin account and Password1 for the remainder of the lab.
 - If your mouse cursor gets stuck inside the VM you can press the right CTRL key to get out.
 - I renamed the ISOs on my host machine to Windows10 and WindowsServer so it might look different from what you download


# Contents
 - [Download links for the files used in this lab](#download-links-for-the-files-used-in-this-lab)
 - [Prepare the files](#prepare-the-files)
 - [Domain Controller](#domain-controller)
   - [Create the VM for the Domain Controller](#create-the-VM-for-the-Domain-Controller)
   - [Install Windows on the Domain Controller](#install-windows-on-the-domain-controller)
   - [Configure the Network Adapters of the Domain Controller](#configure-the-network-adapters-of-the-domain-controller)
   - [Give the Domain Controller a name](#give-the-domain-controller-a-name)
   - [Install Domain Services, DHCP Server, DNS Server, Remote Access](#install-domain-services-dhcp-server-dns-server-remote-access)
   - [Promote this server to a Domain Controller](#promote-this-server-to-a-domain-controller)
   - [Create domain Admin account](#create-domain-admin-account)
   - [Configure NAT on the Domain Controller](#configure-nat-on-the-domain-controller)
   - [Configure DHCP server on the Domain Controller](#configure-dhcp-server-on-the-domain-controller)
 - [Client 1](#client-1)
   - [Create the VM for Client 1](#create-the-vm-for-client-1)
   - [Install Windows 10 on CL1](#install-windows-10-on-cl1)
   - [Change Windows name to CL1 and join domain](#change-windows-name-to-cl1-and-join-domain)
 - [Client 2](#client-2)
   - [Create the VM for Client 2](#create-the-vm-for-client-2)
   - [Install Windows 10 on CL2](#install-windows-10-on-cl2)
   - [Change Windows name to CL2 and join domain](#change-windows-name-to-cl2-and-join-domain)
 - [File Server](#file-server)
   - [Create VM for File Server](#create-vm-for-file-server)
   - [Install Windows Server on FS1](#nstall-windows-server-on-fs1)
   - [Change Windows name to FS1 and join domain](#change-windows-name-to-fs1-and-join-domain)
   - [Add a second hard drive to FS1 VM](#add-a-second-hard-drive-to-fs1-vm)
   - [Add the second hard drive to Windows](#add-the-second-hard-drive-to-windows)
   - [Create a network share in Windows](#create-a-network-share-in-windows)
   - [Download Firefox and Chrome](#download-firefox-and-chrome)
 - [Group Policy Edit](#group-policy-edit)
   - [Create new Organizational Unit](#create-new-organizational-unit)
   - [Create Group Policy](#create-group-policy)
   - [Install Chrome to the computers located inside the Chrome organizational unit](#install-chrome-to-the-computers-located-inside-the-chrome-organizational-unit)
   - [Install Firefox to the computers located inside the Firefox organizational unit](#install-firefox-to-the-computers-located-inside-the-firefox-organizational-unit)
   - [Test Group Policy on CL1](#test-group-policy-on-cl1)
   - [Test Group Policy on CL2](#test-group-policy-on-cl2)
 - [Powershell](#powershell)
   - [New-ADOrganizationalUnit](#new-adorganizationalunit)
   - [New-AdUser](#new-aduser)
   - [Add-ADGroupmember](#new-adgroupmember)
   - [Move-ADObject](#move-adobject)

   

 
# Download links for the files used in this lab:

VirtualBox
https://www.virtualbox.org/wiki/Downloads

Windows Server
https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022
 
Windows 10
https://www.microsoft.com/en-us/software-download/windows10

  
# Prepare the files:

Download and install VirtualBox first, then download and install VirtualBox extension Pack
![Download_VM](https://github.com/user-attachments/assets/e714b563-51a6-4881-b499-c36d3377fd35)

Download and run the Windows 10 media installer, select the option to download the Windows 10 ISO, then save the ISO to a folder of your choice 

![Create_Windows10_ISO](https://github.com/user-attachments/assets/617ba2b8-d709-41b5-bace-8bcc82407bee)
![Create_Windows10_ISO (2)](https://github.com/user-attachments/assets/43e87fe9-9ebf-4f79-8359-baea894d160f)

Download Windows Server 2019 ISO directly from the link above and save it to a folder of your choice
![DownloadWindowsServer](https://github.com/user-attachments/assets/778d5dcb-30d4-4d45-8c2c-2aec4f52d23d)

## Domain Controller

### Create the VM for the Domain Controller

Open VirtualBox and select New to create a new VM
![VB_Create_VM](https://github.com/user-attachments/assets/f3ecd392-ff5d-496e-869a-6b4b57f052b5)

Give the VM a name like DC 

Select the Windows 2019 server ISO

Make sure to check "Skip unattended install"
![DC_VM_Setup_1](https://github.com/user-attachments/assets/cce49cb4-c4a7-4af9-b3d7-824af373b719)

Under hardware you can select how much RAM and how many CPU cores you want to give the VM.  I have 8 cores / 16 threads and 32 gigs of RAM on my host machine so I gave my VM 4096mb RAM and 2 CPU cores.  The more RAM and cores you can give the VM the faster it will be.

Hit Finish
![DC_VM_Setup2](https://github.com/user-attachments/assets/5ad6ccc7-3c39-4ca1-8ff4-d4ee6066a17f)

Under Settings go to expert mode
![DC_VM_Settings](https://github.com/user-attachments/assets/4e8ad6d7-76b3-400d-8305-6ed32916b04c)

Go to network, select Adapter 1, make sure it is enabled and that it is attached to NAT network
![DC_VM_Network1](https://github.com/user-attachments/assets/6d8a904e-ddb2-431e-9867-f33e9cf10d88)

Select Adapter 2, enable it, ensure it is attached to Internal network
![DC_VM_Network2](https://github.com/user-attachments/assets/77f2b4b9-afd4-4539-9656-b67ac3cbe5e9)

Hit Start on the DC virtual machine
![DC_Start_VM](https://github.com/user-attachments/assets/1b774521-b446-4223-be6a-ea1dad7be60e)

### Install Windows on the Domain Controller

The DC virtual machine should boot up into the Windows Server installation

Install Windows Server 2019

Choose Desktop Experience
![DC_Windows_Server_Install1](https://github.com/user-attachments/assets/bfa02ce0-23fd-4315-8862-d18610da73a3)

Choose clean install
![DC_Windows_Server_Install2](https://github.com/user-attachments/assets/ba056825-ea4e-41fe-99cc-96509a5e23b2)

When choosing which disk to install to just select the unallocated space and hit next
![DC_Windows_Server_Install4](https://github.com/user-attachments/assets/35c7bb96-090a-4268-80f2-078c0c2bbb26)

Once you get through the installation process, give the initial account a password. For lab purposes make it easy like Password1

When you sign on you need to use the input menu at the top of the VM window to select CTRL-ALT-DELETE.  If you ever find your mouse gets imprisoned within the VM window you can press the right CTRL button on your keyboard to free it.
![Windows_Logon_CTRL-ALT-Delete](https://github.com/user-attachments/assets/0a1e5512-706d-4985-84e0-fe159cd35c3e)

You will login as a local Administrator with Password1

Once logged in I would select "yes" to allow this PC to be discoverable by other PC’s and devices on this network.  This actually won't work at this point since Server 2019 has some services turned off by default.  We'll turn these services on later.

Install VM guest additions. This will improve performance while working with the VM's.  Go to devices at the top of the VM window and select Insert Guest Additions CD Image.
![Guest_Additions1](https://github.com/user-attachments/assets/6de5b3da-2610-408d-8a3a-95f841c76b97)

Go to file explorer (the folder icon at the bottom of the taskbar) and go to This PC and double click Virtual Box guest additions CD. Install the one ending in AMD64.
![Guest_Additions2](https://github.com/user-attachments/assets/2373fbdb-c63e-4a3d-90ce-23bf4cfb6109)

Restart Windows and login as Administrator again

### Configure the Network Adapters of the Domain Controller

Login

Right click on the network icon in the lower right of the Windows Server

Change adapter settings
![Change_Adapter_settings](https://github.com/user-attachments/assets/6fa10e82-f66a-4053-a3d0-7ffa23e92335)

Right click on one of the adapters and click Status and then Details

The one that has an IP starting with 10 is your Internet adapter. Right click and rename the adapter to Internet.
![Internet_IP](https://github.com/user-attachments/assets/eda13b12-9f33-4c0e-be8c-95378e359616)

The one that has an IP starting with 169 is your Internal adapter. Rename it to Internal. 
![Internal_identification](https://github.com/user-attachments/assets/71d5fde7-4dc9-4137-b6a3-88b0305c3fc0)
![Rename_Adapters](https://github.com/user-attachments/assets/73c53ac2-74df-49b9-8440-913dc24577e8)

This internal adapter will connect to the internal adapters of the other VM's/computers later on in our lab.  We will need to give it a static IP and DNS address.

Right click properties select IPV4

Use the following IP:172.16.0.1

Subnet mask 255.255.255.0

Use the following DNS:

Preferred DNS: 172.16.0.1 (you can also use 127.0.0.1 but I have better luck using the same IP)

Alternate DNS: you can leave this blank

![Internal_IP](https://github.com/user-attachments/assets/faf6e4c4-75cb-4605-bd3d-9cf97875fe3b)


### Give the Domain Controller a name

Even though the VM is named DC, its "Windows" name is still default.  We should rename it to DC.

Right click on the Start menu and select System and select Rename this PC and name it DC
![System](https://github.com/user-attachments/assets/8a4c5b89-e2bc-44a9-9c48-79a8fdf51390)
![Rename](https://github.com/user-attachments/assets/ca4366a7-ca01-4355-ab49-53377df39b18)
![Rename_to_DC](https://github.com/user-attachments/assets/d617c09d-2893-4214-b7f6-36ca18a901d3)
![Restart_Rename](https://github.com/user-attachments/assets/5355dd69-e99b-45f3-a63b-f3ac505d1e4d)

Restart Windows Server and login again

### Install Domain Services, DHCP Server, DNS Server, Remote Access

At the Server Manager dashboard hit Add Roles and Features

Hit next on the Before you Begin screen

Check the Role-based or feature-based installation and hit Next
![Add_roles_features](https://github.com/user-attachments/assets/963b4491-ad84-4ea7-baeb-1ecf14f900cb)

Check select a server from the server pool; it should be just the DC server
![Select_server_pool](https://github.com/user-attachments/assets/95d022c0-434a-48d4-be55-daba3edd9971)

Install Active Directory DHCP Server, DNS Server, Remote Access
![Install_everything](https://github.com/user-attachments/assets/6d779d0e-3e30-4e8e-a8e8-582ff376e612)

Install Routing, Direct Access and VPN
![Select_Routing](https://github.com/user-attachments/assets/3c3c5535-6d63-4bca-90d9-333e13d65f62)

Hit next until you get to Install and then Finish

### Promote this server to a Domain Controller

Go up to the yellow flag on upper right of the Server Manager dashboard
![yellow_flag](https://github.com/user-attachments/assets/28cb8765-43bf-49e4-b58e-eb0ce20acb46)

Promote this server to a domain controller

Add new forest
![Add_forest](https://github.com/user-attachments/assets/8fb04c9e-949e-4d7f-a92b-9c1d84ed961a)

Name the domain mydomain.com for lab purposes

Use the password you created earlier Password1
![DSRM_password](https://github.com/user-attachments/assets/c7a9fbea-1d9b-4e43-9aef-d45dcbba249e)

Leave DNS delegation unchecked

Let the NetBios domain name autofill to MYDOMAIN
![Netbios_MYDOMAIN](https://github.com/user-attachments/assets/77281094-b661-427f-9b0b-44007e73d179)

Keep hitting Next until you get to Install
![Install](https://github.com/user-attachments/assets/4bbfc9b9-7dbd-4813-80b3-451059408680)

Restart Windows Server

### Create domain Admin account

Login to Windows Server

You will notice the sign on name is MYDOMAIN\Administrator

Login with Password1

Hit the Start button

Go to Windows Administrative Tools

Go to Active Directory users and Computers
![AD_Users_Computers](https://github.com/user-attachments/assets/f29a94d0-7803-42da-876e-6da756ebb3db)

Right click on mydomain and create New Organizational Unit and name it _ADMINS
![new_OU_admins](https://github.com/user-attachments/assets/53fcdb4a-462d-4ac6-b412-1d518f1635d3)
![new_OU_admin2](https://github.com/user-attachments/assets/bb2566d0-bc72-45a0-8587-37b220bfb4d8)

Once _ADMINS is created right click on it and create new user
![new_user](https://github.com/user-attachments/assets/f77d1e38-ce1a-446b-b1ef-8f617e1b9e62)

Enter first name, last name and logon name. The typical naming convention for a logon name would be first initial of your first name and full last name. So for John Doe it would be jdoe. For an admin logon put an a- in front of it so it would be a-jdoe. My name is Angel Toquero so I use a-atoquero
![new_user2](https://github.com/user-attachments/assets/27862ece-24bd-4af8-b089-a534c5960bec)

Give the user an easy password for lab purposes, like Password1.

For lab purposes, Uncheck user must change password at logon

For lab purposes, Check password never expires
![new_user3](https://github.com/user-attachments/assets/f4c97920-13c4-4b9e-916e-e9bb63fb848f)

Once created, select the _ADMINS folder and right click the user you just made and select Properties
![domain_admins1](https://github.com/user-attachments/assets/9e0b2ba0-27b0-4d3a-867b-8cc34a16e28c)

Select MemberOf, select Add. Where it says enter the object names to select type in Domain Admins and click Check Names then press OK
![domain_admins2](https://github.com/user-attachments/assets/4d61141a-93e8-4082-a392-500a057c8d69)
![domain_admins3](https://github.com/user-attachments/assets/39057100-194e-457f-bc14-c1f8aae80e0f)

You now have a domain admin account created

Logout and login

At the sign on screen you can Select other user

Type in the domain admin logon name and password you just created

### Configure NAT on the Domain Controller

In Windows Server manager, go to tools in the upper right corner, select Routing and Remote access
![tools_routing_remote_access](https://github.com/user-attachments/assets/b81cd6c8-c22e-406a-8002-dae11725353e)

Right click DC and configure and enable
![Configure_and_enable_Routing](https://github.com/user-attachments/assets/86988c34-34cc-4475-ad99-54b0548cc683)

Select NAT and hit next
![configure_NAT](https://github.com/user-attachments/assets/663d8e2d-0eb1-4bdd-aeeb-2bf06a11c1e5)

Under use the public interface to connect to the Internet you should see your internal and internet adapters

If you don’t see your internal and internet adapters hit cancel, close Routing and Remote access and open it again from the Tools menu. Right click DC, hit configure and enable and see if the network adapters pop up.
![configure_NAT2](https://github.com/user-attachments/assets/727b39b5-4e47-4a47-bdea-c9597e026f2c)

Once they pop up select the Internet adapter. 
![configure_NAT3](https://github.com/user-attachments/assets/88f914f3-eda7-450b-8272-edf397a39d73)

Hit next and finish.

### Configure DHCP server on the Domain Controller

At Server Manager dashboard go up to tools and select DHCP

Under dc.mydomain.com 

Right click IPv4 and select New scope
![new_scope](https://github.com/user-attachments/assets/b43bbfe7-772b-4a7b-afe3-0bd8d006430c)

Name the scope 172.16.0.100-200
![scope_name](https://github.com/user-attachments/assets/a06f79cd-01a2-457b-a607-9dd3fc4bbc89)

Hit next

For IP range:

Start IP address 172.16.0.100

End IP address 172.16.0.200

Length 24, subnet mask 255.255.255.0
![scope_range](https://github.com/user-attachments/assets/15dfd18f-9a50-417b-b9c1-b5d9d8610e9f)

Hit next

For add Exclusions and Delay leave it all blank and hit next
![exclusions_delay](https://github.com/user-attachments/assets/3324c887-b20b-4353-91d6-5e4e6c5ff5d1)

For lease duration I just leave it at 8 hours
![lease_duration](https://github.com/user-attachments/assets/8537e81e-1a34-42f9-87ac-f9b6247be8d7)

Hit next

For configure DHCP options select Yes, I want to configure these options now
![configure_DHCP_options](https://github.com/user-attachments/assets/dddb9042-775c-48b4-9e0e-35f8071effab)

Hit next

The IP of our internal network adapter will be our gateway IP, so input 172.16.0.1 and hit add
![internal_IP](https://github.com/user-attachments/assets/2dc0858c-8aaa-4204-8a45-42732ba04be0)
![default_gateway_IP](https://github.com/user-attachments/assets/e304a464-7310-4735-aec0-17b486e8345d)
![default_gateway_ip2](https://github.com/user-attachments/assets/144d4d49-2647-42d7-b665-11aae7ba61b8)

Hit next

For Domain name and DNS servers:

parent domain is mydomain.com

The IP of our internal network adapter will be the IP of our DNS server

Enter 172.16.0.1 and hit add 
![DNS_IP](https://github.com/user-attachments/assets/2bbe8205-c6fe-4601-b269-892f9979347f)
![DNS_IP2](https://github.com/user-attachments/assets/55180db2-1e73-457e-a7ec-483d1169bb50)

I remove the other IP addresses to be sure 172.16.0.1 is functioning as a valid IP for our DNS server
![DNS_IP3](https://github.com/user-attachments/assets/9c390d1b-1515-4f97-ae4f-5dbb176a3431)

Leave WINS Servers blank and hit next
![WINS_servers](https://github.com/user-attachments/assets/e7d6a7de-2bd2-45e6-9b69-85a7b1aa525f)

Activate scope now next and finish
![Activate_Scope](https://github.com/user-attachments/assets/16f59c13-44bc-46dd-be13-de6279ebb9e3)
![finish](https://github.com/user-attachments/assets/309ba4db-d4a5-45e2-9a53-f2011f35fc7e)

Click on IPv4 to expand it.  See where it says server options.  Right click server options, select configure options, select Router, input 172.16.0.1, hit add and hit OK. Select DNS servers and input 172.16.0.1, hit add and hit OK.
![DHCP_server_options](https://github.com/user-attachments/assets/0ff213dd-7609-455f-873a-eeb9a5a8efe2)
![DHCP_router](https://github.com/user-attachments/assets/bfe155a8-b2b2-433d-9cd3-ec2ae974454b)
![DHCP_DNS](https://github.com/user-attachments/assets/eefbcb21-b935-4e3e-9b27-2177290df899)

Right click dc.mydomain.com and hit authorize
![DHCP_auth](https://github.com/user-attachments/assets/832f5e37-64a7-4869-b6e2-1c88b2a0580d)
![DHCP_auth2](https://github.com/user-attachments/assets/81864dce-f150-4852-91a5-bdf0f1995fdc)

Right click dc.mydomain.com and hit Refresh and IPv4 should turn green
![DHCP_auth_refresh](https://github.com/user-attachments/assets/8792d29d-5800-4709-96ad-1c3497cfeffb)
![DHCP_auth_refresh2](https://github.com/user-attachments/assets/b5d8ce5a-2684-4e09-80ad-edaec0b606cd)

## Client 1

### Create the VM for Client 1

Open VirtualBox and select New to create new VM

Give it a name like CL1 

Select the Windows 10 ISO

Skip unattended install
![CL_VM_name](https://github.com/user-attachments/assets/8a701322-2111-484a-b0c8-b0c18c019aec)

Under hardware give it 4096mb RAM and 2 CPU cores
![CL1_VM_CPU_RAM](https://github.com/user-attachments/assets/c858083a-f228-47f2-a169-4fe7ae6601f0)

For hard drive leave it at default size
![CL1_VM_Hard_drive](https://github.com/user-attachments/assets/ac3c88a2-5c91-44b9-886b-622209872922)

Hit Finish

Under settings go to expert mode

Select Adapter 1, enable it, ensure it is attached to Internal network
![CL1_VM_Network](https://github.com/user-attachments/assets/b3cae830-3506-4075-bbad-8da7772b25a4)

### Install Windows 10 on CL1

Hit Start on the CL1 virtual machine

Install Windows 10

Select I don’t have a product key
![CL1_windows_install](https://github.com/user-attachments/assets/e78faa16-0b62-42a9-b87c-5b49cf10333c)

Select Windows 10 Pro
![CL1_Windows_Pro_install](https://github.com/user-attachments/assets/878eebf7-ec1e-42a2-8453-bc90c9ff37e3)

Hit next on unallocated space

For the next few screens I would select “I don’t have internet” and “Limited Setup”

Give the initial account a username and password.  For lab purposes type in “User” for the login and leave password blank.

I would opt out of all the options for the next few screens until setup is complete.

Once setup is complete, Install VM guest additions.  Go to devices at the top of the VM window and select Insert Guest Additions CD Image.

Go to file explorer, This PC, and double click Virtual Box guest additions CD.  Install the one ending in AMD64.
![VBox_WindowsAdditions_AMD64](https://github.com/user-attachments/assets/09ecf274-bcc2-4902-9279-9d916f732460)

Restart Windows 10

### Change Windows name to CL1 and join domain

Login as User

Right click Start Menu

Go to System
![CL1_system](https://github.com/user-attachments/assets/73f11ade-d87f-4dbb-84a3-9132f6b85a8e)

Rename this PC advanced
![Cl1_rename_PC_advanced](https://github.com/user-attachments/assets/4cef4d90-16a6-4ccc-9e28-3c5354a109eb)

Change name
![Cl1_rename_change](https://github.com/user-attachments/assets/d8764a50-09ed-4a5c-aecd-b0bffc7c7adc)

Rename it to CL1

Domain is mydomain.com
![Cl1_rename_change2](https://github.com/user-attachments/assets/a7a82691-b572-4635-ae06-96be2c3e1162)

Use the domain admin account credentials (the user you created in the _ADMIN folder on the DC) to join mydomain.com
![Cl1_rename_change_3](https://github.com/user-attachments/assets/147abb5c-02e4-4693-9591-f1b528809c07)

Restart Windows 10

Login as domain admin

Go to Network and Sharing center, turn discovery on and allow this computer to see and be seen by other computers on the domain


## Client 2

### Create the VM for Client 2

Open VirtualBox and select New to create new VM

Give it a name like CL2

Select the Windows 10 ISO

Skip unattended install

Under hardware give it 4096mb RAM and 2 CPU cores

Hit Finish

Under settings go to expert mode

Select Adapter 1, enable it, ensure it is attached to Internal network

### Install Windows 10 on CL2

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

### Change Windows name to CL2 and join domain

Login as User

Right click Start Menu

Go to System

Rename this PC advanced

Change name

Rename it to CL2

Domain is mydomain.com

Use the domain admin account credentials (the user you created in the _ADMIN folder on the DC) to join mydomain.com

Restart Windows 10

Login as domain admin

Go to Network and Sharing center, turn discovery on and allow this computer to see and be seen by other computers on the domain


## File Server

### Create VM for File Server

Open VirtualBox and select New

Give it a name like FS1 

Select the Windows 2019 server iso

Skip unattended install
![FS1_VM](https://github.com/user-attachments/assets/70e72ef9-426a-4052-9155-eba1d6f828db)

Under hardware give it 4096mb RAM and 2 CPU cores
![FS1_VM2](https://github.com/user-attachments/assets/b1332735-3e95-4171-83ed-b5b7a1440a73)

Hit Finish

Under settings go to expert mode

Go to Network

Select Adapter 1, enable it, ensure it is attached to Internal network

### Install Windows Server on FS1

Hit Start on the FS1 virtual machine

Install Windows Server 2019 Standard Edition (desktop edition)

Give the initial account a password. For lab purposes make it easy like “Password1”

Install VM guest additions. Go to devices at the top of the VM window and select Insert Guest Additions CD Image.
![FS1_guest_additions](https://github.com/user-attachments/assets/a8ed79aa-3ff0-4ef8-935f-85b91b1e1bf4)

Go to file explorer, go to This PC, open the Virtual Box guest additions CD. Install the one ending in AMD64.
![FS1_guest_additions2](https://github.com/user-attachments/assets/c73dd66e-62ec-4844-b8d2-89c9c784fae8)

Restart Windows Server

### Change Windows name to FS1 and join domain

Log into Windows Server

Right click Start Menu

Hit System
![start_system](https://github.com/user-attachments/assets/7266e18c-873a-47e7-9ae5-684dc2d4d924)

Hit System Info

Under computer name, domain and workgroup settings select change settings
![change_settings](https://github.com/user-attachments/assets/0ce77c69-0674-4c57-bcfc-2f2651acc93f)

Click change
![change](https://github.com/user-attachments/assets/61339bd2-8bcf-44f7-8f89-158e12d314f7)

Rename it to FS1

Domain is mydomain.com

Use the domain admin account for credentials
![change_name_auth](https://github.com/user-attachments/assets/7ec0e8c3-4f6b-42aa-8b79-0a8102aae4dc)

Shut down Windows server

### Add a second hard drive to FS1 VM

Back to Virtualbox Manager 

FS1 should be powered off.

Select FS1 and go to Settings.
![settings](https://github.com/user-attachments/assets/71da2b8f-cbc3-4570-b45c-dff46c8d2d58)

Go to Storage

Next to Controller: SATA there is an icon to add a hard disk.  Click it.
![add_hard_disk](https://github.com/user-attachments/assets/6ff3cc15-5a7e-4f08-a53f-bd9d18c65204)

At the Hard Disk selector screen hit Create
![create_HD_1](https://github.com/user-attachments/assets/ce79a52f-1098-418c-838d-b5a19fd62b8b)

Use the default name.  Mine is called FS1_1

For size 10 gigs will be enough for the lab.
![create_HD_2](https://github.com/user-attachments/assets/d0892803-157b-442a-b0d9-9eb312934272)

Hit finish 

In Hard Disk selector you will see it is listed under Not Attached.  
![unattached_drive](https://github.com/user-attachments/assets/e0ba3955-9d82-478c-b239-4e477420fcce)

Just double click on the name and it will attach itself.  

You'll see it listed under Controller: SATA

![attached_drive](https://github.com/user-attachments/assets/0bea0629-0d34-49b1-9464-72d6f49b2092)

### Add the second hard drive to Windows

Start up the FS1 virtual machine again

Log into Windows Server with the domain account we created earlier

Right click on the start menu and hit Run

Type in diskmgmt.msc
![dismgmt](https://github.com/user-attachments/assets/96ca6991-5016-4d25-a80b-b647d3b0e253)

The disk we just attached to the FS1 virtual machine will now initialize in Windows

Once initialized it will resemble a black bar
![initialize_disk](https://github.com/user-attachments/assets/07bf3a47-74f6-48bf-b6ec-df2f05e7a8f3)

Right click on it and create a simple volume using all the default settings
![new_simple_volume](https://github.com/user-attachments/assets/dfc833ca-dcfb-47d5-9f66-be6711ea1e8a)
![new_simple_volume2](https://github.com/user-attachments/assets/861edd91-67d7-4f8b-b7b9-767432a86d5c)
![new_simple_volume_4](https://github.com/user-attachments/assets/36cee744-0bc3-414a-9231-77d87f42ad2d)
![New_simple_volume5](https://github.com/user-attachments/assets/7af59256-3961-452d-8847-c17d236ae9f1)

Once finished you can go to file explorer and under This PC see that there is now a second hard drive.  Mine is setup as E: drive
![network_share_1](https://github.com/user-attachments/assets/711159b1-bcf4-4ee1-a9be-578f3e989baa)

### Create a network share in Windows

For lab purposes we’re going to create a network share at the root of E: drive. 

Create a folder at the root of the E: Drive called SoftwareDeploy. 
![SoftwareDeploy_folder](https://github.com/user-attachments/assets/b739a3d6-6cbf-4fc3-9b12-6a7708cd839e)

Right click SoftwareDeploy

Hit properties
![Software_deploy_properties](https://github.com/user-attachments/assets/15e096e1-5074-4603-9466-6e297f4da2c1)

Go to the sharing tab

Click share
![FS1_share](https://github.com/user-attachments/assets/4d2427de-2560-4bda-b7ef-1cd430b4558f)

Type in Domain Computers and hit Add

Type in Domain Users and hit Add
![FS1_share_domain_admins_users](https://github.com/user-attachments/assets/1072d8af-e186-436e-96d0-faceb65ea4cf)

Give both only Read only access.  

Hit Share at the bottom.

Go to the Security tab

Edit permissions 

Add Domain Users 

Add Domain computers
![domain_computers_security_add](https://github.com/user-attachments/assets/ef326b7b-3194-4695-a657-1007b4453e70)

Give both read access only 
![domain_computers_security_read_only](https://github.com/user-attachments/assets/b1058977-d104-4a74-b924-37a6e850cc8f)


Hit OK and finish

### Download Firefox and Chrome

At the server manager dashboard hit Configure this local server and turn off IE Enhanced Security Configuration

![configure_this_local_server](https://github.com/user-attachments/assets/da54d6f0-b542-43b2-a0e7-2546af4dd747)
![turn_off_ie_security2](https://github.com/user-attachments/assets/920fe2cc-3047-4537-a08b-1802ff207f70)
![FS1_IE_security_off_2](https://github.com/user-attachments/assets/7e517d24-63ed-4055-aaea-a06093ca01d4)

Open Internet Explorer
Go to https://www.mozilla.org/en-US/firefox/all/desktop-release/](https://www.mozilla.org/en-US/firefox/all/desktop-release/

Download the Windows 64-bit MSI of Firefox and place it in the SoftwareDeploy folder at the root of the E: drive. 

Go to https://chromeenterprise.google/download/

Download the Windows 64-bit MSI of Chrome and place it in the SoftwareDeploy folder at the root of the E: drive.  

Make sure to get the MSI versions of the installation files because we will need that version to deploy it later in the lab.

### Turn on services to make FS1 visible to domain computers

By default Windows server has services disabled or off that prevents network discovery

Right click on start

Click Run
![run](https://github.com/user-attachments/assets/fefa3673-4387-45ec-b686-47ce9d4eeb35)

Type in Services.msc
![services msc](https://github.com/user-attachments/assets/e8144858-e7f8-4b0e-8252-d12a23e11b66)

Right click the following services and set to automatic and right click to start them:

Function Discovery Provider Host
![function_discovery_provider_auto](https://github.com/user-attachments/assets/3ad5443e-2433-4786-befb-9928109c9f87)
![Function_Discovery_provider_host_start](https://github.com/user-attachments/assets/4d93ef5e-f811-4f20-bf85-ff5528d6aa7a)

Function Discovery Resource Publication
![Function_discovery_resource_publication_start](https://github.com/user-attachments/assets/841e4a0a-b22d-4512-ac5d-47fb3987385b)

SSDP Discovery
![ssdp_auto](https://github.com/user-attachments/assets/b19308f8-415f-4d09-9dc9-7d51240bde58)
![SSDP_discovery_start](https://github.com/user-attachments/assets/be19d609-adcd-4980-af92-c94a97ecff5f)

UPnP Device Host
![UPNP_auto](https://github.com/user-attachments/assets/9c32f817-f2d9-4f21-9c7b-9ae560e66602)
![UPNP_start](https://github.com/user-attachments/assets/dba6a6d9-bf2b-4a02-9072-53e144dad69f)

Right click on the network icon on the desktop
![right_click_network_adapter](https://github.com/user-attachments/assets/6850942b-156b-4cb1-8ef9-47d393357cd8)

Open network and internet settings
![open_network_sharing_settings](https://github.com/user-attachments/assets/2f5b4a2d-b0f3-4c8a-9ae7-c3094cb116bd)

Click Network and Sharing Center
![open_network_and_sharing_center](https://github.com/user-attachments/assets/934dd498-4f30-49f5-ab36-ee89cd71d76e)

Click Change advanced sharing settings
![change_advanced_sharing_settings](https://github.com/user-attachments/assets/4da2a102-c625-414b-9229-139874d9d7ff)

Under Domain (current profile) Turn on network discovery
![change_advancedsharing_settings](https://github.com/user-attachments/assets/6691f10a-6e61-42ec-8617-bb0d93966ecb)
![turn_on_network_discovery](https://github.com/user-attachments/assets/9beee97a-50a4-4242-a001-7ed2b4c6e7a7)

## Group Policy Edit

### Create new Organizational Unit

An organizational unit is a way to put computers or users into specific groups

You can then apply a group policy to these organizational units that would affect only the users or computers inside the group

Login into DC

At the Server Manager dashboard go to Tools

Open Group Policy Management

Right click mydomain.com

Click New Organizational Unit and call it _mydomain_computers
![create_new_OU](https://github.com/user-attachments/assets/cb2a6fcf-0f91-4821-ab62-fb9b804ab77f)

Right click _mydomain_computers and create another organizational unit and call it Chrome

Right click _mydomain_computers and create another organizational unit and call it Firefox

![create_new_OU_under_mydomain_computers](https://github.com/user-attachments/assets/86baac8b-2aaf-4622-8ac7-a77a52278a6d)
![create_chrome_firefox_OU](https://github.com/user-attachments/assets/ed1e9d87-722a-4ea6-ab1b-30c1884b20ae)

At the Server Manager dashboard go to Tools

Open Active Directory Users and Computers
![AD_users_computers](https://github.com/user-attachments/assets/9e204966-6faa-4da1-9fb4-41b461380dcf)

Under mydomain.com you will see a built-in organizational unit called Computers

Inside Computers you will see all the computers connected to the domain such as FS1, CL1 and CL2
![AD_computers](https://github.com/user-attachments/assets/e8522441-71ef-44f2-b1fa-4b1acf15084d)

Drag CL1 to Chrome
![Cl1_chrome_OU](https://github.com/user-attachments/assets/756359fa-d0d4-47b8-9245-359bd89140f2)
![CL1_chrome_OU2](https://github.com/user-attachments/assets/db74353b-7274-4a51-a6d9-10da7108b5df)

Drag Cl2 to Firefox
![CL2_firefox_OU](https://github.com/user-attachments/assets/9a32a9fd-1ff6-4d94-9020-33ff84f37ccc)

### Create Group Policy
#### Install Chrome to the computers located inside the Chrome organizational unit

At the Server Manager dashboard go to Tools

Open Group Policy Management

Under mydomain.com open _mydomain_computers
![create_chrome_firefox_OU](https://github.com/user-attachments/assets/805e56d2-2d6a-4287-b6fc-68efac57d7c3)

Right Click Chrome

Click Create a GPO in this domain and link it here

Give the group policy object a new like Chrome_deploy
![GPO_chrome_deploy](https://github.com/user-attachments/assets/416e9fe7-f056-48ed-8387-46597855d272)

Right Click Chrome_deploy and click Edit
![GPO_chrome_deploy2](https://github.com/user-attachments/assets/67a093ae-27c2-4684-880f-f19cd9fd8e97)

This should open the Group Policy Management Editor for Chrome_deploy

Under Chrome_deploy open Computer Configuration > Policies > Software Settings > Software Installation

Right click software installation and select new > Package
![GPO_chrome_deploy3](https://github.com/user-attachments/assets/b0ca561a-86c1-4d34-a3c9-368f9e145686)

Go to the network 

Type in //FS1/SoftwareDeploy into the address bar or click on FS1 and get into SoftwareDeploy

Choose the Chrome msi 

![GPO_chrome_deploy4](https://github.com/user-attachments/assets/c226cdf9-05e1-4bf9-ab6c-8a84564f51f1)

Choose Assigned and hit OK

![GPO_chrome_deploy5](https://github.com/user-attachments/assets/101cefc8-b38a-401c-abb3-5d4a71470bbd)
![GPO_chrome_deploy6](https://github.com/user-attachments/assets/39808e3c-6528-4e88-a5b2-9d2c89f9be41)

### Create Group Policy
#### Install Firefox to the computers located inside the Firefox organizational unit

At the Server Manager dashboard go to Tools

Open Group Policy Management

Under mydomain.com open _mydomain_computers

Right Click Firefox 

Click Create a GPO in this domain and link it here
![GPO_firefox_deploy](https://github.com/user-attachments/assets/532e9df6-dded-48ae-944e-16bfa67150c4)

Give the group policy object a new like Firefox _deploy
![GPO_firefox_deploy2](https://github.com/user-attachments/assets/198f48ba-95a4-4046-b0b6-4e8daf578efa)

Right Click Firefox _deploy and click Edit
![GPO_firefox_deploy3](https://github.com/user-attachments/assets/2e6bd09c-ab8d-4a7b-93a7-89793b787630)

This should open the Group Policy Management Editor for Firefox _deploy

Under Firefox _deploy open Computer Configuration > Policies > Software Settings > Software Installation

Right click software installation and select new > Package
![GPO_firefox_deploy4](https://github.com/user-attachments/assets/c5671991-90f1-4669-953b-dc791edc0201)

Go to the network 

Type in //FS1/SoftwareDeploy into the address bar or click on FS1 and get into SoftwareDeploy

Choose the Firefox  msi 

![GPO_firefox_deploy5](https://github.com/user-attachments/assets/6faa1e9c-d2d9-42a6-b12c-b291cb97cb6d)

Choose Assigned and hit OK

![GPO_firefox_deploy6](https://github.com/user-attachments/assets/3c136bfc-c9e4-40f8-9ad2-fb60e8444a7d)
![GPO_firefox_deploy7](https://github.com/user-attachments/assets/84c68257-44a1-49d4-9a15-c9642061b97f)

### Test Group Policy on CL1

CL1 should be in the Chrome organizational unit, which in turn has the Chrome_deploy group policy object

Login to CL1 with domain account

Right click Start menu and hit Run

Type in CMD and hit OK

Type in gpupdate /force
![gpupdate_force](https://github.com/user-attachments/assets/8444339c-609d-4eeb-a5c1-f823d91f9214)

Restart Windows

Login to CL1 again with domain account

Cl1 should have Chrome installed
![gpupdate_force2](https://github.com/user-attachments/assets/23990de4-ccef-490b-9e44-fdac2efb56b9)

### Test Group Policy on CL2

CL2 should be in the Firefox organizational unit, which in turn has the Firefox_deploy group policy object

Login to CL2 with domain account

Right click Start menu and hit Run

Type in CMD and hit OK

Type in gpupdate /force

Restart Windows

Login to CL2 again with domain account

CL2 should have Firefox installed

# Powershell
Logon to DC with domain admin credentials

Open PowerShell ISE with admin privileges

## New AD Organizational Unit
### New-AdOrganizationalUnit
We'll create a new organizational unit called _Users at the root of mydomain.com

Type in:

New-ADorganizationalUnit -Name "_Users" -Path "DC=mydomain,DC=com"

![NewADOrganizationalUnit1](https://github.com/user-attachments/assets/18feebba-7d02-46b3-bc26-038fab2281f7)

If you go to Active Directory Users and Computers you should now see a new organizational unit under mydomain.com called _Users

![NewADOrganizationalUnit2](https://github.com/user-attachments/assets/a0125daf-31cd-49d7-b6c1-390eb02a7aaf)

## New AD User
### New-AdUser
Let's put a new user in the _Users folder we just created

In PowerShell type in:

New-AdUser -Name "Princess Peach" -SamAccountName "ppeach" -UserPrincipalName "ppeach@mydomain.com" -Path "OU=_Users,DC=mydomain,DC=com" -AccountPassword (ConvertTo-Securestring "Password1" -AsPlainText -Force) -Enable $true

![NewADUser1](https://github.com/user-attachments/assets/a1da7542-bbcf-4854-8cb5-2f3e7554821f)

IF you go back to the _Users organizational unit we created earlier you should now see Princess Peach inside

![NewADUser2](https://github.com/user-attachments/assets/4938305f-3f4f-4340-9527-14f33b3681d0)


## Add Group Member
### Add-ADGroupmember
Let's make Princess Peach a domain admin

In PowerShell type in:

Add-ADGroupmember =Identity "Domain Admins" -Members "ppeach"

![Add-ADGroupmember1](https://github.com/user-attachments/assets/26817d9e-68be-4154-976f-bbe7d01ec653)

Princess Peach is still located in the _Users organizational unit but if you right click, properties, and hit MemberOf you should see she is now a member of the Domain Admins

![Add-ADGroupmember2](https://github.com/user-attachments/assets/f72d7546-b2d4-4960-a6c4-e0fd0af0bcf6)


## Move AD Object
### Move-ADObject
Princess Peach is now a member of the Domain Admins and we would like to move her to the _ADMINS organizational unit

In PowerShell:

Move-ADObject "CN=Princess Peach,OU=_Users,DC=mydomain,DC=com" -TargetPath "OU=_Admins,DC=mydomain,DC=com"

![Move-ADObject1](https://github.com/user-attachments/assets/7202ba1c-5ab2-4808-8587-ef765516a912)

Princess Peach should now be located in the _ADMINS organizational unit

![Move-ADObject2](https://github.com/user-attachments/assets/91915711-2d71-42aa-a06d-4c8ad14aa7d7)





