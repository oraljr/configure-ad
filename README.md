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
- Ensure Connectivity between the client and Domain Controller
- Enable ICMPv4 in on the local windows Firewall and check connectivity
- Install Active Directory
- Create an Admin and Normal User Account in Active Directory
- Join Client-1 to your domain
- Setup Remote Desktop for non-administrative users on Client-1
- Create additional users and log into client-1 with one of the users


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

Return to DC-1 in Azure and refesh the VM for the public ip address so you can log back into the remote desktop. The ip address should be the same but it might change upon refresh. DC-1 is now a domain controller, so you'll have to long in with the context of the fully qualifed domain name (FQDN) as username. So: yourdomainname.com\whatever username thats a member of this domain, which you created when you initally set up the VM.

i.e.: yourdomainname.com\o4estjr

![image](https://github.com/oraljr/configure-ad/assets/152557529/fd10935d-c860-4061-a332-d9335b7ff35c)

<hr>
Now you'll create an Admin and Normal User Account in Active Directory.
</p>
From the search bar on DC-1, you'll locate Active Directory Users and Computers and you'll right click to create an Organizational Unit (OU) named '_EMPLOYEES' (the underscore and caps are not necessary, but they do help to eaisly differentiate the OU's you create from the other containers) and another OU named '_ADMINS' then simply right click on the domain to refresh.

![image](https://github.com/oraljr/configure-ad/assets/152557529/4e74a0f0-e478-4414-becd-79227c9da50f)

Next you'll go to the _ADMINS OU, open it up and right click inside for New -> Users and add new user. Fill in the name and create a password. 

![image](https://github.com/oraljr/configure-ad/assets/152557529/c43dc90a-57d7-48be-b0ef-b357162e2bc1)

Next you'll add your new user to the Domains Admins Security Group to make them a domain admin. Right click your user to open properties -> Member of -> Add and type 'domain admins' in the box and select Check Names and it should underline, then Apply your changes and select ok.

![image](https://github.com/oraljr/configure-ad/assets/152557529/89ba08f2-6419-42d4-a322-29d4502a45c1)

Next you'll log out of DC-1 and reopen Remote Desktop and log back in as your newly created admin user.
</p>
i.e.: yourdomainname.com\a-jane
</p>
Next you'll join Client-1 to your domain.
<hr>
From the Azure Portal you'll set Client-1’s DNS settings to the DC’s Private IP address.

Select Client-1 -> Network Settings -> 

![image](https://github.com/oraljr/configure-ad/assets/152557529/95ccc2fc-3222-44bd-bdc1-1db8e9b9eb0f)


NIC ->

![image](https://github.com/oraljr/configure-ad/assets/152557529/572f911a-7557-41d3-8074-13fa2b4b4e08)


DNS servers -> 

![image](https://github.com/oraljr/configure-ad/assets/152557529/36a15e34-08bc-43ff-840c-1556fec89476)


Custom -> 

![image](https://github.com/oraljr/configure-ad/assets/152557529/009ab830-ad33-482d-97e1-f262ac5dbde1)


Enter DC-1's private ip and save.

![image](https://github.com/oraljr/configure-ad/assets/152557529/2ae320f6-6615-46ff-9da9-c6c44c1667da)

Now from the Azure portal, restart Client-1.

![image](https://github.com/oraljr/configure-ad/assets/152557529/5093e993-bd68-460b-aca1-9f956f64aaa4)

Next you'll login to Client-1 (Remote Desktop) as the original local admin (original username only, sans domain name) and join it to the domain. 

Right click start -> System ->

![image](https://github.com/oraljr/configure-ad/assets/152557529/cf509633-cbf6-4333-894c-ca3b8011aac9)

rename this pc advanced -> 

![image](https://github.com/oraljr/configure-ad/assets/152557529/04c6c6fe-d06c-4450-bad8-78fd65f6ff73)


change -> 

![image](https://github.com/oraljr/configure-ad/assets/152557529/a6c60b67-5a98-4d8c-81db-4c73e9a3832d)


domain -> 

![image](https://github.com/oraljr/configure-ad/assets/152557529/2d60f0e1-a294-4963-b7a2-c85bc3d66a16)


Enter the domain name you created in the box (i.e.: yourdoaminname.com) -> 



Enter the credentials of the domain admin account -> yourdomainname.com\(admin user) -> enter admin users password. Welcome to (domainaddress).com should populate somewhere behind the open windows. 

![image](https://github.com/oraljr/configure-ad/assets/152557529/e2a8b5e3-7d8e-48ea-bb10-4657c2783055)

Then host will need to restart.

Next you'll login to the Domain Controller (Remote Desktop) and verify Client-1 shows up in Active Directory Users and Computers (ADUC) inside the 'Computers' container on the root of the domain.

![image](https://github.com/oraljr/configure-ad/assets/152557529/a48bbcb6-1be8-4df1-a2c5-941ef4944cf5)

<hr>
Now you're going to set it up where all noraml domain users are able to remote into Client-1. As of now, only the administrative accounts have remote access into Client-1.

So log into CLient-1 as your domain admin account (i.e.: yourdomainname.com\a-jane), you'll go back to system properties (right click the Start menu and select 'System'). Then you'll select Remote Desktop ->

![image](https://github.com/oraljr/configure-ad/assets/152557529/acfa1c7b-d6b7-4aee-9e2d-04a820f1534c)

Select users that can remotely access this PC -> Add... ->

![image](https://github.com/oraljr/configure-ad/assets/152557529/e7644a26-85d3-4497-a478-1f0d7bb3ebb0)

Type 'domain users" in the box -> Check names -> Ok.

![image](https://github.com/oraljr/configure-ad/assets/152557529/0d1f23e7-1678-4461-8d7a-ed3e51b4a5de)

Essentially now all domain users are allowed to log in to this computer. Now you can log into Client-1 as a normal, non-administrative user.
<hr>
Next you'll create a bunch of additional users and attempt to log into Client-1 with one of the new users.
<hr>
Login in DC- as your admin account and open PowerShell_ISE as an administrator

![image](https://github.com/oraljr/configure-ad/assets/152557529/50eeb0ec-8f9e-4c9b-8f17-6880b782778d)

Select New File in the upper left corner ->

![image](https://github.com/oraljr/configure-ad/assets/152557529/ff9c3f5a-1a3a-45a1-824e-80d78c50ebac)

Paste the contents of you name generator script into Powershell and run the script.

![image](https://github.com/oraljr/configure-ad/assets/152557529/97b36bc9-0bca-4062-8908-8703265c2567)

Observe the accounts being created

![image](https://github.com/oraljr/configure-ad/assets/152557529/056b716d-1cc7-4292-9f59-789b5e75ffdd)

Once complete, open ADUC and refresh the '_EMPLOYEES' container 

![image](https://github.com/oraljr/configure-ad/assets/152557529/0c556c73-3c85-4606-a1e4-bd15517476b0)

Observe the accounts in the appropriate OU.

![image](https://github.com/oraljr/configure-ad/assets/152557529/babb3b69-a8d0-4fb1-ad9b-193231375332)

Now attempt to log into Client-1 with one of the newly added accounts using the password from the script.


![image](https://github.com/oraljr/configure-ad/assets/152557529/e1d8ce77-2abc-446b-b7c6-8265c0ce4759)


![image](https://github.com/oraljr/configure-ad/assets/152557529/b381a765-7d19-413b-9caf-d615a9aca205)


You've done it!! You successfully Deployed Active Directory!! Big sigh...
