---
title: Quick Proxy Setup for LXD
date: 2023-09-17 17:00
categories: [homelab, lxd, networking, proxy]
tags: [homelab, lxd, networking, proxy]
---

# The why?

So I don't forget is the main reason on this occasion. I have been playing with the idea of setting up an LXD container for code-server, but with a machine with enough grunt and doesn't zap too much power at idle. Queue in my laptop with a ryzen 5800h.

Problem with the laptop? Yes. I don't have ethernet for it, so I cant bridge a network with my wifi the way i normally would. So, i've learned about this proxy device trick instead.


```bash
# On the host...
# make sure you're on local if using laptop like me
lxc remote list
# Then launch an ubuntu container and enter with bash
lxc launch ubuntu:22.04 code-server
lxc exec code-server bash

# Inside the container...
# Replace ${VERSION} with the version you're installing
apt update && apt upgrade -y

# Grab this version from their github and set the version variable.
VERSION=4.16.1
wget https://github.com/coder/code-server/releases/download/v${VERSION}/code-serer_${VERSION}_amd64.deb

dpkg -i code-server_${VERSION}_amd64.deb

# Create a user for code-server. I'm adding to sudo, but you probably don't have to.
useradd -m -s /bin/bash -G sudo jonny
passwd jonny
su jonny

# Enable the service for jonny
sudo systemctl enable --now code-server@$USER

# Grab the password set for your user in their home directory
cat .config/code-server/config.yaml

sudo reboot

#### On The host

lxc config device add code-server codeserverPort proxy listen=tcp:0.0.0.0:8080 connect=tcp:127.0.0.1:8080

```

On your laptop, you can access via localhost:8080, but should be discoverable with your laptop ip address on port 8080 also.

Obviously, you'll want firewall rules in place here, but this will get you up and running.
