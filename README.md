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
- Windows 10 (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup Resources in Azure
- Set Domain Controller’s NIC Private IP address to static
- Create Client-1 VM
- Ensure Connectivity between the client and Domain Controller
- Enable ICMPv4 in on the local windows Firewall and check connectivity
- Install Active Directory
- Create an Admin and Normal User Account in Active Directory


<h2>Deployment and Configuration Steps</h2>

<p>
</p>
<p>
Create the Domain Controller VM (Windows Server 2022) named 'DC-1'. Take note of the Resource Group (which can be created on the VM creation page or <a href = "https://github.com/oraljr/rg-setup"> from the Resource Group creation page </a>) and Virtual Network (Vnet) that get created here. Select a Region to place both your VMs and you'll want to select a size that has at lest 2 vcpus; anything less will cause the OS on your VM to run slowly. Once everything is filled out, review and create your VM.

![image](https://github.com/oraljr/configure-ad/assets/152557529/569e8e45-ef2b-4959-b989-2ac461548d99)


![image](https://github.com/oraljr/configure-ad/assets/152557529/1d3eabce-514b-4368-b2f9-cb672b868231)

<hr>
  
  Next you'll set the Domain Controller’s NIC Private IP address to be static. 

  First you'll go to Network Settings ->

  ![image](https://github.com/oraljr/configure-ad/assets/152557529/914fcc69-6e96-4652-bb2d-909beb114c6c)


  NIC ->

  ![image](https://github.com/oraljr/configure-ad/assets/152557529/32905b73-1cd0-4198-94d1-36f3d90efeda)


  ipconfig ->

![image](https://github.com/oraljr/configure-ad/assets/152557529/ba60989d-24ae-48c5-8912-8ec7e162eeee)

Then on the Edit IP configuration page you'll change the allocation from Dynamic to Static and select save.

![image](https://github.com/oraljr/configure-ad/assets/152557529/d9528d18-ec88-4f96-b974-40ab3805156a)
<hr>

Next you'll create your second VM named 'Client-1'. Make sure to use the same Resource group as DC-1, check 'licensing' then select Next: Disks and then Next: Networking to reach the networking page.

Ensure that both VMs are in the same Vnet. (**_NOTE_**: if the Vnet from DC-1 isnt showing in the drop dwon menu, then allow more time for the Vnet to fully populate before creating your Client-1 VM) 

![image](https://github.com/oraljr/configure-ad/assets/152557529/a63960f9-e849-4206-bd5a-b7e232b70db0)

Once you've verified the Vnet, select Review + create and if your validation passes you can create your VM.
<hr>
Next you'll ensure connectivity between the client and Domain Controller.
</p>
Login to Client-1 with Remote Desktop and ping DC-1’s private IP address with ping -t (DC-1 private ip address)   
</p>
  
![image](https://github.com/oraljr/configure-ad/assets/152557529/6ee7fbd5-3669-459a-a5f7-6a1d14e1780c)

![image](https://github.com/oraljr/configure-ad/assets/152557529/4598ec4f-7685-472e-a664-4bc6b7003b83)

The ping will time out so you'll need to start another instance of remote desktop and login to the Domain Controller to enable ICMPv4 in on the local windows Firewall.

From the search bar in DC-1 you'll type wf.mfc to open Microsoft Common Console Document -> Inbound Rules

![image](https://github.com/oraljr/configure-ad/assets/152557529/ed24d295-21d2-4276-93c7-023deab439b9)


Sort by protocol -> ICMPv4 -> 

![image](https://github.com/oraljr/configure-ad/assets/152557529/efa80de6-dafb-4c87-bb7c-f92d7f24bad9)

Right click to enable Core Network Diagnosis rules.

Return to Client-1 and the ping should now be responding.

![image](https://github.com/oraljr/configure-ad/assets/152557529/df61574f-e2d1-48c2-b51b-23c4ff6377b9)

Next you'll install Active Directory.
<hr>

First, switch back to DC-1. Then you'll go to Server Manager -> Add roles and features -> Active Directory Domain Services 

(**NOTE**: Make sure to only select Active Directory Domain Services as there are multiple Active Directory available)

![image](https://github.com/oraljr/configure-ad/assets/152557529/180a4e67-a057-4888-8e5e-bd0293428001)

![image](https://github.com/oraljr/configure-ad/assets/152557529/6cc6396d-0618-4f03-8ca7-a793bdf08478)

Add features and select next and install.
<hr>

Next you'll promote the server as a Domain Controller. In the upper right corner you'll see a flag with an action sign next to it in the Server Manager notifications section. You'll select promote this server as a domain controller.

![image](https://github.com/oraljr/configure-ad/assets/152557529/7e2a3878-0783-43dd-a131-1e16964418a4)

![image](https://github.com/oraljr/configure-ad/assets/152557529/92acbd98-78e6-4a36-a312-c91d4c13e970)

Select Add new forest ->

![image](https://github.com/oraljr/configure-ad/assets/152557529/65a88f56-1d2f-4dcd-91ba-e4ca1adce8a9)

Setup a new forest as yourdomainname.com. It can be name it anything, just remember what you choose. 

![image](https://github.com/oraljr/configure-ad/assets/152557529/78f24c9f-70e4-4e54-849d-27592f01e01d)

Choose a password on the next page and just click through 'Next' and install. Once the installation is complete the host will restart.

![image](https://github.com/oraljr/configure-ad/assets/152557529/146fe53c-171b-4d06-ae4e-28c3fc44e329)

Return to DC-1 in Azure and refesh the VM for the public ip address so you can log back into the remote desktop. The ip address should be the same but it might change upon refresh. DC-1 is now a domain controller, so you'll have to long in with the context of the fully qualifed domain name (FQDN) as username. So: yourdomainname.com\whatever username is thats a member of this domain, which you created when you initally set up the VM.

i.e.: yourdomainname.com\o4estjr

![image](https://github.com/oraljr/configure-ad/assets/152557529/fd10935d-c860-4061-a332-d9335b7ff35c)

<hr>
Now you'll create an Admin and Normal User Account in Active Directory.
</p>
From the search bar on DC-1, you'll locate Active Directory Users and Computers and you'll right click to create an Organizational Unit (OU) called '_EMPLOYEES' ( **NOTE** : the underscore and caps are not necessary but they do help to eaisly differentiate the OU's you create from the other folders)

![image](https://github.com/oraljr/configure-ad/assets/152557529/4e74a0f0-e478-4414-becd-79227c9da50f)


</p>
<br />
