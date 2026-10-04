Note: WSUS is being depreciated by Microsoft. But I'm going to run it on my on-perm lab for now and use Windows AutoPatch in my future Cloud lab. <br>
This setup is on hold. WSUS is not up-to-date without internet so have to configure internet on my lab before I continue with WSUS <br>  <br>

Run this on a fresh (and named) Windows Server VM that you'd making it to be your WSUS Server. <br>
Reference: https://github.com/JohnsonDTon/ActiveDirectoryLab/blob/main/3.%20Create%20VM.md <br>
Make sure the IP/DNS is configured <br>
Reference: https://github.com/JohnsonDTon/ActiveDirectoryLab/blob/main/4.%20Configure%20IP.md

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

<br>

Configure Update Source and Proxy Server
```text
Open Windows Servers Update Services
Expand WSUS01
Click Options
Click Update Source and Proxy Server

Select:
Synchronize from Microsoft Update
Do not select "This server is a replica of the upstream server"

Proxy: Leave Use a proxy server when synchronizing unchecked.
Click OK.
```

Sync Updates
```text
Open Windows Servers Update Services
Expand WSUS01
Right click Synchronizations -> Synchronize Now
```
