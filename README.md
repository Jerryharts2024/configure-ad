<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1: Setup Resources in Azure
- Step 2: Ensure Connectivity between the client and Domain Controller
- Step 3: Install Active Directory
- Step 4: Create an Admin and Normal User Account in AD
- Step 5: Join Client-1 to your domain (mydomain.com)
- Step 6: Setup Remote Desktop for non-administrative users on Client-1
- Step 7: Create a bunch of additional users and attempt to log into client-1 with one of the users
- Step 8: Done!

<h2>Deployment and Configuration Steps</h2>
<br />
<h3>Step 1: Setup Resources in Azure</h3>
<p>
<img src="https://user-images.githubusercontent.com/131130119/235380469-c636f956-79d5-43b5-957a-169052ec218b.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>

- Create the Domain Controller VM (Windows Server 2022) named “DC-1”
  - create new virtual machine
  - name the VM
- Take note of the Resource Group and Virtual Network (Vnet) that get created at this time
  - ensure that a resource group is created this time and intall the VM in the RG 
- Set Domain Controller’s NIC Private IP address to be static
  - In Azure, go to the network topology change the NIC IP to static
- Create the Client VM (Windows 10) named “Client-1”. Use the same Resource Group and Vnet that was created in Step 1.a
- Ensure that both VMs are in the same Vnet (you can check the topology with Network Watcher

<br />

<h3>Step 2: Ensure Connectivity between the client and Domain Controller</h3>
<p>
<img src="https://user-images.githubusercontent.com/131130119/235381805-edd6fa5d-4636-4264-a642-e21ccd031f4f.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>

- Login to Client-1 with Remote Desktop and ping DC-1’s private IP address with ping -t <ip address> (perpetual ping)
  - run cmd (command prompt)
  - type "ping -t <ip address>"
  - notice that the echo result isn't successful as it will timeout
  
  <p>
<img src="https://user-images.githubusercontent.com/131130119/235381924-35374d42-5669-49b0-a96e-f1cdf910c2c9.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>
  
- Login to the Domain Controller and enable ICMPv4 in on the local windows Firewall
  - run window firewall (wf.msc)
  - click on inbound rule
  - go to tcp protocol column
  - look for ICMPv4 and enable the networking diagnosis echo requests
- Check back at Client-1 to see the ping succeed
  - go back to the perpetual ping on the client 1
  - verify if the request has been recieved.
  - it should be successful at this time after the ICMPv4 request has been enabled in the DC
  
<br />

<h3>Step 3: Install Active Directory</h3>
<p>
<img src="https://user-images.githubusercontent.com/131130119/235382528-af5bc10d-714f-4924-9f79-e647386fe23b.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>
  <p>
<img src="https://user-images.githubusercontent.com/131130119/235383548-bc13e538-bf15-4c34-b4e3-17f8d6081ffe.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>

- Login to DC-1 and install Active Directory Domain Services
    - add role and features -->
    - active directory domain services
    - add features
    - install
- Promote as a DC: Setup a new forest as mydomain.com (can be anything, just remember what it is)
    - add new forest
    - root domain name (mydomain.com)
    - install
- Restart and then log back into DC-1 as user: mydomain.com\labuser
  
<br />
  
  <h3>Step 4: Create an Admin and Normal User Account in AD</h3>
<p>
<img src="https://user-images.githubusercontent.com/131130119/235386531-bcac59c3-3be6-4845-8721-3e0c684418cb.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>
  
- In Active Directory Users and Computers (ADUC), create an Organizational Unit (OU) called “_EMPLOYEES”
    - new 
    - organizational unit 
    - name = _EMPLOYEES
- Create a new OU named “_ADMINS”
    - new 
    - organizational unit 
    - name = _ADMINS
- Create a new employee named “Jane Doe” (same password) with the username of “jane_admin”
    - new
    - user
    - name = Jane Doe 
  
    <p>
<img src="https://user-images.githubusercontent.com/131130119/235387295-32781c87-7f80-4d15-8a55-596c9b704b69.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>
  
- Add jane_admin to the “Domain Admins” Security Group
    - right click (Jane Doe) and go to properties
    - go to security
    - search for domain admin
    - add Jane Doe to this security group to allow access as admin user 
- Log out/close the Remote Desktop connection to DC-1 and log back in as “mydomain.com\jane_admin”
- User jane_admin as your admin account from now on
  
<br />

  <h3>Step 5: Join Client-1 to your domain (mydomain.com)</h3>
<p>
<img src="https://user-images.githubusercontent.com/131130119/235389155-5bb24a7b-a348-4c3a-af65-deccdc37bcd0.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>

- From the Azure Portal, set Client-1’s DNS settings to the DC’s Private IP address
    - go to the network topology
    - dns server and 
    - set Client-1’s DNS to DC’s Private IP address
- From the Azure Portal, restart Client-1
- Login to Client-1 (Remote Desktop) as the original local admin (labuser) and join it to the domain (computer will restart)
  - go to rename this PC
  - connect the domain name to the client one PC and save
  - computer will restart
- Login to the Domain Controller (Remote Desktop) and verify Client-1 shows up in Active Directory Users and Computers (ADUC) inside the “Computers” container on the root of the domain
  - login to the domain controller
  - open active directory 
  - go to active directory user and computer
  -inside the computer folder, check if client 1 appears in it
- Create a new OU named “_CLIENTS” and drag Client-1 into there 
   
<br />

  <h3>Step 6: Setup Remote Desktop for non-administrative users on Client-1</h3>
<p>
<img src="https://user-images.githubusercontent.com/131130119/235403363-b24fae3b-abbb-45b0-9840-335c601df477.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>

- Log into Client-1 as mydomain.com\jane_admin and open system properties
  - Click “Remote Desktop”
  - Allow “domain users” access to remote desktop
  
 <p>
<img src="https://user-images.githubusercontent.com/131130119/235403575-301ebfae-7bf2-43bf-80d3-e843b30f1cf8.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>
  
- You can now log into Client-1 as a normal, non-administrative user now
- Normally you’d want to do this with Group Policy that allows you to change MANY systems at once

<br />

  <h3> Step 7: Create a bunch of additional users and attempt to log into client-1 with one of the users </h3>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Deployment and Configuration Steps"/>
</p>
  
- Login to DC-1 as jane_admin
  - Open PowerShell_ise as an administrator
  - Create a new File and paste the contents of the script into it (https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)
  - Run the script and observe the accounts being created
- When finished, 
  - open ADUC and observe the accounts in the appropriate OU
  - attempt to log into Client-1 with one of the accounts (take note of the password in the script)

<br />
<p>
  DONE!
  </p>
