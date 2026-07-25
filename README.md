# Active Directory Home Lab

![Project Status](https://img.shields.io/badge/status-completed-brightgreen)
![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-0078D4)
![Windows Client](https://img.shields.io/badge/Windows%2011-Pro-0078D4)
![Hypervisor](https://img.shields.io/badge/Hyper--V-Lab-5C2D91)

## Overview

This project demonstrates the deployment and administration of a small business
Windows domain in a virtualized home-lab environment. I built the environment
from the ground up using Hyper-V, Windows Server 2022, Windows 11 Pro, Active
Directory Domain Services, DNS, DHCP, Group Policy, SMB shares, and NTFS
permissions.

The fictional organization used for the lab is **Cabrera Technologies**, with
the internal domain **cabreralab.test**.

## Objectives

- Deploy a Windows Server 2022 domain controller.
- Configure a private Hyper-V network for the lab.
- Install and configure Active Directory Domain Services and DNS.
- Design an organizational unit structure for a small business.
- Create departmental users and security groups.
- Apply least-privilege SMB and NTFS permissions.
- Join a Windows 11 Pro workstation to the domain.
- Deploy security settings and mapped drives through Group Policy.
- Configure DHCP for automatic client addressing.
- Reproduce, diagnose, and resolve a DNS configuration problem.

## Lab Architecture

```mermaid
flowchart TD
    Host["Windows 11 Pro Host<br>Hyper-V"]
    Switch["LAB-SWITCH<br>10.10.10.0/24"]
    DC["DC01<br>10.10.10.10<br>AD DS · DNS · DHCP · File Services"]
    Client["CLIENT01<br>DHCP Client<br>Windows 11 Pro"]

    Host --> Switch
    Switch --> DC
    Switch --> Client
```

## Environment

| Component | Configuration |
|---|---|
| Hypervisor | Microsoft Hyper-V |
| Domain controller | DC01 — Windows Server 2022 Standard Evaluation |
| Client | CLIENT01 — Windows 11 Pro |
| Domain | cabreralab.test |
| NetBIOS name | CABRERALAB |
| Network | 10.10.10.0/24 |
| Server address | 10.10.10.10 |
| DHCP scope | 10.10.10.100–10.10.10.200 |
| Client DNS | 10.10.10.10 |

## Active Directory Design

```text
Cabrera Technologies
├── Users
│   ├── IT
│   ├── Human Resources
│   ├── Finance
│   └── Sales
├── Computers
│   ├── Workstations
│   └── Servers
├── Groups
├── Service Accounts
└── Disabled Accounts
```

Global security groups were created for role-based access:

- `GG_IT_Users`
- `GG_HR_Users`
- `GG_Finance_Users`
- `GG_Sales_Users`
- `GG_HelpDesk_Tier1`

## File Services and Access Control

Departmental SMB shares were created on DC01:

| Share | Authorized group | NTFS access |
|---|---|---|
| `\\DC01\IT` | GG_IT_Users | Modify |
| `\\DC01\HR` | GG_HR_Users | Modify |
| `\\DC01\Finance` | GG_Finance_Users | Modify |
| `\\DC01\Sales` | GG_Sales_Users | Modify |
| `\\DC01\Public` | Domain Users | Modify |

`SYSTEM` and `Administrators` retain Full Control. Departmental users receive
Modify instead of Full Control so they cannot change permissions or ownership.

## Group Policy

The following policies were implemented:

### Workstation Security

- Machine inactivity limit: 600 seconds.
- Guest account disabled.
- Linked to the Workstations OU.

### Domain Password Policy

- Password history: 5 passwords.
- Maximum password age: 90 days.
- Minimum password age: 1 day.
- Minimum password length: 10 characters.
- Complexity requirements enabled.
- Account lockout after 5 invalid attempts.
- Lockout duration and reset counter: 15 minutes.

### Department Drive Maps

Group Policy Preferences and item-level targeting automatically map:

| Group | Drive |
|---|---|
| Domain Users | `P:` → `\\DC01\Public` |
| GG_IT_Users | `I:` → `\\DC01\IT` |
| GG_HR_Users | `H:` → `\\DC01\HR` |
| GG_Finance_Users | `F:` → `\\DC01\Finance` |
| GG_Sales_Users | `S:` → `\\DC01\Sales` |

For example, an IT user receives only the Public and IT drives.

## DHCP and DNS

DC01 provides DHCP leases from `10.10.10.100` through `10.10.10.200`. Clients
receive `10.10.10.10` as their DNS server, allowing them to locate domain
services and resolve `cabreralab.test`.

The configuration was validated using:

```powershell
ipconfig /all
nslookup cabreralab.test
dcdiag /test:dns
Get-DhcpServerv4Scope
```

## Troubleshooting Scenario

### Client Could Not Resolve the Internal Domain

**Symptoms**

- `CLIENT01` could not resolve `cabreralab.test`.
- Domain resources were unavailable.

**Cause**

The client was configured to use a public DNS server. Public DNS servers do not
contain records for the private Active Directory domain.

**Resolution**

1. Restored automatic DNS configuration from DHCP.
2. Flushed the local DNS cache.
3. Released and renewed the DHCP lease.
4. Verified that the client received `10.10.10.10` as DNS.
5. Confirmed successful domain resolution.

```powershell
ipconfig /flushdns
ipconfig /release
ipconfig /renew
nslookup cabreralab.test
```

## Validation

The completed environment was validated by:

- Joining CLIENT01 to `cabreralab.test`.
- Signing in with a domain user.
- Confirming that Group Policy applied successfully.
- Verifying automatic drive mapping based on group membership.
- Confirming access to authorized shares.
- Confirming Access Denied for unauthorized departmental shares.
- Confirming a valid DHCP lease and internal DNS resolution.
- Running a successful domain controller DNS health check.

## Skills Demonstrated

- Windows Server administration
- Active Directory Domain Services
- DNS and DHCP
- Hyper-V virtualization
- TCP/IP configuration and troubleshooting
- Organizational units and identity administration
- Security groups and role-based access control
- SMB shares and NTFS permissions
- Group Policy Objects and Group Policy Preferences
- Windows domain joins
- PowerShell and command-line diagnostics
- Technical documentation

## Evidence

See the [`screenshots`](screenshots/) directory for selected implementation and
validation evidence.

## Author

**Carlos Cabrera**  
CompTIA A+ Certified | IT Support and Systems Administration

