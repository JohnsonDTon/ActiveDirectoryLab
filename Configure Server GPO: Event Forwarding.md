Create GPO and link to OU.
```powershell
New-GPO `
    -Name "Servers - Event Forwarding" `
    -Comment "Forwards server events to WEC01"

New-GPLink `
    -Name "Servers - Event Forwarding" `
    -Target "OU=Servers,DC=adlab,DC=test" `
    -LinkEnabled Yes
```

