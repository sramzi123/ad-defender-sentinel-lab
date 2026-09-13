# Active Directory + Microsoft Defender + Sentinel Lab

This repository documents a cloud-hosted Active Directory environment, built to gain hands-on experience with Microsoft's security stack: Defender for Endpoint and Microsoft Sentinel, using KQL for detection engineering.

The lab consists of a Windows Server domain controller and a domain-joined client, both hosted on Azure, onboarded to Microsoft Defender for Endpoint, with detections investigated and built out in Microsoft Sentinel.

## Current Lab

- Azure (free tier virtual machines)
- Windows Server — Active Directory Domain Services
- Windows client, domain-joined
- Microsoft Defender for Endpoint
- Microsoft Sentinel
- KQL (Kusto Query Language)

## Architecture

*(architecture diagram once the domain is built)*

## Documentation

*(setup writeup once the DC and client are deployed)*

## Detections

- Account Lockout Monitoring — IT operations focused
- GPO Change Monitoring — IT operations focused
- Kerberoasting — T1558.003, cybersecurity focused, built out as a Sentinel analytics rule

## Current Progress

- [ ] Deploy Domain Controller VM
- [ ] Deploy domain-joined client VM
- [ ] Onboard both to Microsoft Defender for Endpoint
- [ ] Account lockout detection
- [ ] GPO change detection
- [ ] Kerberoasting detection
- [ ] Connect to Microsoft Sentinel
- [ ] Build Sentinel analytics rule and triage a real incident

## Repository Structure

```
architecture/     diagram source and exported image, once built
docs/             setup guide and any troubleshooting writeups
detections/       one subfolder per detection, each with its query, writeup, and screenshots
```

## Roadmap

### Phase 1
- Deploy Domain Controller and client VMs in Azure
- Configure Active Directory Domain Services
- Onboard both machines to Microsoft Defender for Endpoint

### Phase 2
- Account lockout detection
- GPO change detection

### Phase 3
- Kerberoasting detection
- Connect Defender data to Microsoft Sentinel
- Build a Sentinel analytics rule and triage a real incident

## Related Projects

- [Splunk + Sysmon Detection Lab](https://github.com/sramzi123/splunk-sysmon-detection-lab) — an earlier, on-prem style SOC lab covering Windows Event Logs, Sysmon, and SPL-based detection engineering
