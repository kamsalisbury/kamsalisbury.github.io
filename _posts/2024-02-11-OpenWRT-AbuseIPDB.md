---
layout: post
title: "OpenWRT-AbuseIPDB"
date: 2024-02-11
categories: Code Arm Shell
---

A purposely simplified script to report port scans to [AbuseIPDB](https://www.abuseipdb.com/user/26499).
Emphasizing efficiency, everything happens in RAM.
No temporary files to cause degraded flash storage.
Tested on [GL.iNet](https://www.gl-inet.com/) Brume2, an Arm processor Ash Shell based firewall.

Script source code and installation instructions at the repository, [OpenWRT-AbuseIPDB](https://github.com/kamsalisbury/OpenWRT-AbuseIPDB).

## The Problem To Solve
Contribute to the AbueIPDB Project by reporting observed [Port Scans](https://www.fortinet.com/resources/cyberglossary/what-is-port-scan). 

### Contraints
The router-firewall device design emphasizes efficient operation.

Arm CPU, RAM and flash storage are limited. 

### Solution
Create a shell script suitable for cron execution that parses log output for port scan dropped packets, then submits the required data points to AbuseIPDB using an API.

### Results

<img src="https://kamsalisbury.github.io/images/OpenWRT-AbuseIPDB_Firewall-Config.png" alt="Example firewall log configuration" />
<img src="https://kamsalisbury.github.io/images/OpenWRT-AbuseIPDB_Firewall-Log.png" alt="Example firewall log output" />
<img src="https://kamsalisbury.github.io/images/OpenWRT-AbuseIPDB_Submitted.png" alt="Example AbueIPDB status showing reported IPs" />
