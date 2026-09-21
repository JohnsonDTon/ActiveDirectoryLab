Check Group Policies in XML. <br>
This assumes you have a C:\Temp folder, otherwise do a path thats for your enviornment. <br>
Run on your DC.

```powershell
Get-GPO -Name "Default Domain Policy" |
    Get-GPOReport -ReportType Html -Path "C:\Temp\DefaultDomainPolicy.html"
```

```powershell
Start-Process "C:\Temp\DefaultDomainPolicy.html"
```

<br> <br>
Check applied GPOs on a windows client. Run in admin powershell, Windows Client machine.
```powershell
gpupdate /force
gpresult /r /scope computer
```
