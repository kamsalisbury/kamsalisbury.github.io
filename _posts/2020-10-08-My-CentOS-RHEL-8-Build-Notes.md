I use the CentOS Linux platform for the majority of my computing tasks. I am not against other Linux distributions. The target hardware is HyperV Server 2016 on Intel hardware, 4GB RAM.
Update: Redhat has ended their support of resources for the CentOS program. CentOS will not receive updates after 2021. [Redhat has added a Redhat Enterprise Linux subscription benefit to their developers program](https://developers.redhat.com/articles/getting-red-hat-developer-subscription-what-rhel-users-need-know) which basically allows me to convert my few CentOS VMs to RHEL. I have updated these notes to reflect any RHEL specific changes.

HyperV specific VM settings: Secure Boot enabled, Microsoft UEFI Certificate authority. 4096MB RAM install and virtualization guest tools, then after reboot 2048 with dynamic RAM minimum 512MB. 2 vCPUs. All integration services. Checkpoints disabled.

1. Minimal install, set hostname and static networking
2. Register this system `subscription-manager register` then `subscription-manager attach`
3. Install core utilities `yum install nano wget cockpit rsyslog rsyslog-doc unzip podman buildah -y`
4. Enable Cockpit (Compariable to Windows Admin Center) `systemctl enable --now cockpit.socket`
5. Enable firewall rules for cockpit and other services being installed
```
sudo firewall-cmd --add-service=cockpit --permanent
sudo firewall-cmd --add-service=syslog --permanent
sudo firewall-cmd --reload
```
6. Configure /etc/ssh/sshd_config to disable remote root login `nano /etc/ssh/sshd_config "PermitRootLogin no"`
7. Configure client workstation for ssh key logins `ssh-keygen -t rsa -b 4096` (Implement later `PasswordAuthentication no` ` ChallengeResponseAuthentication no ` `UsePAM no`)
7. Install fail2ban 
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
ARCH=$( /bin/arch )
subscription-manager repos --enable "codeready-builder-for-rhel-8-${ARCH}-rpms"
sudo yum install fail2ban -y
```

Configure to protect ssh and log to syslog.
```
nano /etc/fail2ban/jail.local
[DEFAULT]
ignoreip = yourIP_or_subnet
bantime  = 21600
findtime  = 300
maxretry = 3
banaction = iptables-multiport
backend = systemd
logtarget = SYSLOG
[sshd]
enabled = true
```
8. Configure rsyslog to recieve firewall and switch logs `nano /etc/rsyslog.conf` then `systemctl restart rsyslog`
9. Configure logrotate for 53 weeks of logs `nano /etc/logrotate.conf`
10. [Install PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-7)
11. Install and onboard Azure Log Analytics agent then `ln -s /var/opt/microsoft/omsagent/log/omsagent.log /var/log/omsagent.log`
(It took less than an hour for the host to appear in Azure, then two hours more to become available in Azure Automation Update Management)
12. Implement ssh key authentication and then install and configure [Duo](https://duo.com/docs/duounix) or [other 2FA](https://github.com/CERN-CERT/pam_2fa)
13. See [my notes about running containers via podman.](https://kamsalisbury.github.io/My-CentOS-8-Podman-Container-Notes/)
