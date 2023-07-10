---
layout: post
lang: en
title: How To Detect Ssh Public Key Attempts  
description: Public Key Authentication Indicators
categories: [Cybersecurity, Ssh]
#image: image-link #this appears on share cards in social media
---

It is universally accepted that configuring sshd to require publickey authentication, is more secure than a password alone.

How can you validate ssh logins are authenticated using publickey? Or, if a former employee is trying to use an outdated key?

The organization changes keys when there is a change in personnel... right?

I have found that without specific sshd configuration, ssh publickey authentication events identifying risks to the organization are missed.

## Here is my example.

I configured my example linux system to require publickey and password authentication.

```
AuthenticationMethods publickey,password
```

On the example linux system, ```/var/log/messages``` syslog shows the ssh login password event, but the publickey event is missing.

```
Jul  8 09:29:43 host auth.info sshd[3170]: Accepted password for kam from ###.###.###.### port 51527 ssh2
```

Checking ```/etc/ssh/sshd_config``` I can see the default sshd logging level is INFO.

```
# Logging
#SyslogFacility AUTH
#LogLevel INFO
```

I am surprised that ```LogLevel VERBOSE``` is required to log publickey login events.

```
Jul  8 11:19:36 jump1 auth.info sshd[3502]: Server listening on 0.0.0.0 port 22.
Jul  8 11:19:36 jump1 auth.info sshd[3502]: Server listening on :: port 22.
Jul  8 11:20:02 jump1 auth.info sshd[3510]: Connection from ###.###.###.### port 52268 on ###.###.###.### port 22 rdomain ""
Jul  8 11:20:03 jump1 auth.info sshd[3510]: Accepted key ED25519 SHA256:LoooooooongKey found at /home/kam/.ssh/authorized_keys:1
Jul  8 11:20:03 jump1 auth.info sshd[3510]: Postponed publickey for kam from ###.###.###.### port 52268 ssh2 [preauth]
Jul  8 11:20:03 jump1 auth.info sshd[3510]: Accepted key ED25519 SHA256:LoooooooongKey found at /home/kam/.ssh/authorized_keys:1
Jul  8 11:20:03 jump1 auth.info sshd[3510]: Partial publickey for kam from ###.###.###.### port 52268 ssh2: ED25519 SHA256:LoooooooongKey
Jul  8 11:20:19 jump1 auth.info sshd[3510]: Accepted password for kam from ###.###.###.### port 52268 ssh2
Jul  8 11:20:19 jump1 auth.info sshd[3510]: User child is on pid 3512
Jul  8 11:20:19 jump1 auth.info sshd[3512]: Starting session: shell on pts/1 for kam from ###.###.###.### port 52268 id 0
```

Thanks for reading this.

Got feedback? Connect with me on [Github](https://github.com/kamsalisbury) or [LinkedIn](https://www.linkedin.com/in/kam-reef-salisbury/).

###### Notes
1. Alpine linux 3.1x was used for this topic. Your linux distribution of choice may have different sshd and syslog configuration defaults. For example, rsyslog filtering ssh authentication to ```/var/log/auth.log```.
1. The example sshd configuration used for this topic purposely combines something the login account has (a publickey) with something the login account knows (a complex password).
1. This configuration requirement [has existed for a very long time](https://serverfault.com/questions/291763/is-it-possible-to-get-openssh-to-log-the-public-key-that-was-used-in-authenticat).