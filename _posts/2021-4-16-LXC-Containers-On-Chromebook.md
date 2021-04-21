Chromebooks are lean and are blessed with long battery life. My family gifted me a new Chromebook and I immedaitely embraced the Linux development environment (Beta) as well as running Visual Studio Code on it. But wait... I read that there are [LXC containers too](https://blog.mdfranz.com/linux-containers-vms-on-your-chromebooks-from-vmc-to-alpine-88bccdc78fdf)! Here are my notes.

1. Jump into the crosh shell using ctrl+alt+t
2. Then `vsh termina` to enter the termina VM, read more about it [here](https://chromium.googlesource.com/chromiumos/docs/+/8c8ac04aed5d45bb6a14605c422dbbd01eeadf15/containers_and_vms.md#How-many-VMs-can-I-run)
3. I want a REHL like shell so I enter `lxc launch images:centos/8 centos`
4. `lxc list` shows the onboarded containers, including the default named penguin which is debian
5. To enter my new container shell I use `lxc exec centos bash` 

A more detailed walkthrough, including how to side step having to use crosh to manage containers is at [https://ubuntu.com/blog/using-lxd-on-your-chromebook](https://ubuntu.com/blog/using-lxd-on-your-chromebook) 

Consult [images.linuxcontainers.org](https://images.linuxcontainers.org/) for the list of current publically available containers.
