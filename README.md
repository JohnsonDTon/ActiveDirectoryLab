# ActiveDirectoryLab
Personal Sandbox notes for my Active Directory Lab. Will write notes here on what I done, so in case I ever revisit these tasks on the job I have personal notes I can reference back to. </br>
Will use powershell and avoid doing things via GUI as much as possible. </br>

This lab is in progress. I use Windows Server 2025 Standard Edition and Windows 11 Pro for end-users in this lab.
Workgroup client name will be "EndUser" during Client VM creations.

### Network Layout
Domain: adlab.test / ADLAB <br>
Network Range: 192.168.100.0/24 <br>
DHCP Scope: 192.168.100.50 - 192.168.100.254

| System | Role | IP Address |
|---|---|---|
| DC1 | Primary Domain Controller | 192.168.100.10 |
| DC2 | Secondary Domain Controller | 192.168.100.11 |
| FILE01 | File Server | 192.168.100.20 |
| WIN11-01 | Windows 11 Client | 192.168.100.101 |
| WIN11-02 | Windows 11 Client | 192.168.100.102 |
| WIN11-03 | Windows 11 Client | 192.168.100.103 |

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
