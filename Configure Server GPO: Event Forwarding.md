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

Set the subscription manager to point to WEC01
```powershell
Set-GPRegistryValue `
    -Name "Servers - Event Forwarding" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager" `
    -ValueName "1" `
    -Type String `
    -Value "Server=http://WEC01.adlab.test:5985/wsman/SubscriptionManager/WEC,Refresh=60"
```

Let the forwarder read the Security log
```text
Group Policy Management -> Servers - Event Forwarding -> Edit
Computer Configuration -> Preferences -> Control Panel Settings -> Local Users and Groups
Right click -> New -> Local Group
    Action: Update
    Group name: Event Log Readers (built-in)
    Members -> Add...
        Name: NT AUTHORITY\NETWORK SERVICE     <- type this directly, don't click "..."
    OK -> Apply -> OK
```
