# ActiveDirectoryLab
Personal Sandbox notes for my Active Directory Lab. Will write notes here on what I done, so in case I ever revisit these tasks on the job I have personal notes I can reference back to. </br>
Will use powershell and avoid doing things via GUI as much as possible. </br>

This lab is in progress. I use Windows Server 2025 Standard Edition and Windows 11 Pro for end-users in this lab.

### Planned Network Layout
Network Range: 192.168.100.0/24
DHCP Scope: 192.168.100.20 - 192.168.100.254

| System | Role | IP Address |
|---|---|---|
| DC1 | Primary Domain Controller | 192.168.100.10 |
| DC2 | Secondary Domain Controller | 192.168.100.11 |
| CLIENTx | Domain-Joined Computer(s) | DHCP |
