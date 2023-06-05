# Active-directory-set-up


Active Directory is a Microsoft directory service that runs on a Windows Server and allows administrators to manage resources, assign permissions and control access to network resources within an organization


The purpose of this project is to set up and configure an on-premises Active Directory within Azure VMs.

<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)


<h2>Deployment and Configuration Steps</h2>

<p>
<img src="https://i.imgur.com/aJ9juBo.png" height="80%" width="80%" alt="Create Domain Controller"/>
</p>
<p>
Step 1: Set up Resources in Azure
  
- Create the Domain Controller VM (Windows Server 2022) named “DC-1”
   - Take note of the Resource Group and Virtual Network (Vnet) that get created at this time
- Set Domain Controller’s NIC Private IP address to be static
- Create the Client VM (Windows 10) named “Client-1”. Use the same Resource Group and Vnet that was created in Step 1.a
- Ensure that both VMs are in the same Vnet (you can check the topology with Network Watcher

</p>
<br />
<hr>

<p>
<img src="https://i.imgur.com/CBNCkGM.png" height="80%" width="80%" alt="NIC to Static"/>
</p>
<p>
Step 2. Ensure Connectivity between the client and Domain Controller
  
- Log in to the Domain Controller and enable ICMPv4 in on the local windows Firewall
  - Login to the Domain Controller in the Remote Desktop
  - Open Windows Defender Firewall
  - Select "Advanced Settings" on Left
  - Select "Inbound Rules"
  - Sort by "Protocol"
  - Enable ICMPv4 rules
- Log in to Client-1 with Remote Desktop and ping DC-1’s private IP address with ping -t <ip address> (perpetual ping) to verify connectivity
</p>
<br />
<hr>


<p>
<img src="https://i.imgur.com/9aAxXeI.png" height="80%" width="80%" alt="Install AD"/>
</p>
<p>
Step 3. Install Active Directory

- Login to DC-1 through Remote Desktop
- Install Active Directory Domain Services: 
  - In the Server Manager, Select "Add Roles and Features"
  - Continue- Select Next, Next, Next, 
  - Select "Active Directory Domain Services"
  - "Add Features"; "Next"; "Next"; "Next"; "Install"; "Close"


</p>
<br />
<hr>

<p>
<img src="https://i.imgur.com/RGA5XH7.png" height="50%" width="50%" alt="New Forest Set Up"/><img src="https://i.imgur.com/uGcernN.png" height="50%" width="50%" alt="Promote server to Domain Controller"/>
</p>
<p>
Step 4. Set Up Active Directory

- Click "notification" to Select: "Promote this server to a Domain Controller"
- Select: "Add a new forest" (mydomain.com or your choice)
- Choose a Password and make note of this
- Complete Installation ("Next"; "Next"; "Next"; "Next" and "Install") 
- Allow the server to close, which will disconnect the Remote Desktop. 
- Restart and then log back into DC-1 as user: mydomain.com\labuser
</p>
<br />
<hr>

<p>
<img src="https://i.imgur.com/n5jZdaQ.png" height="80%" width="80%" alt="AD Set Up"/>
</p>
<p>
Step 5. Create Admin and Normal User Accounts in AD 

- Navigate to Active Directory Users and Computers (ADUC)
- Create and take note of names and passwords: 
  - an Organizational Unit (OU) called “_EMPLOYEES” 
  - a new OU named “_ADMINS”
  - a new employee named “Jane Doe” with the username of “jane_admin”
    (For practice purposes, select "Password never expires")

</p>
<br />
<hr>
<p>
<img src="https://i.imgur.com/cO8PUUn.png" height="80%" width="80%" alt="Add Admin"/>
</p>
<p>
Step 6. Add jane_admin to the “Domain Admins” Security Group

- Select the _ADMIN Jane Doe and right click to Select Properties
- Select "Member Of"
- Add Domain Users: "Domain"
- Select "Check Names" to open name options
- Select "Domain Admins"
- Complete by Selecting "Ok"; "Ok"; "Apply"; "Ok"
- Log out and close the Remote Desktop connection to DC-1
- Log back in as mydomain\jane_admin
</p>
<br />

<hr>

<p>
<img src="https://i.imgur.com/fmcc1ZY.png" height="40%" width="40%" alt="Join Client-1 to Domain"/><img src="https://i.imgur.com/d02jnLv.png" height="40%" width="40%" alt="Join Client-1 to Domain"/>
</p>
<p>
Step 7. Join Client-1 to your domain (mydomain.com)

- From the Azure Portal, set Client-1’s DNS settings to the DC’s Private IP address
  - In Azure, Locate DC's Private IP address in the VM DC's Overview
  - Open the VM Client-1
  - Select "Networking"
  - Select the "Network Interface" link
  - Select "DNS Servers" in the Left Column
  - Choose "Custom" DNS Servers
  - Enter the DC's Private IP address as the DNS Server 
  - From the Azure Portal, restart Client-1
- Login to Client-1 (Remote Desktop) as the original local admin (labuser) and join it to the domain (computer will restart) 
  - Log into Client-1 (Remote Desktop) as original local admin (labuser)
  - Right Click Start menu 
  - Select "System"
  - Select "Rename this PC (advanced)"
  - Select "Change"
  - In "Domain" box type: mydomain.com
  - Select "OK"
  - In Computer Name/Domain Changes box: 
    -"mydomain.com\jane_admin" and password
  - Select "OK" and restart when prompted
- Login to the Domain Controller (Remote Desktop)
- Navigate to Active Directory Users and Computers (ADUC)
- Verify Client-1 shows up inside “Computers” container on the root of the domain 


</p>
<br />
<hr>
<p>
<img src="https://i.imgur.com/DYPq1RL.png" height="80%" width="80%" alt="Set up Non-Admin Users"/>
</p>
<p>
Step 8. Setup Remote Desktop for non-administrative users on Client-1

- Log into Client-1 as mydomain.com\jane_admin and open system properties
- Click “Remote Desktop”
- Allow “domain users” access to remote desktop
- You can now log into Client-1 as a normal, non-administrative user
(Andother option is to do this with Group Policy, which allows you to change many systems at once) 

</p>
<br />
<hr>

<p>
<img src="https://i.imgur.com/O29Taca.png" height="80%" width="80%" alt="Create Random Users"/>
</p>
<p>
Step 9. Create random additional users

- Within DC-1 Remote Desktop 
- Open PowerShell ISE by right clicking to "Run as Administrator" 
- Open new file
- Paste the contents of [this script file](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1) into it (randomly creating new users with "Password1" as their passwords for testing purposes)
- Open Active Directory and _EMPLOYEES to see the list of random users being added 

</p>
<br />
<hr>

<p>
<img src="https://i.imgur.com/sW5RkkZ.png" height="40%" width="40%" alt="Random names"/><img src="https://i.imgur.com/irOaD64.png" height="40%" width="40%" alt="Testing log in"/>
</p>
<p>
10. Test by choosing random name and accounts

- Choose a random name, take note of account info
- Log off of Client-1
- Log into Client-1, using new account name to test access
</p>
<br />
<hr>
<p>
<img src="https://i.imgur.com/YlS1NXn.png" height="40%" width="40%" alt="Unlock Account"/><img src="https://i.imgur.com/Y0hOoq7.png" height="40%" width="40%" alt="Reset Password"/>
</p>
<p>
11. Fixing Common Password Issues

- Log into DC-1 
- Navigate to: _EMPLOYEES
- Choose Name and Right Click to find properties
- Select Account
- Unlock Account when user is locked out
  - Check box to Unlock Account
- Reset Passwords
  - Right Click Name
  - Find Drop Down Menu to "Reset Password"

</p>
<br />
<hr>
