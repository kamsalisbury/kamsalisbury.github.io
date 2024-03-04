---
layout: post
title: "On Premise to Azure Rclone Backups"
date: 2020-10-16
categories: Code Backup
---

[Rclone v1.53.1](https://rclone.org) supports encrypting rclone's configuration, adding additional security to securing off site backups. Using Azure Automation reduces complexity and simplifies job monitoring.

1. The Rclone website explains installation and configuration of the utility but it is not obvious that during configuration you can choose to protect the configuration with a password
2. [Azure Automation](https://docs.microsoft.com/en-us/azure/automation/automation-intro) provides the service to store the rclone backup script (in Azure speak, a runbook)
3. In the Azure Automation object, Variables blade, add a variable. Name = RCLONE_CONFIG_PASS, Type = Encrypted, Value = 'password you set in rclone config'
4. Running the rclone backup script on physical hosts and VMs on premises means using an [Azure Hybrid Worker](https://docs.microsoft.com/en-us/azure/automation/automation-hybrid-runbook-worker). It is not obvious that your will need to onboard a hybrid worker for each non-domain joined host/VM because the script will run in the same security context of the hybrid worker, on that specific host/VM. Azure supports creating multiple hybrid workers and assigning one each to a hybrid worker group
5. In the PowerShell runbook, add `$env:RCLONE_CONFIG_PASS = (Get-AutomationVariable -Name 'RCLONE_CONFIG_PASS')` to set the rclone config password for each script run. This step avoids exposing the password in your runbook code, enables a single place to update the password for multiple runbooks, enables having multiple rclone config passwords by assigning one to each differently named variable, avoids leaving the password exposed in a script or in local host variables after the script has ran
6. Rclone will use slightly different flags (parameters) depending on the storage being used. Here is an example `Invoke-Expression -Command "c:\path\to\rclone.exe sync E:\Data OFFSITE:BUCKET/Folder --config c:\path\to\.config\rclone\rclone.conf --skip-links --fast-list --transfers 32 --log-file c:\path\to\optional\rclone.log --log-level ERROR"`
7. Link the runbook to an automation schedule, then monitor for failures using Azure Log Analytics and Azure Alerts
