# Active-Directory-lab
Active Directory lab: Azure domain environment, AD setup and documentation

<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

# Active Directory in Azure Virtual Machines

In this lab, I set up an Active Directory environment using Azure Virtual Machines. I configured a domain controller, connected a Windows client to the domain, created user accounts, and tested remote access.

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines)
- Remote Desktop
- Active Directory Domain Services
- PowerShell
- DNS

<h2>Operating Systems Used</h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Deployment and Configuration Steps</h2>

I started by creating two virtual machines in the same virtual network. DC-1 will act as the domain controller, while Client-1 will be the Windows client.

I assigned DC-1 a static private IP address so the client could reliably connect to it. I then tested the connection by pinging DC-1 from Client-1. The ping didn't work at first, so I enabled the appropriate ICMPv4 rule in the Windows Firewall. After that, the connection worked.

<p>
<img src="https://i.imgur.com/d22FHIm.png" width="80%" alt="Azure virtual machine setup"/>
</p>

<p>
<img src="https://i.imgur.com/HvZBWzc.png" width="60%" alt="Testing the connection between virtual machines"/>
</p>

<img src="https://i.imgur.com/1lrrGPw.png" width="60%" alt="Network connectivity test"/>

Next, I installed Active Directory Domain Services on DC-1 and promoted it to a domain controller. I created a new forest called `mydomain.com` and restarted the server. After logging back in with `mydomain.com\labuser`, I opened Active Directory Users and Computers to check the setup.

<img src="https://i.imgur.com/cGjvRke.png" width="80%" alt="Active Directory Users and Computers"/>

I then created two organizational units named `_EMPLOYEES` and `_ADMINS` to keep the user accounts organized. I also created a user named Jane Doe with the username `Jane_admin` and added the account to the Domain Admins group.

<img src="https://i.imgur.com/hL7g5Y5.png" width="80%" alt="Creating organizational units"/>

<img src="https://i.imgur.com/kcgvzdE.png" width="50%" alt="Creating a domain user"/>

After setting up Active Directory, I changed Client-1's DNS settings in Azure to use DC-1's private IP address. I restarted the client and checked that it was using the correct DNS server.

<img src="https://i.imgur.com/jbrGTXW.png" width="80%" alt="Client DNS settings"/>

<img src="https://i.imgur.com/kvcm2cY.jpg" width="80%" alt="Client virtual machine"/>

Once DNS was configured, I joined Client-1 to the `mydomain.com` domain using my domain credentials. After restarting the machine, I checked that it had successfully joined the domain.

<img src="https://i.imgur.com/Ze0Em5e.png" width="80%" alt="Client joined to the domain"/>

I also configured Remote Desktop on Client-1 so regular domain users could log in. I updated the access settings and allowed the appropriate domain users to connect.

<img src="https://i.imgur.com/SApOKiE.png" width="80%" alt="Remote Desktop settings"/>

To test the setup with multiple accounts, I used a PowerShell script to create thousands of domain users. I then selected one of the accounts and tried logging in to Client-1 through Remote Desktop.

<img src="https://i.imgur.com/EzWG8ug.png" width="80%" alt="PowerShell script for creating users"/>

<img src="https://i.imgur.com/Gkpe68K.png" width="60%" alt="Generated domain users"/>

<img src="https://i.imgur.com/n3gMwQV.png" width="60%" alt="Remote Desktop login test"/>

The script created a user with the username `bab.hubo`, and I was able to log in to Client-1 with that account as a regular domain user. This confirmed that the domain setup and user access were working as expected.
