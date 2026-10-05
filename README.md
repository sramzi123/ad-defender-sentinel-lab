# Active Directory + Microsoft Defender + Sentinel Lab
 
This repository documents a cloud hosted Active Directory environment, built to gain hands on experience with Microsoft's security stack: Defender for Endpoint and Microsoft Sentinel, using KQL for detection engineering.
 
The lab currently consists of a Windows Server domain controller and a second Windows Server machine joined to its domain, both running in Azure. Security events from both machines flow into Microsoft Sentinel. As the project grows, I plan to build detections in KQL and, if licensing allows, onboard both machines to Microsoft Defender for Endpoint.
 
## Current Lab
 
- Azure (free account, East US)
- Windows Server, Active Directory Domain Services
- Second Windows Server machine, joined to the domain
- Microsoft Defender for Endpoint (planned)
- Microsoft Sentinel, fed by the Azure Monitor Agent
- KQL, Kusto Query Language
## Architecture
 
<!-- ![Architecture diagram](architecture/ad-lab-architecture.png) -->
 
## Documentation
 
- [Setup Guide](docs/setup.md) — deploying the domain controller, building the domain, and joining the second machine
## Detections
 
- Account Lockout Monitoring — IT operations focused
- GPO Change Monitoring — IT operations focused
- Kerberoasting — T1558.003, cybersecurity focused, built out as a Sentinel analytics rule
## Current Progress
 
- [x] Deploy domain controller VM
- [x] Configure Active Directory Domain Services
- [x] Create test accounts
- [x] Deploy second machine and join it to the domain
- [ ] Onboard both machines to Microsoft Defender for Endpoint (optional, pending licensing)
- [ ] Account lockout detection
- [ ] GPO change detection
- [ ] Kerberoasting detection
- [x] Connect to Microsoft Sentinel
- [ ] Build Sentinel analytics rule and triage a real incident
## Repository Structure
 
```
architecture/     diagram source and exported image, once built
docs/             setup guide, troubleshooting, and investigation writeups
screenshots/      raw evidence referenced from docs, organized by topic
  setup/
detections/       one subfolder per detection, each with its query, writeup, and screenshots, once built
```
 
## Roadmap
 
### Phase 1
- [x] Deploy the domain controller and a second machine in Azure
- [x] Configure Active Directory Domain Services
- [x] Connect Security events from both machines to Microsoft Sentinel
- [ ] Onboard both machines to Microsoft Defender for Endpoint (optional, pending licensing)
### Phase 2
- [ ] Account lockout detection
- [ ] GPO change detection
### Phase 3
- [ ] Kerberoasting detection
- [x] Connect Security events to Microsoft Sentinel
- [ ] Build a Sentinel analytics rule and triage a real incident
## Related Projects
 
- [Splunk + Sysmon Detection Lab](https://github.com/sramzi123/splunk-sysmon-detection-lab) — an earlier, on prem style SOC lab covering Windows Event Logs, Sysmon, and SPL based detection engineering
