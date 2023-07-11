---
layout: post
title: How To Configure NXLog to Receive Consumer FiOS Router Logs
---

Log aggregators are an important part of cybersecurity management, compliance and monitoring. The powerful utility [NXLOG Community Edition](https://nxlog.co/products/nxlog-community-edition) is an excellent solution for receiving syslog messages from 1+ hosts.

While the NXLog documentation is well written, the exact configuration needed for a common FiOS router was not obvious.

Optionally, the collected logs can be post-collection processed by another script or service, etc. (For example, I use Azure Log Analytics)

## I have shared my configuration below so that it may help others

```
Panic Soft
#NoFreeOnExit TRUE

define ROOT     C:\Program Files (x86)\nxlog
define CERTDIR  %ROOT%\cert
define CONFDIR  %ROOT%\conf
define LOGDIR   %ROOT%\data
define LOGFILE  %LOGDIR%\nxlog.log
LogFile %LOGFILE%

Moduledir %ROOT%\modules
CacheDir  %ROOT%\data
Pidfile   %ROOT%\data\nxlog.pid
SpoolDir  %ROOT%\data

<Extension _syslog>
    Module      xm_syslog
</Extension>

<Input udp>
    Module  im_udp
    Host    0.0.0.0
    Port    514
    Exec    parse_syslog();
</Input>

<Output file>
    Module  om_file
    Exec to_syslog_bsd();
    File 'E:\path\to\log\file\syslog.txt'
    # Rotate the syslog collector file
    <Schedule>
        When    @weekly
        Exec    if file_exists('E:\path\to\log\file\syslog.txt') file_cycle('E:\path\to\log\file\syslog.txt', 52);
	Exec	file->reopen();
    </Schedule>
</Output>

<Route syslog_to_file>
    Path    udp => file
</Route>

<Extension _charconv>
    Module      xm_charconv
    AutodetectCharsets iso8859-2, utf-8, utf-16, utf-32
</Extension>

<Extension _exec>
    Module      xm_exec
</Extension>

<Extension _fileop>
    Module      xm_fileop

    # Rotate our nxlog log file every week on Sunday at midnight
    <Schedule>
        When    @weekly
        Exec    if file_exists('%LOGFILE%') file_cycle('%LOGFILE%', 8);
    </Schedule>

</Extension>
```
