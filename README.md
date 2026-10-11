# ActiveDirectoryLab
Personal Sandbox notes for my Active Directory Lab. Will write notes here on what I done, so in case I ever revisit these tasks on the job I have personal notes I can reference back to. </br>

I use Windows Server 2025 Standard Edition and Windows 11 Pro for end-users in this lab. <br>
Workgroup client name will be "EndUser" and passwordless during Client VM creations.

### Network Layout
Domain: adlab.test / ADLAB <br>
Network Range: 192.168.100.0/24 <br>
DHCP Scope: 192.168.100.50 - 192.168.100.254

| System | Role | IP Address |
|---|---|---|
| DC1 | Primary Domain Controller (w/DNS) | 192.168.100.10 |
| DC2 | Secondary Domain Controller (w/DNS) | 192.168.100.11 |
| FILE01 | File Server | 192.168.100.20 |
| WIN11-01 | Windows 11 Client | 192.168.100.101 |
| WIN11-02 | Windows 11 Client | 192.168.100.102 |

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
    └── Domain Local
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
| Account          | Full Name                 | Account Type     | OU             |
| ---------------- | ------------------------- | ---------------- | -------------- |
| `alice.johnson`  | Alice Marie Johnson       | Standard User    | IT             |
| `3alice.johnson` | Alice Marie Johnson Admin | Privileged Admin | IT             |
| `bob.smith`      | Bob Thomas Smith          | Standard User    | HR             |
| `carol.davis`    | Carol Davis               | Standard User    | Finance        |
| `david.wilson`   | David James Wilson        | Standard User    | Standard Users |

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
