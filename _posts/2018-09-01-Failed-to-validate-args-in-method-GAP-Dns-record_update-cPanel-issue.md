---
layout: post
title: Failed to validate args in method GAP::Dns::record_update - cPanel issue
date:   2018-09-16 08:15:31 +0200
tags: [Firestore, Firebase, Google, cPanel, WHM, VPS,]
category: Hosting
---
![img](https://amdtllc.com/storage/blog/1537230115.png)

Recently I needed to verify my domain with Google's Firestore to allow custom domain on emails sent by the app. I headed to the cPanel to update the TXT and CNAME records as instructed by Google.  That is where I encountered an issue  Failed to validate args in method GAP::Dns::record_update. A quick internet search proved futile with many having the same issue and hosting companies providing generic answers or telling users to talk to their tech support.  The problem is, most tech support response was that it is not possible to do so. 

While tinkering with different settings, I accidentaly came across a fairly easy solution. This will only work if you have access to a WebHost Manager (WHM) account. Usually if you have a VPS package with your hosting and can be accessed on port 2086 of your domain (example.com:2086). 

1. Login to your cPanel and navigate to the domain's DNS editor.
1. Enter the host record but remove any underscores in the field "Points To:" then click "add record". The record should save successfully. ![img](https://snag.gy/6oZmTz.jpg) 
1. Login to your WHM and navigate to "edit DNS Zone"
1. Select the domain you are working on and click "edit" ![img](https://snag.gy/7yFTwU.jpg)
1. Scroll down to the record you just entered in cPanel DNS editor and overwrite the "Points To: " record with the correct one including the underscores.
1. Save and go back to cPanel and reload to verify that the settings updated ![img](https://snag.gy/ToqWOG.jpg)

## Update zones via SSH
If you have VPS hosting or root access to your server, you can use the following steps to update the zone configurations. 
1. Login to your account using SSH terminal as root
1. Your zone configurations are located in `/var/named` Enter command `cd /var/name $$ ls`
>This will navigate to the /var/named directory and list files. 
1. Edit the zone for the domain you want from the list of domain names ending with `.com.db` using nano or vim For example `nano johnmuchiri.com.db` 
1. Enter the CNAME or TXT entry as needed on the bottom of the file. For example

```sh
johnmuchiri.com.        14400   IN      TXT     firebase=myapp
firebase1._domainkey    14400   IN      CNAME   mail-myapp-com.dkim1._domainkey.firebasemail.com.
firebase2._domainkey    14400   IN      CNAME   mail-myapp-com.dkim2._domainkey.firebasemail.com.
```

Include the trailing dots as shown in TXT and CNAME

Save the file and reboot server or use rndc command instead of rebooting the server `rndc reload <zone or domain>`

>DNS changes can take up to 4 hours for propagation but in most cases, I have seen the changes take only 10 - 20 minutes. 

