[HyperV Server](https://www.microsoft.com/en-us/evalcenter/evaluate-hyper-v-server-2019) is a free Hypervisor Host from Microsoft. It is a console only, core version of the same HyperV role that can be added to Windows Server. I am glad to share my running HyperV Server 2016 (and 2019) notes with you.

## Notes
1. Use PowerShell for everything. If you "must" use commandline or a GUI, see guidance at  [http://www.informit.com/articles/article.aspx?p=1245848&seqNum=3](http://www.informit.com/articles/article.aspx?p=1245848&seqNum=3) and
[https://blogs.msmvps.com/ad/blog/2008/09/18/admin-s-guide-to-server-core-commands/](https://blogs.msmvps.com/ad/blog/2008/09/18/admin-s-guide-to-server-core-commands/) and
[Manage a Server Core server](https://docs.microsoft.com/en-us/windows-server/administration/server-core/server-core-manage) and
[Windows Server 2012 Server Core - Part 5: Tools](https://4sysops.com/archives/windows-server-2012-server-core-part-5-tools/) also
[Manage Hyper-V Server 2016 or 2019 in a workgroup using Windows 10 Hyper-V Manager… in 13 Simple Steps](https://theserverplaypen.wordpress.com/2018/01/23/manage-hyper-v-server-2012r22016-in-a-workgroup-with-windows-10-hyper-v-manager-in-13-simple-steps/) as well as
[Manage Windows Server 2019 with Admin Center, PowerShell Core, and sconfig](https://4sysops.com/archives/manage-windows-server-2019-with-admin-center-powershell-core-and-sconfig/)
2. From the PowerShell prompt (or commandline), `notepad *filename*` still works.
3. You do not need to install Hyper-V in a domain (unless you have a large domain with many physical hosts, please consider that it may be more sustainable and possibly more secure to not install Hyper-V hosts in a domain) [https://timothygruber.com/hyper-v-2/remotely-managing-hyper-v-server-in-a-workgroup-or-non-domain/](https://timothygruber.com/hyper-v-2/remotely-managing-hyper-v-server-in-a-workgroup-or-non-domain/)
Also, the powershell cmdlet "Enable-PSRemoting -SkipNetworkProfileCheck" will sidestep an issue concerning Public network types if you have HyperV installed on the Windows 10 PC.
4. The best way to protect from catastrophic disk failure is to use RAID1, disk mirrors. Creating a disk mirror from PowerShell prompt (or commandline);
```
diskpart; (please reference diskpart help for the actual command syntax, the following is a pseudo code guide of the workflow)
"clean" the mirror disks
"convert dynamic" each disk
"create volume mirror disk=x,y"
"select the volume and assign letter"
```
5. Change the pagefile location to be stored somewhere other than the OS disk (I typically use an SSD for the OS and large spindle disks for data);
Verify current pagefile location
`wmic pagefile list /format:list`
Change pagefile location
`wmic pagefileset create name="D:\pagefile.sys"`
`wmic pagefileset where name="D:\\pagefile.sys" set InitialSize=2048,MaximumSize=2048`
Delete the old pagefile
`wmic pagefileset where name="C:\\pagefile.sys" delete`
6. Renaming the Administrator account name should already be part of standard security practices
`wmic UserAccount where Name=”Administrator” call Rename Name=”new-unique-name”`
7. Using free Hyper-V Server for off-site replicas is not obvious and can save your organization licensing costs. See guidance at [https://medium.com/@pbengert/setup-2-hyper-v-2016-servers-enable-hyper-v-replica-with-self-created-certificates-and-connect-to-fceef21c8b8e](https://medium.com/@pbengert/setup-2-hyper-v-2016-servers-enable-hyper-v-replica-with-self-created-certificates-and-connect-to-fceef21c8b8e) If choosing to purchase SSL certificates, consider long term impacts. For example, in a domain, certificates are used for many, many things and are a part of the Active Directory infrastructure. In small environments, self-signed certificates may meet the need more flexibly and with lower long term cost. For example, if you are the only admin, being able to reissue certs and deploy them in an hour or by using a script automated workflow is much more flexible than going through a vendor portal.
During setup of the offsite Hyper-V server intended for use as a replica holder you may also need to change the computer's FQDN; [http://www.allthingstechie.net/2015/04/use-powershell-to-change-hosts-fqdn.html](http://www.allthingstechie.net/2015/04/use-powershell-to-change-hosts-fqdn.html)
```
$DNSSuffix = "abc.com"
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\" -Name Domain -Value $DNSSuffix
Set-DnsClientGlobalSetting -SuffixSearchList $DNSSuffix
```
*Note:* The offsite hardware does not have to be capable of running live VM guests unless it will actually be used for a disaster recovery site or live migrations or fail over. Using 8-16GB of RAM for the HyperV server is more than enough RAM to receive the replicas and export VMs to large USB disk storage for transport to the office or disaster recovery site. Also, Hyper-V server will allow storing replicas straight to USB disk directly. So, the replica storage does not need to be a fast (expensive) disk either.
8. I recommend [DUO.com](https://duo.com/) for Two Factor Authentication (2FA) even if you are not exposing the Hyper-V host to the Internet (or even if you are using a VPN to offsite). Adding 2Fa is an essential step in protecting offsite hosts.
9. Use "w32tm" to specify a reliable time source, do not use one of the Hyper-V hosted VM guests as the time source. That also means disabling the HyperV provided guest service for time on the VM guests, particularly for DC VM guests.
```
w32tm /query /status (Will show current time sync source status. On HV host it should be external. On domain, it should be the DC with the PDC emulator role, typically the first DC.)
net stop w32tm
w32tm /configure /syncfromflags:manual /manualpeerlist:"0.us.pool.ntp.org,0x1 1.us.pool.ntp.org,0x1 2.us.pool.ntp.org,0x1 3.us.pool.ntp.org,0x1"
net start w32time
w32tm /configure /reliable:yes /update
w32tm /resync
```
*Note:* Domain Controllers must be "the" time source for the domain clients. Basically, set the first DC similar to the code example shown above, then all other DCs get `w32tm /config /syncfromflags:domhier /update` and they will always default to the first DC for time.
10. Installing updates options;
Commandline `sconfig`
Powershell `Install-Module PSWindowsUpdate`, then `Install-WindowsUpdates` (Allows selection of specific optional updates from list of available updates. This ends up being important when updated chipset drivers are available. On W2012R2, manual download and Install Windows Management Framework 5.1 feature first, reboot required.)
11. The Data Backup Rule of 3.
    Backup task to locally mounted USB 3 dedicated drive.
    HyperV replication partner offsite.
    Rclone or Restic data only task to offsite.
12. Uninstalling from the command line follows this pseudo-script;
```
wmic
product get name
product where name="name of program" call uninstall
exit
```
13. CentOS guest info [https://www.altaro.com/hyper-v/centos-linux-hyper-v/](https://www.altaro.com/hyper-v/centos-linux-hyper-v/)
14. HyperV Server Hardware info [https://www.altaro.com/hyper-v/hardware-tweaks-hyper-v-performance/](https://www.altaro.com/hyper-v/hardware-tweaks-hyper-v-performance/)
