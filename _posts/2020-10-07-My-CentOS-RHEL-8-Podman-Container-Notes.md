On the CentOS (and Redhat Enterprise) Linux platform, podman is the utility to run containers without the complexity of a Kubernetes daemon service configuration. Even better, podman uses the same commands as Docker, so I don't "have to" install a Docker environment.
Update: Redhat has ended their support of resources for the CentOS program. CentOS will not receive updates after 2021. [Redhat has added a Redhat Enterprise Linux subscription benefit to their developers program](https://developers.redhat.com/articles/getting-red-hat-developer-subscription-what-rhel-users-need-know) which basically allows me to convert my few CentOS VMs to RHEL. I have updated these notes to reflect any RHEL specific changes.

Links to get started...
[CHAPTER 4. RUNNING CONTAINERS AS SYSTEMD SERVICES WITH PODMAN](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/managing_containers/running_containers_as_systemd_services_with_podman)

[How To run Docker Containers using Podman and Libpod](https://www.osradar.com/how-to-run-docker-containers-using-podman-and-libpod/)

[How to Run Containers with Podman on CentOS 8 / RHEL 8](https://www.linuxtechi.com/run-containers-podman-centos-8-rhel-8/)

1. Coding scripts to start containers is easier than all other options. For example, here is a simple shell script with the complete command line to start the container. `sudo podman run -it -d -p 8080:80 --name nginx -v /usr/share/nginx/html:/usr/share/nginx/html -v /home/me/default.conf:/etc/nginx/conf.d/default.conf -e TERM=xterm docker.io/library/nginx`
2. Some containers will just crash because they need a configuration for SELinux. I am looking at you nginx! (An option for occasional bypass is to `sudo setenforce 0`. Research using udica to create the proper SELinux policy instead of bypassing)
3. Copy container's configuration files as needed, so you can have a customized version of the file that persists after the container gets updated. `podman cp nginx:/etc/nginx/conf.d/default.conf default.conf`
4. Enter a shell on the running container `sudo podman exec -it nginx bash`
5. Running containers upon boot from systemd is realistic. See the workflows below...
6. Containers not designed to provide an interactive shell session (single use or command) expect parameters passed to them like this. Notice no command to exec specified after the containerimage `sudo podman run -d --rm=true containerimage --parameter_one="value1" --parameter_two="value 2" --parameter_three="looongkey"`
7. Quick check on running container `sudo podman logs containerId`

## Workflow to run pihole as a container on CentOS 8
```
# Create a podman volume to hold pihole configuration files
sudo podman volume create pihole
# Pull the image
sudo podman pull docker.io/pihole/pihole
# Run the image the first time, use google foo to research and solve any port conflict issues
sudo podman run -dt --name pihole -p 53:53/udp -p 80:80/tcp -e SERVERIP=yourcentosip  -e DNS1=149.112.112.112 -e DNS2=9.9.9.9 -e TZ=America/New_York -v pihole:/etc/pihole:Z -e WEBPASSWORD=piholewebadminpassword docker.io/pihole/pihole
# Stop the container temporarily
sudo podman stop pihole
# Create the systemd configuration so pihole container can run as a systemd service upon boot
sudo podman generate systemd pihole > /etc/systemd/system/pihole-container.service
# Refresh systemd
sudo systemctl daemon-reload
# Enable pihole container to as auto run as a systemd service
sudo systemctl enable pihole-container
# Start pihole as a container from systemd
sudo systemtcl start pihole-container
```

## Workflow to update the pihole as a container image
```
# Stop the running systemd pihole container
sudo systemctl stop pihole-container
# Pull the latest image
sudo podman pull docker.io/pihole/pihole
# Start the systemd pihole container
sudo systemctl start pihole-container
```
