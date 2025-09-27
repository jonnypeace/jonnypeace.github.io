---
title: About
icon: fas fa-info-circle
order: 4
---

# Short History of When it all Started

Recently completing my LFCS certification, I also really enjoy coding/scripting/automation with Bash, VBA, Python, Gawk & Ansible. 

I have been home labbing since 2014, starting out with my very first Ubuntu 14.04 server. I started off running a snapraid(samba)/plex/vpn/sendmail(gmail) server, running on an old (now running Arch) i7 3770k, asus p8z77-v lx motherboard and 16GB g-skill tridentX 2400mhz (fast for ddr3) of RAM. I did have an old OVH Linux VPS, but was mostly used for testing my linux skills on. I would also run my own satellite tuner PCI card and run my own TV server. I've owned every raspberry pi that has been released, but the fun Pi work started with the raspberry pi 2, where i set up my own music server/internet radio using kodi (or XBMC (or openelec if i remember right)from back in the day), which also hosted it's own pi hat amp, and speakers.

A lot has happened since then, so i feel the time is right to document/blog all these cool projects I get myself involved in.

Currently I manage 8 Ubuntu servers locally

* 3 are in a LXD cluster for hosting services such as Vaultwarden, Nginx proxy, Nextcloud, Onlyoffice, utilizing microceph.
* 1 server is being used for desktops requiring GPU passthrough virtualisation.
* 1 server is an LXD playground, but also runs pihole and bind9 DNS servers in LXD.
* 1 server is being used as an Ubuntu router with vlans, DHCP, 5 port 2.5gb LAN, and a ZFS raid pool.
* Raspberry pi4 with another pihole, for backup DNS.
* Raspberry pi4 with storage dock where I rsync backups to.

I also run another server for proxmox, currently offline due to energy prices, but it also hosts 2 x ZFS raid pools.

I have a smart switch, vlan aware to help with network separation.

And now into the cloud...

I have been running something in the cloud for several years, using contabo or linode, but i've condensed my infrastructure for a mostly self-hosting experience. I reverse proxy my traffic with Linode, but also capable of running services through a WAF with cloudflare.

See you in the next blog post. :)

[![LFCS Certified](/assets/lfcs-linux-foundation-certified-systems-administrator.2.png)](https://www.credly.com/badges/47e41727-76a8-4299-827b-52dccbb43bc9/public_url)
_LFCS Certified_

### _Update 12-03-2023_

Recently just completed my entry level python institute exam PCEP, and will probably move on to the PCAP level for my next cert, we shall see.

[![PCEP certified](/assets/pcep-30-02-pcep-certified-entry-level-python-programmer.png)](https://www.credly.com/badges/bc743aac-74be-4f5f-9853-ce306cbd0d56/public_url)
_PCEP Certified_

### _Update 27-09-2025_

It's been a while since my last blog post, I've been super busy with work and upskilling. Around April 2023, I started a new role in Machine Learning using python. I've since thinned out a lot of the servers I had running, and centralised into one. I was so busy with work and upskilling, I needed to keep things more simple, so now...

* A workstation/server using Ryzen 5650g Pro, and ECC RAM, and a RTX5060Ti. This has around 4TB of available storage in Raid5 using ZFS. I used Ubuntu Desktop so i could still get some Desktop use out of it. While I do have the linux router  with VLANS, i've decided to sandbox a network on LXD with firewall rules, so I can expose some parts of this server with wireguard to VPS (without opening ports), mainly a Flask App i've created, and Open WebUI so I can have my own privacy AI.
* 1 x raspberry pi working as DNS server.
* 1 x 5 port 2.5Gbe Ubuntu Router
* VPS, which is my gateway to services running at home. 

Sacrificed high availability for something smaller to manage, and less power consumption.