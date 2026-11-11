<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

# Active Directory Lab — Azure Domain Environment

Demonstrating how to build a small Active Directory environment in Azure using two virtual machines. It includes the architecture, deployment steps, configuration commands, and validation checks needed to create a working domain environment for testing.

## Lab goals

- Deploy a simple AD environment in Azure with one domain controller and one client VM.
- Configure DNS, promote the domain controller, and create OUs and user accounts.
- Join a Windows client to the domain and validate authentication and remote access.
- Automate user provisioning with PowerShell for testing scale and access scenarios.

## Technologies used

- Microsoft Azure (Virtual Machines, Virtual Network)
- Active Directory Domain Services (Windows Server)
- DNS (Windows DNS role)
- Remote Desktop (RDP)
- PowerShell

## Operating systems

- Windows Server 2022 (Domain Controller)
- Windows 10 (Client)

## Architecture overview

| Component | Role | Operating system | Notes |
| --- | --- | --- | --- |
| DC-1 | Domain controller and DNS server | Windows Server 2022 | Static private IP assigned inside the lab VNet |
| Client-1 | Windows client joined to the domain | Windows 10 | Uses DC-1 as its DNS server |

The lab uses a single virtual network and subnet so both VMs can communicate reliably. The domain controller is configured as the DNS server, and the client is pointed to it during the join process.

## Prerequisites

Need to make sure you have:

- An Azure subscription
- A resource group and virtual network configured for the lab
- Two Azure VMs created in the same subnet
- Network access to RDP into the VMs
- Administrative access to the Windows servers

## Deployment steps

### 1. Provision the Azure VMs

Create two VMs in the same virtual network and subnet:

- DC-1: Domain controller / DNS server
- Client-1: Domain-joined Windows client

Make sure both VMs are in the same VNet so they can resolve each other by name and IP.

### 2. Assign a static private IP to the domain controller

Assign a static private IP address to DC-1 to avoid DNS issues when the client joins the domain.

This is important because the client will be configured to use DC-1 as its DNS server. A stable IP makes configuration simpler and avoids future name resolution problems.

### 3. Open the required ports and firewall rules

Allow the necessary traffic for lab operations:

- RDP (TCP 3389)
- DNS (TCP/UDP 53)
- Active Directory replication and domain traffic

You can do this through Azure Network Security Groups (NSGs) and the Windows Firewall if needed.

### 4. Install AD DS on DC-1

Install the Active Directory Domain Services role and promote the server to a domain controller.

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Import-Module ADDSDeployment
Install-ADDSForest -DomainName "mydomain.local" -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) -InstallDNS
```

After the server reboots, the machine should be a functioning domain controller.

### 5. Create OUs and test users

Create organizational units and sample users to represent a simple enterprise directory.

```powershell
New-ADOrganizationalUnit -Name "EMPLOYEES" -Path "DC=mydomain,DC=local"
New-ADOrganizationalUnit -Name "ADMINS" -Path "DC=mydomain,DC=local"
New-ADUser -Name "Jane Doe" -SamAccountName "jane.doe" -AccountPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) -Enabled $true
Add-ADGroupMember -Identity "Domain Admins" -Members "jane.doe"
```

This gives you a basic AD structure for testing user access and group membership.

### 6. Configure Client-1 to use DC-1 as DNS

On Client-1, change the network adapter DNS settings so it points to the private IP address of DC-1.

Once the DNS is configured, reboot the client and join it to the domain.

### 7. Join the client to the domain

Use the Windows System Properties or PowerShell to join the client machine to the domain.

You can then log in with a domain user account such as `jane.doe` and verify that the client is functioning correctly in the AD environment.

### 8. Enable RDP access for users

If needed, enable Remote Desktop on Client-1 and grant access to domain users or administrators.

This allows you to test user authentication and remote access from within the lab environment.

## Validation and testing

After the setup is complete, validate the environment with the following checks:

- Ping and resolve DC-1 from Client-1 to confirm network connectivity and DNS resolution.
- Verify the client appears as a member of the domain in System Properties > Computer Name.
- Log in to the client using a created domain user account via RDP.
- Check the DNS management console on DC-1 to confirm the expected SRV and host records were created.

## Useful PowerShell examples

```powershell
# View domain controllers
Get-ADDomainController -Filter *

# List users in a domain
Get-ADUser -Filter * | Select-Object Name, SamAccountName

# View groups
Get-ADGroup -Filter * | Select-Object Name
```

## Basic infrastructure and Screenshots

<p>
<img src="https://i.imgur.com/d22FHIm.png" width="80%" alt="Azure virtual machine setup"/>
</p>
<p align="center"><em>Figure — Azure virtual machine setup: VM names, VNet, subnet, and NIC layout used in this lab.</em></p>

<p>
<img src="https://i.imgur.com/cGjvRke.png" width="80%" alt="Active Directory Users and Computers"/>
</p>
<p align="center"><em>Figure — Active Directory Users and Computers: Created organizational units (EMPLOYEES, ADMINS) and example test user accounts.</em></p>

<p>
<img src="https://i.imgur.com/Ze0Em5e.png" width="80%" alt="Client joined to the domain"/>
</p>
<p align="center"><em>Figure — Client joined to the domain: System Properties showing the computer is a member of mydomain.local.</em></p>

## Notes and lessons learned

- Use a static private IP, or a reserved IP, for the domain controller to avoid DNS issues when joining clients.
- Azure network security rules and Windows Firewall allow DNS and RDP traffic for lab testing.
- Automating user creation with PowerShell is useful for stressing authentication and session limits.
- Keep the lab isolated and use test-only credentials to avoid confusion with production environments.

## Summary

This lab provides a simple Azure-based Active Directory environment for learning/testing core AD concepts such as DNS, domain promotion, OU structure, user creation, and domain joining. Its a good starting point for anyone learning how AD works in a virtualized infrastructure.
