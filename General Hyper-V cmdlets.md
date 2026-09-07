Get info about the Hyper-V Host Machine.
```powershell
Get-VMHost
```
</br>


Get info about the VM switches configured on the Hyper-V Host Machine.
```powershell
Get-VMSwitch
```
</br>

Get CPU Usage and Processor Count of a VM. </br>
If CPUUSage says Low, no need to adjust. Change -Name to your environment
```powershell
Get-VM -Name "DC01" | Select-Object Name, CPUUsage, ProcessorCount
```
</br>
