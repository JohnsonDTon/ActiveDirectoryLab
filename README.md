# ActiveDirectoryLab
Personal Sandbox notes for my Active Directory Lab. Will write notes here on what I done, so in case I ever revisit these tasks on the job I have personal notes I can reference back to. </br>

## Summary
Active Directory Home Lab
- Built a 9-VM Hyper-V environment primarily through PowerShell, including redundant Windows Server 2025 domain controllers with DNS and DHCP, a file server, WSUS, event collector and backup servers, an OPNsense firewall gateway, and domain-joined Windows 11 workstations; documented in a 40+ step runbook 
- Designed an Active Directory domain with a structured OU hierarchy, AGDLP security groups, SMB/NTFS permissions with Access-Based Enumeration, and separate standard and privileged admin accounts to enforce least privilege 
- Deployed 19 GPOs across workstations, users and servers, including password and lockout policy, UAC, Windows Firewall, Windows LAPS for unique rotating local admin passwords, Microsoft Defender with ASR rules, and BitLocker with recovery keys escrowed to Active Directory
- Configured and tested high availability for core services: took DC01 offline and confirmed domain logons and DNS still worked through DC02, and load-balanced DHCP failover across both domain controllers with a client receiving its lease from the partner server
- Enabled Advanced Audit Policy and PowerShell Script Block/Module logging via Group Policy; forwards workstation logon (4624/4625) and script block (4104) events to a central Windows Event Collector server

Backup & Disaster Recovery Lab (Veeam)
- Built a workgroup (non-domain-joined) Veeam backup server to keep backups isolated from AD; protected domain controllers and member servers with agent-based, encrypted daily jobs to a ReFS repository (64 KB clusters, Fast Clone), with application-aware (VSS) processing so Active Directory is captured consistently
- Applied least privilege: separate service accounts for DC and member-server tiers, local admin via AD security group and GPO, interactive/RDP logon denied by GPO, and firewall rules scoped to the backup server's IP
- Validated recovery with a file restore, deleted-user recovery through both the AD Recycle Bin and Veeam Explorer for AD, and a full-disk restore booted on an isolated Hyper-V network; documented procedures in a runbook


I use Windows Server 2025 Standard Edition and Windows 11 Pro for end-users in this lab. <br>
Workgroup client name will be "EndUser" and passwordless during Client VM creations.

### Network Layout
Domain: adlab.test / ADLAB <br>
Network Range: 192.168.100.0/24 <br>
DHCP Scope: 192.168.100.50 - 192.168.100.254

| System | Role | IP Address |
|---|---|---|
| OPNSENSE01 | Lab Firewall / Default Gateway | 192.168.100.1 |
| BACKUP01 | Backup Server (Veeam) - workgroup, not domain-joined | 192.168.100.5 |
| DC01 | Primary Domain Controller (w/DNS + DHCP) | 192.168.100.10 |
| DC02 | Secondary Domain Controller (w/DNS + DHCP) | 192.168.100.11 |
| FILE01 | File Server | 192.168.100.20 |
| WSUS01 | Windows Update Server (WSUS) | 192.168.100.21 |
| WEC01 | Windows Event Collector (WEC) | 192.168.100.22 |
| WIN11-01 | Windows 11 Client | DHCP address |
| WIN11-02 | Windows 11 Client | DHCP address |

### OU Structure

```text
ADLAB.TEST
│
├── User Accounts
│   ├── IT
│   ├── HR
│   ├── Finance
│   └── Standard Users
│
├── Workstations
│   ├── IT
│   ├── HR
│   ├── Finance
│   └── Standard Users
│
├── Servers
│   ├── Member Servers
│   └── File Servers
│
└── Service Accounts
```

### FILE01 Directory Structure
``` text
├── C:\                  Windows Server OS
├── D:\                  DVD Drive
└── E:\                  FileServerData
    └── Shares
        ├── IT
        ├── HR
        ├── Finance
        └── Public
```

### Security Group
```text
ADLAB.TEST
│
└── Security Groups
    │
    ├── Global
    │   ├── GG-All-Employees
    │   ├── GG-IT-Users
    │   ├── GG-HR-Users
    │   └── GG-Finance-Users
    │
    └── Domain Local (most of these are not used/configured)
        ├── DL-FILE01-IT-Read
        ├── DL-FILE01-IT-Modify
        ├── DL-FILE01-IT-Full
        ├── DL-FILE01-HR-Read
        ├── DL-FILE01-HR-Modify
        ├── DL-FILE01-HR-Full
        ├── DL-FILE01-Finance-Read
        ├── DL-FILE01-Finance-Modify
        ├── DL-FILE01-Finance-Full
        ├── DL-FILE01-Public-Read
        ├── DL-FILE01-Public-Modify
        └── DL-FILE01-Public-Full
    └── Backup-Server-Admins   (local admin on member servers, for the Veeam service account)
```

### Testing this NTFS format
```text
E:\Shares\IT
    ├── Administrators      Full Control
    ├── SYSTEM              Full Control
    └── DL-FILE01-IT-Modify Modify

E:\Shares\HR
    ├── Administrators      Full Control
    ├── SYSTEM              Full Control
    └── DL-FILE01-HR-Modify Modify

E:\Shares\Finance
    ├── Administrators      Full Control
    ├── SYSTEM              Full Control
    └── DL-FILE01-Finance-Modify Modify

E:\Shares\Public
    ├── Administrators      Full Control
    ├── SYSTEM              Full Control
    └── DL-FILE01-Public-Read Read
```

### Access Control List Design for File Server
```text
IT
SMB  → Authenticated Users → Full Control
NTFS → DL-FILE01-IT-Modify → Modify

HR
SMB  → Authenticated Users → Full Control
NTFS → DL-FILE01-HR-Modify → Modify

Finance
SMB  → Authenticated Users → Full Control
NTFS → DL-FILE01-Finance-Modify → Modify

Public
SMB  → Authenticated Users → Full Control
NTFS → DL-FILE01-Public-Read → Read & Execute
```

### User Accounts
```text
| Account          | Full Name                 | Account Type     | OU               |
| ---------------- | ------------------------- | ---------------- | ---------------- |
| `alice.johnson`  | Alice Marie Johnson       | Standard User    | IT               |
| `3alice.johnson` | Alice Marie Johnson Admin | Privileged Admin | IT               |
| `bob.smith`      | Bob Thomas Smith          | Standard User    | HR               |
| `carol.davis`    | Carol Davis               | Standard User    | Finance          |
| `david.wilson`   | David James Wilson        | Standard User    | Standard Users   |
| `svc-backup-srv` | Veeam member servers      | Service Account  | Service Accounts |
| `svc-backup-dc`  | Veeam domain controllers  | Service Account  | Service Accounts |

```

### GPO Architecture
```text
DEFAULT DOMAIN POLICY
│
├── Password Policy
│   ├── Minimum length = 12
│   ├── History = 24
│   ├── Maximum age = 90 days
│   └── Complexity = Enabled
│
└── Account Lockout
    ├── Threshold = 5
    ├── Duration = 15 min
    └── Observation = 15 min
```

## Users GPO
```text
User Accounts OU
│
├── Users - Screen Saver
│   ├── Screen saver enabled
│   ├── Password protected
│   └── Timeout = 900 sec
│
└── Users - Network Drives
    ├── P: → \\FILE01\Public
    ├── I: → \\FILE01\IT
    ├── H: → \\FILE01\HR
    └── F: → \\FILE01\Finance
```

## Workstations GPO
```text
Workstations OU
│
├── Workstations - Security Baseline
│   └── UAC
│
├── Workstations - Firewall
│   ├── Firewall enabled
│   ├── Inbound = Block
│   ├── Outbound = Allow
│   ├── Firewall logging
│   └── Firewall rules
│       ├── Allow ICMPv4 Echo Request
│       └── Allow RDP - TCP 3389
│
├── Workstations - Screen Lock
│   └── Machine inactivity limit = 900 sec
│
├── Workstations - Local Administrators
│   └── GG-IT-Admins
│       └── Local Administrators group
│
├── Workstations - Windows LAPS
│   └── Local Administrator password management
│
├── Workstations - Windows Defender
│   ├── Real-time Protection
│   ├── Behavior Monitoring
│   ├── IOAV Protection
│   ├── Cloud-Delivered Protection
│   ├── Sample Submission
│   ├── PUA Protection
│   ├── Cloud Block Level
│   ├── Removable Drive Scanning
│   ├── Network File Scanning
│   └── Scheduled Quick Scan
│
├── Workstations - Defender ASR
│   ├── Attack Surface Reduction enabled
│   ├── Block credential stealing from LSASS
│   ├── Block abuse of exploited vulnerable signed drivers
│   └── Block persistence through WMI event subscription
│
├── Workstations - BitLocker
│   └── Operating System Drives
│       ├── Recovery keys saved to AD DS
│       └── BitLocker not enabled until backup succeeds
│
├── Workstations - Windows Logging
│   ├── Security log = 128 MB
│   ├── System log = 64 MB
│   ├── Application log = 64 MB
│   └── Retention = Overwrite events as needed
│
├── Workstations - Advanced Auditing
│   ├── Account Logon
│   │   └── Credential Validation
│   ├── Logon/Logoff
│   │   ├── Logon
│   │   ├── Logoff
│   │   ├── Account Lockout
│   │   └── Special Logon
│   ├── Detailed Tracking
│   │   └── Process Creation
│   ├── Account Management
│   │   ├── User Account Management
│   │   ├── Security Group Management
│   │   └── Computer Account Management
│   └── Policy Change
│       └── Audit Policy Change
│
├── Workstations - PowerShell Logging
│   ├── Script Block Logging
│   ├── Module Logging
│   └── PowerShell Transcription
│
├── Workstations - RDP
│   ├── RDP enabled
│   └── Network Level Authentication (NLA) required
│
├── Workstations - Windows Update
│   ├── Configure Automatic Updates
│   │   ├── Auto download and schedule the install
│   │   └── Install daily at 03:00
│   ├── Active hours = 08:00 - 18:00
│   ├── Target release version = Windows 11 25H2
│   ├── Intranet update service = http://WSUS01.adlab.test:8530
│   ├── Client-side targeting = Workstations
│   └── Do not connect to Windows Update Internet locations
│
└── Workstations - Event Forwarding
    ├── Subscription manager = WEC01.adlab.test:5985
    ├── Windows Remote Management service = Automatic
    └── Network Service added to Event Log Readers (Group Policy Preferences)
```

### Servers GPO
```text
Servers OU
│
└── Member Servers - Backup Access
    └── ADLAB\Backup-Server-Admins added to local Administrators (GPP)

Servers OU and Domain Controllers OU
│
├── Service Accounts - Deny Logon
│   ├── Deny log on locally: svc-backup-srv, svc-backup-dc
│   └── Deny log on through Remote Desktop Services: svc-backup-srv, svc-backup-dc
│
└── Servers - Veeam Firewall
    ├── Allow TCP 135, 445 from BACKUP01
    ├── Allow TCP 49152-65535 from BACKUP01
    └── Allow TCP 6160, 6162, 10006 from BACKUP01
```

### Backup and Disaster Recovery
```text
BACKUP01 (workgroup, 192.168.100.5)
├── Veeam Backup & Replication, Community Edition
├── Repository: ReFS, 64 KB clusters (fast cloning), E:\Backups
└── Encrypted backup files and encrypted configuration backup

Protected servers (agent-based)
├── DC01   → job: Lab Servers - DCs (application-aware)
├── FILE01 → job: Lab Servers - Members
├── WEC01  → job: Lab Servers - Members
└── WSUS01 → job: Lab Servers - WSUS (content drive excluded)
```

| Account | Scope |
|---|---|
| `svc-backup-srv` | Local admin on FILE01, WSUS01, WEC01 (via GPO) |
| `svc-backup-dc`  | Administrators on DC01 only |

| Restore test | Result |
|---|---|
| Single file on FILE01 | Passed |
| Deleted AD user (AD Recycle Bin) | Passed |
| Deleted AD user (Veeam Explorer for AD) | Passed |
| WEC01 disk export, booted on isolated switch | Passed |

Lab limitations: backups share the host's physical drive (no off-host copy), and svc-backup-dc has Administrators rights on DC01. OPNsense is exported manually, since an agent can't run on it.
