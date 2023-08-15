---
layout: post
title: PowerShell Backfill DNS PTR Records 
description: Automate DNS Reverse Lookup Record Creation
---

There is an IT Engineering meme that says "It is always DNS" because, once the issue reaches root cause analysis phase, often DNS is at fault.

Of course, DNS did not operate inconsistently, it was misconfiguration at fault. One common misconfiguration is not creating a reverse lookup zone and PTR (Pointer) records in the reverse lookup zone. Those PTR records make it easy for DNS to answer queries about an IP address instead of the normal DNS name. The reverse of the normal DNS function.

## Here is My PowerShell Script Solution

```
$ComputerName = 'AD.DC.FQDN'
$Domain = '.sub.domain.tld'
$Zone = 'sub.domain.tld'

$records = (Get-DnsServerResourceRecord -ZoneName $Zone -RRType A -ComputerName $ComputerName | Where-Object -FilterScript {$_.HostName -like '*.not-zone-sub'})

ForEach ($record in $records){
              $ipv4 = Get-DnsServerResourceRecord -ZoneName $Zone -RRType A -ComputerName $ComputerName -Name $record.HostName | Select -ExpandProperty RecordData
              $Name = ($ipv4.IPv4Address.ToString() -replace '^(\d+)\.(\d+)\.(\d+).(\d+)$','$4')
              $ZoneName = ($ipv4.IPv4Address.ToString() -replace '(\d+)\.(\d+)\.(\d+)\.(\d+)', '$3.$2.$1') + ".in-addr.arpa"
              $PtrDomainName = $record.HostName + $Domain
              Add-DnsServerResourceRecordPtr -Name $Name -ZoneName $ZoneName -PtrDomainName $PtrDomainName -ComputerName $ComputerName
}
```

*Yay!* Now I can throw reverse lookup queries at DNS and it will respond back with the configured DNS A record.

### Thanks for reading!

Got feedback? Connect with me on [Github](https://github.com/kamsalisbury) or [LinkedIn](https://www.linkedin.com/in/kam-reef-salisbury/).

###### Notes

1. DNS will function perfectly without any reverse lookup zones or PTR records. With the exception (of course) of not being able to match up a given IP address with a DNS A record.
2. The PowerShell script in this post can be [downloaded directly from Github](https://github.com/kamsalisbury/ps/blob/b5934c1be92abc8f232cd475badc0cbd92c61999/Example-Backfill-DNS-PTR.ps1).
