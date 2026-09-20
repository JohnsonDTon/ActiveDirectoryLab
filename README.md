# ActiveDirectoryLab
Personal Sandbox notes for my Active Directory Lab. Will write notes here on what I done, so in case I ever revisit these tasks on the job I have personal notes I can reference back to. </br>
Powershell-centric lab. Won't be using GUI to configure, only to verify configuration is implemented.<br>

This lab is in progress. I use Windows Server 2025 Standard Edition and Windows 11 Pro for end-users in this lab. <br>
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

Testing this NTFS format
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
