The [Open Threat Exchange by Alienvault](https://otx.alienvault.com/) was purchased by AT&T and continues to go through updates, services such as the endpoint agent failing to be seen by the API, etc.

1. Before uninstalling the endpoint agent, osqueryd, consider just stoping and disabling the daemon.
Linux:
```
sudo systemctl stop osqueryd
sudo systemctl disable osqueryd
```
Windows PS:
```
Stop-Service osqueryd
Set-Service osqueryd -StartupType Disabled
```
2. Really want to uninstall? First stop the service, then...
Linux:
```
sudo rm -rf /etc/osquery
sudo rm -rf /usr/share/osquery
sudo rm -rf /var/log/osquery
sudo rm -rf /usr/lib/osquery
sudo rm -rf /usr/bin/osqueryctl
sudo rm -rf /usr/bin/osqueryd
sudo rm -rf /usr/bin/osqueryi
```
Windows PS:
```
Remove-Service osqueryd
$uninstall64 = Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" | foreach { Get-ItemProperty $_.PSPath } | ? { $_ -match "AlienVault Agent" } | select UninstallString
$uninstall64 = $uninstall64.UninstallString -Replace "msiexec.exe","" -Replace "/I","" -Replace "/X",""
$uninstall64 = $uninstall64.Trim()
start-process "msiexec.exe" -arg "/X $uninstall64 /qb" -Wait
```
