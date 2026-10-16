Note: WSUS is being depreciated by Microsoft. But I'm going to run it on my on-perm lab for now and use Windows AutoPatch in my future Cloud lab. <br><br>

Run this on a fresh (and named) Windows Server VM that you'd making it to be your WSUS Server. <br>
Make sure the IP/DNS is configured <br>

Join WSUS01 to domain. Run on WSUS01.
```powershell
Add-Computer `
    -DomainName "adlab.test" `
    -Credential "ADLAB\Administrator" `
    -Restart
```

Move WSUS01 to correct OU. Run on DC.
```powershell
Move-ADObject `
    -Identity "CN=WSUS01,CN=Computers,DC=adlab,DC=test" `
    -TargetPath "OU=Member Servers,OU=Servers,DC=adlab,DC=test"
```

Create a second VHDX. Run on Hyper-V Host.
```powershell
New-VHD `
    -Path "E:\ActiveDirectoryLab\WSUS01\WSUS-Content.vhdx" `
    -SizeBytes 150GB `
    -Dynamic
```

Connect disk to VM.
```powershell
Add-VMHardDiskDrive `
    -VMName "WSUS01" `
    -Path "E:\ActiveDirectoryLab\WSUS01\WSUS-Content.vhdx"
```

Verify Disk is attached. Run on WSUS01
```powershell
Get-Disk
```

Initialize Disk <br>
New Disk from Get-Disk output is going to be number 1. Check Get-Disk just in case.
```powershell
Initialize-Disk -Number 1 -PartitionStyle GPT
```

Create partition.
```powershell
New-Partition `
    -DiskNumber 1 `
    -UseMaximumSize `
    -DriveLetter E
```

Format the volume
```powershell
Format-Volume `
    -DriveLetter E `
    -FileSystem NTFS `
    -NewFileSystemLabel "WSUS-Content"
```

Verify
```powershell
Get-Volume
```

Install the WSUS role
```powershell
Install-WindowsFeature `
    -Name UpdateServices `
    -IncludeManagementTools
```

Verify
```powershell
Get-Service -Name WsusService |
    Select-Object Name, Status, StartType
```

Create WSUS folder
```powershell
New-Item `
    -Path "E:\WSUS" `
    -ItemType Directory
```

Set default WSUS downloads to E:\WSUS
```powershell
& "C:\Program Files\Update Services\Tools\WsusUtil.exe" postinstall CONTENT_DIR="E:\WSUS"
```

Verify
```powershell
Get-ItemProperty `
    -Path "HKLM:\SOFTWARE\Microsoft\Update Services\Server\Setup" `
    -Name "ContentDir" |
    Select-Object ContentDir
```
```powershell
Get-ItemProperty `
    -Path "HKLM:\SOFTWARE\Microsoft\Update Services\Server\Setup" |
    Select-Object SqlServerName, SqlDatabaseName, ContentDir
```

Configure Initial Configurations
```text
Open Windows Server Update Services
Expand WSUS01
Options
WSUS Server Configuration Wizard

Before You Begin -> Next
Microsoft Update Improvement Program -> uncheck -> Next
Choose Upstream Server -> Synchronize from Microsoft Update -> Next
Specify Proxy Server -> leave unchecked -> Next
Connect to Upstream Server -> Start Connecting -> wait for it to finish -> Next
Choose Languages -> Download updates only in these languages: English -> Next
Choose Products -> uncheck everything, then check only "Windows 11" -> Next
Choose Classifications -> Critical Updates, Security Updates, Definition Updates -> Next
Configure Sync Schedule -> Synchronize manually -> Next
Finish -> uncheck "Begin initial synchronization"
```

Let WSUS know clients are configured by GPOs.
```text
Open Windows Server Update Services
Expand WSUS01
Options -> Computers
Select: Use Group Policy or registry settings on computers
OK
```

Create a computer group in WSUS
```text
Open Windows Server Update Services
Expand WSUS01
Computers -> All Computers
Add Computer Group
Name: Workstations

```

Sync updates
```text
Right click Synchronizations -> Synchronize Now
```

Set up automatic approval <br>
Without this means you'd have to manually approve each one
```text
Options -> Automatic Approvals
New Rule...
Check: When an update is in a specific classification
    Select: Critical Updates, Security Updates
Check: Approve the update for a specific computer group
    Select: Workstations
Name: Auto-approve Critical and Security
OK -> check "Enable rule" -> Run rule
```

Point clients to WSUS. Run on DC
```powershell
Set-GPRegistryValue `
    -Name "Workstations - Windows Update" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" `
    -ValueName "WUServer" -Type String -Value "http://WSUS01.adlab.test:8530"
```
```powershell
Set-GPRegistryValue `
    -Name "Workstations - Windows Update" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" `
    -ValueName "WUStatusServer" -Type String -Value "http://WSUS01.adlab.test:8530"
```
```powershell
Set-GPRegistryValue `
    -Name "Workstations - Windows Update" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" `
    -ValueName "UseWUServer" -Type DWord -Value 1
```

Add group targetting to Workstations
```powershell
Set-GPRegistryValue `
    -Name "Workstations - Windows Update" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" `
    -ValueName "TargetGroupEnabled" -Type DWord -Value 1
```
```powershell
Set-GPRegistryValue `
    -Name "Workstations - Windows Update" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" `
    -ValueName "TargetGroup" -Type String -Value "Workstations"
```
