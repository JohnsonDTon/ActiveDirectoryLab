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

Create the subscription. (this assumes you have C:\Temp folder). Run on WEC01
```powershell
@'
<Subscription xmlns="http://schemas.microsoft.com/2006/03/windows/events/subscription">
  <SubscriptionId>Account-Policy-Changes</SubscriptionId>
  <SubscriptionType>SourceInitiated</SubscriptionType>
  <Description>Account and group changes, audit policy changes, NTLM validation</Description>
  <Enabled>true</Enabled>
  <Uri>http://schemas.microsoft.com/wbem/wsman/1/windows/EventLog</Uri>
  <ConfigurationMode>MinLatency</ConfigurationMode>
  <Delivery Mode="Push">
    <Batching>
      <MaxItems>1</MaxItems>
      <MaxLatencyTime>30000</MaxLatencyTime>
    </Batching>
    <PushSettings>
      <Heartbeat Interval="3600000"/>
    </PushSettings>
  </Delivery>
  <Query><![CDATA[
    <QueryList>
      <Query Id="0">
        <Select Path="Security">*[System[(EventID=4720 or EventID=4722 or EventID=4724 or EventID=4725 or EventID=4726 or EventID=4732 or EventID=4733 or EventID=4719 or EventID=4776)]]</Select>
      </Query>
    </QueryList>
  ]]></Query>
  <ReadExistingEvents>false</ReadExistingEvents>
  <TransportName>HTTP</TransportName>
  <ContentFormat>RenderedText</ContentFormat>
  <Locale Language="en-US"/>
  <LogFile>ForwardedEvents</LogFile>
  <AllowedSourceNonDomainComputers></AllowedSourceNonDomainComputers>
  <AllowedSourceDomainComputers>O:NSG:NSD:(A;;GA;;;DC)(A;;GA;;;NS)</AllowedSourceDomainComputers>
</Subscription>
'@ | Set-Content -Path "C:\Temp\account-subscription.xml"
```
```powershell
wecutil cs C:\Temp\account-subscription.xml
```
