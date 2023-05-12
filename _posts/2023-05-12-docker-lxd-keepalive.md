---
title: High Availability with KeepaliveD-Docker-LXD
date: 2023-04-28 13:00
categories: [homelab, Docker, keepalived, LXD, bash, high availability]
tags: [omelab, Docker, keepalived, LXD, bash, high availability]
---

# Docker-LXD-KeepaliveD

This should be a _mostly straight forward_ setup. There are plenty other ways to configure, this probably the easiest.

Also, this would work better if you are in an LXD cluster, which is out of scope for this, but will add a short section about it further down.

Also also, instead of a network share to synchronize the containers, you should probably use microceph with LXD (microcloud), which would also handle migration much better, but for now, i'll go with network share, and will probably add a bit with microceph later.

## Git Repo

First of all, clone the git repo...

```bash
git clone https://github.com/jonnypeace/docker-lxd-keepalived.git
```

## Setting up an LXD container to use with docker

Docker cant run on zfs
```bash
lxc storage list # you'll see storage pools here.
```

Create a pool called docker with btrfs
```bash
lxc storage create docker btrfs
```

Launch an ubuntu container called fast-ubuntu
```bash
lxc launch images:ubuntu/22.04 fast-ubuntu
```

Create storage volume for fast-ubuntu
```bash
lxc storage volume create docker fast-ubuntu
```

Add storage from storage pool docker created earlier to fast-ubuntu, and use it for /var/lib/docker
```bash
lxc config device add fast-ubuntu docker disk pool=docker source=fast-ubuntu path=/var/lib/docker
```

Poke some holes so docker can run inside the LXD containers
```bash
lxc config set fast-ubuntu security.nesting=true security.syscalls.intercept.mknod=true security.syscalls.intercept.setxattr=true
```

Restart container
```bash
lxc restart fast-ubuntu
```

Enter container and run bash
```bash
lxc exec fast-ubuntu bash
```

Install docker (Worth checking dockers website in case this procedure changes) and install docker-compose & keepalived
```bash
apt update && apt upgrade -y

sudo apt-get update
sudo apt-get install \
   ca-certificates \
   curl \
   gnupg \
   lsb-release

mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-compose keepalived
```

Test docker install
```bash
docker run -it ubuntu bash # run exit or ctrl+D to exit - you'll want to remove this ubuntu image/container probably.
```

List containers and grab ip.
```bash
lxc list fast-ubuntu # for ip
```

Add network share - if you use this feature as per cron job below, adjust as necessary for both
```bash
# add network shares mounted on host
# Source is the host mount
# path is the mount inside the lxd container
# network-share is the name of the device
lxc config device add fast-ubuntu network-share disk source=/mnt/shares/ path=/mnt/docker
```

You'll need two of these containers to run keepalived and since its high availability and LXD clusters...

```bash
lxc copy fast-ubuntu fast-ubuntu-slave --target server-name
```

If you're just testing on a local machine, remove the --target server-name part of the above command

## monitor.sh

This is used by keepalived to keep track of a service on your network... in my example, i've used portainer as an example

You'll want to modify this for your own subnet...

```bash
ka_host_ip='10.10.200.69 9443' # EDIT to suit subnet
```

This ip will be the same (virtual) IP that is set in your keepalived config files

You'll also want to edit this array for container names - as per docker-compose files.

```bash
cont_arr[1]='portainer' # This will also be used as my keepalived monitor container
cont_arr[2]='freshrss'
cont_arr[3]='vaultwarden'
cont_arr[4]='watchtower'
#cont_arr[5]=''
#cont_arr[6]=''
#cont_arr[7]=''
```

Lastly for this script, the directory of /docker(/container) is assumed for all your docker-compose files.
So you'll want to modify as necessary. If you have docker-compose files scattered everywhere, this script won't work.

```bash
# root docker directory where containers located, following a pattern like so...
# /docker/portainer, /docker/freshrss etc adjust as necessary.
dock_dir='/docker'
```

## Master & Slave keepalived config (obviously this is for a pair of servers).

In your master server, copy etc.keepalived.master.conf file into /etc/keepalived/keepalived.conf

For your slave server, copy etc.keepalived.slave.conf file into /etc/keepalived/keepalived.conf

All that really needs modified here is this line, and ensure same virtual ip as in your monitor script.

```bash
  10.10.200.69 # EDIT to suit subnet
```

Lastly, you'll want to enable and start keepalived

```bash
systemctl enable --now keepalived
```

If you run 

```bash
systemctl status keepalived
```
The master will show...

```
Entering MASTER STATE
```
The slave will show...

```
Entering BACKUP STATE
(VI_2) Changing effective priority from 80 to 81
```

Or something along those lones, depending probably whether the slave was a master to begin with. The main takeaway is that one will be in master state, and the other in backup state.

## docker-update.sh

I will probably update this to make it more fancy, but for now, it's a simple brace expansion, test directory, change to directory, and pull new image and start container.

Again, this is assuming a root directory of /docker/containers....

So modify as necessary.

## cron jobs

Again, this should be mostly self explanatory. I've shown how often i would run the monitor script, this means there should only really be around 1minute of downtime, give or take, depending if theres a docker image update.

So edit this for the location of the keepalived monitor script.

```bash
Monitors every minute for keepalived
*/1 * * * * /home/user/monitor.sh
```

I've also shown a short rsync command to keep containers in sync over a mounted share, so you'll want both servers to have the same mount if you use the same command as I. 

Hope this helps, feel free to let me know if there are any issues.
