# Setup Walkthrough
 
This document covers how the lab was built: a Windows Server domain controller and a second server joined to its domain, both running in Azure. Everything here is the groundwork for the detections that come later.
 
## Prerequisites
 
An Azure free account. Portal navigation is not covered here since it changes often and is well documented. This picks up at the point where the first VM gets created.
 
## Environment
 
- Cloud: Azure free account, East US, resource group `homelab-ad-dg`
- Domain controller: `homelab-dc`, Windows Server 2022 Datacenter: Azure Edition, private IP `172.16.0.4`
- Second machine: `homelab-client`, Windows Server 2022 Datacenter: Azure Edition, private IP `172.16.0.5`
- Network: `vnet-eastus-1` and `snet-eastus-1` (`172.16.0.0/24`), shared by both machines
- Domain: `homelab.local`
## 1. Deploy the domain controller VM
 
I created `homelab-dc` from the Windows Server 2022 Datacenter: Azure Edition image and opened RDP only. The burstable B series sizes were unavailable on my subscription, even after registering the `Microsoft.Compute` resource provider, so I went with `Standard_D2as_v7` (2 vCPUs, 8 GiB). Since that size is not free tier, I deallocate the VM every time I stop working so it only bills while I am using it.
 
![homelab-dc deployed](../screenshots/setup/setup-00-dc-vm-deployed.png)
 
## 2. Install AD DS and promote the server
 
I installed the Active Directory Domain Services role through Server Manager, then confirmed it:
 
```powershell
Get-WindowsFeature -Name AD-Domain-Services
```
 
![AD DS role installed](../screenshots/setup/setup-01-adds-role-installed.png)
 
The "Promote this server to a domain controller" task was greyed out in Server Manager even though the role showed as installed, so I skipped the wizard and did the promotion in PowerShell instead:
 
```powershell
Install-ADDSForest -DomainName "homelab.local" -DomainNetbiosName "HOMELAB" -InstallDns -SafeModeAdministratorPassword (ConvertTo-SecureString "<DSRM password>" -AsPlainText -Force) -Force
```
 
The server restarts partway through, and the first boot afterward sits on "Please wait for the Group Policy Client" for a few minutes. That is normal. Once I was back in, I verified the domain:
 
```powershell
Get-ADDomain
```
 
![Domain created](../screenshots/setup/setup-02-domain-created.png)
 
## 3. Create test accounts
 
I made two accounts: a regular user, and one that will stand in as a service account later.
 
```powershell
New-ADUser -Name "Test User1" -SamAccountName "testuser1" -UserPrincipalName "testuser1@homelab.local" -AccountPassword (ConvertTo-SecureString "<password>" -AsPlainText -Force) -Enabled $true
New-ADUser -Name "Service Account1" -SamAccountName "svc-sql" -UserPrincipalName "svc-sql@homelab.local" -AccountPassword (ConvertTo-SecureString "<password>" -AsPlainText -Force) -Enabled $true
```
 
```powershell
Get-ADUser -Filter * | Select-Object Name, SamAccountName
```
 
![Test accounts created](../screenshots/setup/setup-03-test-users-created.png)
 
## 4. Deploy the second machine on the same network
 
I wanted a Windows 11 client, but Azure requires multi tenant hosting rights to run Windows client images, and a free account does not have them. I used Windows Server 2022 again instead. For domain joins and everything Active Directory does, it behaves the same way a workstation would.
 
The setting that matters here is the virtual network. Azure defaults to creating a new one for each VM, and a machine on a separate network could never find the domain controller. I switched it to the existing `vnet-eastus-1` so both machines share a subnet.
 
![homelab-client deployed on the same network](../screenshots/setup/setup-04-client-vm-deployed.png)
 
## 5. Point the client at the domain controller for DNS
 
The client came up using Azure's default DNS (`168.63.129.16`), which knows nothing about `homelab.local`. A join attempt would have failed with a domain not found error. The domain controller is also running DNS, so I pointed the client at it:
 
```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 172.16.0.4
Get-DnsClientServerAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4
```
 
![Client DNS pointing at the domain controller](../screenshots/setup/setup-05-client-dns-configured.png)
 
## 6. Join the domain
 
```powershell
Add-Computer -DomainName "homelab.local" -Credential (Get-Credential) -Restart
```
 
At the credential prompt I used the domain admin account, `HOMELAB\azureadmin`. The machine restarts on its own once the join succeeds. After it came back I logged in as `.\localadmin` (the `.\` forces the local account) and checked:
 
```powershell
Get-ComputerInfo | Select-Object CsDomain, CsDomainRole
```
 
![Client joined to homelab.local](../screenshots/setup/setup-06-client-domain-joined.png)
 
`CsDomainRole` says MemberServer instead of MemberWorkstation because this is a Server image, which is expected.
 
## 7. Confirm the domain controller sees the client
 
The Active Directory PowerShell module only exists on the domain controller, so this one runs there:
 
```powershell
Get-ADComputer -Filter * | Select-Object Name, DNSHostName
```
 
![Domain controller listing both machines](../screenshots/setup/setup-07-dc-sees-client.png)
 
## 8. Send Security events to Microsoft Sentinel
 
The domain generates the events, but the detections need somewhere to collect and query them. This is the same job the forwarder and indexer did in my Splunk lab. I could not reuse that Splunk instance here because it runs on my laptop, and I did not want to open a port to the internet just so cloud VMs could reach it.
 
I created a Log Analytics workspace, `homelab-law`, in the same resource group and region as the VMs, then enabled Microsoft Sentinel on it. That also starts Sentinel's 31 day free trial. Sentinel may redirect you to the Defender portal, which is expected, since Microsoft is moving it there.
 
To connect the machines, I installed the Windows Security Events solution from the Content hub, opened the Windows Security Events via AMA connector, and created a data collection rule covering both `homelab-dc` and `homelab-client`. Creating the rule installs the Azure Monitor Agent on each VM, so both have to be running. I chose All Security Events over the Common set because I was not sure Common includes event 5136, which a GPO change detection needs. Two small VMs stay far below the 10 GB a day free allowance.
 
I did not trust the connection until data actually arrived. In the Logs page (switched to KQL mode, since Simple mode hides the editor), I ran:
 
```kusto
SecurityEvent
| summarize count() by Computer, EventID
| sort by count_ desc
```
 
![Both machines reporting Security events to Sentinel](../screenshots/setup/setup-08-sentinel-ingestion-verified.png)
 
Both `homelab-dc.homelab.local` and `homelab-client.homelab.local` showed up, including event 4769, the Kerberos service ticket event the Kerberoasting detection depends on. Events that nothing has triggered yet, like 4740 for account lockouts, were absent, which is expected.
 
## Result
 
A working Active Directory domain in Azure with a domain controller, a joined member machine, and test accounts, with Security events from both machines flowing into Microsoft Sentinel and ready for the first detections.
