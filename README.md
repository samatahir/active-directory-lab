<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

# Active Directory Lab — Azure Domain Environment

In this lab, I demonstrate building a small Active Directory environment on Azure Virtual Machines. I documented the architecture, deployment steps, configuration commands, and validation checks I use[...] 

## Lab goals

- Deploy a simple AD lab in Azure with one domain controller (DC-1) and one client VM (Client-1).
- Configure DNS, promote the domain controller, and create OUs and user accounts.
- Join a Windows client to the domain and validate authentication and remote access.
- Automate user provisioning with PowerShell for testing scale and access scenarios.

## Environments and Technologies

- Microsoft Azure (Virtual Machines, Virtual Network)
- Active Directory Domain Services (Windows Server)
- Remote Desktop (RDP)
- PowerShell (automation and bulk account creation)
- DNS (Windows DNS role)

## Operating systems

- Windows Server 2022 (Domain Controller)
- Windows 10 (Client)

## Architecture overview

- DC-1: Domain controller and DNS server, assigned a static private IP inside the lab VNet.
- Client-1: Windows client in the same VNet, configured to use DC-1 as its DNS server.

## Deployment and key steps

1. Provision two Azure VMs in the same Virtual Network and subnet: DC-1 (Server) and Client-1 (Client).
2. Assign a static private IP address to DC-1 (Azure NIC configuration) to ensure reliable DNS resolution.
3. Enable the necessary NSG rules or Windows Firewall ports for RDP, DNS (TCP/UDP 53), and AD replication if extending the lab.
4. Install Active Directory Domain Services on DC-1 and promote to a domain controller (example PowerShell):

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Import-Module ADDSDeployment
Install-ADDSForest -DomainName "mydomain.local" -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) -InstallDNS
```

5. After promotion and reboot, create Organizational Units and test user creation (GUI or PowerShell):

```powershell
New-ADOrganizationalUnit -Name "EMPLOYEES" -Path "DC=mydomain,DC=local"
New-ADOrganizationalUnit -Name "ADMINS" -Path "DC=mydomain,DC=local"
New-ADUser -Name "Jane Doe" -SamAccountName "jane.doe" -AccountPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) -Enabled $true
Add-ADGroupMember -Identity "Domain Admins" -Members "jane.doe"
```

6. Configure Client-1's network DNS settings to point at DC-1's private IP, reboot, and join the domain.
7. Enable Remote Desktop on Client-1 and grant domain users access as required.
8. Optionally, use a PowerShell script to generate bulk test accounts to validate scale and authentication workflows.

## Validation and testing

- Ping and resolve DC-1 from Client-1 to confirm DNS and network connectivity.
- Confirm the client shows the domain under System Properties > Computer Name.
- Test logins with created domain users via RDP.
- Verify DNS records and SRV records were created in the DNS management console on DC-1.

## Artifacts and screenshots

<p>
<img src="https://i.imgur.com/d22FHIm.png" width="80%" alt="Azure virtual machine setup"/>
</p>
<p align="center"><em>Figure — Azure virtual machine setup: VM names, VNet, subnet, and NIC layout used in this lab.</em></p>

<p>
<img src="https://i.imgur.com/cGjvRke.png" width="80%" alt="Active Directory Users and Computers"/>
</p>
<p align="center"><em>Figure — Active Directory Users and Computers: Created organizational units (_EMPLOYEES, _ADMINS) and example test user accounts.</em></p>

<p>
<img src="https://i.imgur.com/Ze0Em5e.png" width="80%" alt="Client joined to the domain"/>
</p>
<p align="center"><em>Figure — Client joined to the domain: System Properties showing the computer is a member of mydomain.com.</em></p>

## Notes and lessons learned

- Use a static private IP or a reserved IP for the domain controller to avoid DNS issues when joining clients.
- Ensure the Azure network security rules and Windows Firewall allow DNS and RDP traffic for lab testing.
- Automating user creation with PowerShell is useful to stress-test authentication and session limits, but always secure test credentials and remove them after testing.



