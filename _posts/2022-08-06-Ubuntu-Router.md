---
title: Ubuntu Router
date: 2022-08-06 13:30
categories: [homelab, project, routing]
tags: [servers,homelab,bash,routing,firewall,ubuntu]
---

# Ubuntu Router

I've been using my raspberry pi4 with 2 extra USB3 to LAN Gb adapters for a few months and it has felt quite stable for a routing device. My ISP speed is 65Mbps Download and 18Mbps Upload, and with the lack of AES, i've not attempted to do VPN traffic, but i have tested it with openelec, and it was able to push through my ISP download speed, but it might struggle if you have a better internet connection.

## So, the Ubuntu router

I've decided to stay with the UFW firewall, but i have thought about trying firewalld.. maybe for another day. With the type of routing I required, I decided to make a bridged network, and hence the two extra USB ethernet ports.

Packages installed: 
* isc-dhcp-server
* cockpit
  
So lets begin

## Netplan

Check which interfaces are available...

```bash
ip a
```

The three interfaces i'm interested are the built in ethernet and 2 x USB ethernet.


> eth0 - built-in ethernet
> 
> enxa0cec8c0b0e2 - usb ethernet
> 
> enx000ec6dab3a7 -usb ethernet

eth0 will be my WAN

enx000ec6dab3a7 & enxa0cec8c0b0e2 will be my bridge network br0.10

Firstly, lets create the bridge network using netplan...

```yaml
network:
    ethernets:
      eth0:
        addresses: [192.168.1.10/24]
        routes:
          - to: default
            via: 192.168.1.1
        nameservers: 
          addresses: [9.9.9.9, 149.112.112.112]
      enx000ec6dab3a7: {}
      enxa0cec8c0b0e2: {}
    bridges:
      br0.10:
        addresses: [10.10.10.1/24]
        nameservers:
          addresses: [9.9.9.9, 149.112.112.112]
        interfaces:
          - enxa0cec8c0b0e2
          - enx000ec6dab3a7
    version: 2
    renderer: NetworkManager
```
***eth0***

Addresses is the ip address for the ubuntu router, which is connected to my ISP router (for the WAN), and the ISP router supplies this LAN ip.

For eth0 default routes via... this is my router ip address 192.168.1.1.

For eth0 nameservers, i've just included quad 9's IP address, but update with your own preference.

enx000ec6dab3a7: {} - This basically identifies the interface for later use

enxa0cec8c0b0e2: {} - This basically identifies the interface for later use

***Bridges***

br0.10 is the interface i'll be using for my LAN, using the ip address 

10.10.10.10.1 will be the ubuntu lan ip which devices connect to.

Nameservers is the same idea as the WAN.

Includes both interfaces for the bridge

renderer - i added this in for NetworkManager, so i could use cockpit with this setup.

## isc-dhcp-server

```bash
sudo apt install isc-dhcp-server
```

Now lets edit this file /etc/default/isc-dhcp-server

at the bottom include these two lines

```bash
INTERFACESv4="br0.10"
INTERFACESv6=""
```

## /etc/dhcp/dhcpd.conf

Now lets edit this file /etc/dhcp/dhcpd.conf

```bash
default-lease-time 43200;
max-lease-time 86400;
authoritative;
subnet 10.10.10.0 netmask 255.255.255.0 {
    option domain-name "home.lan";
    option routers 10.10.10.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 10.10.10.255;
    range 10.10.10.100 10.10.10.200;
    option domain-name-servers 9.9.9.9;
}
```
Lease times are something a DHCP server will renew, and these are the times i've decided on.

This sets up a 10.10.10.1/24 LAN network for your devices to connect to with DHCP.

The range of 10.10.10.100-10.10.10.200 is for the dhcp server. This reserves the 10.10.10.2-10.10.10.99 & 10.10.10.201+ range for static IPs

Looking at ubuntu's documentation on this, it looks like you can assign static IP's using the MAC addresses in this config file.

[Ubuntu - isc-dhcp-server link](https://help.ubuntu.com/community/isc-dhcp-server)

All my static IP's are done on the host, so it's not something i've centralised like this.

## dhcp leases

You can view your DHCP leases here...

```bash
less /var/lib/dhcp/dhcpd.leases
```

## UFW

Ok, this will depend a little on your services, but this is where i've went with it.

```bash
sudo ufw limit from 10.10.100.20 to any port 22 proto tcp
sudo ufw allow from 10.10.10.0/24 to any port 67 proto udp
sudo ufw allow from 10.10.10.0/24 to any port 68 proto udp
sudo ufw allow from 10.10.10.0/24 to any port 53 proto tcp
sudo ufw allow from 10.10.10.0/24 to any port 53 proto udp
sudo ufw allow from 10.10.10.0/24 to any port 9090 proto tcp

sudo ufw route allow in on br0.10 out on eth0

```
I've allowed ssh (with a rate limit) access on the LAN, DHCP (67,68), DNS(53), and cockpit(9090).

I'm allowing traffic to be forwarded from the bridge, to the WAN.

We also need to edit /etc/ufw/sysctl.conf and uncomment this..

```bash
net.ipv4.ip_forward=1
```

Lastly for the firewall, we need a NAT masquerade.

Lets edit the /etc/ufw/before.rules and include these lines before the filter rules.

```bash
# nat Table rules
*nat
:POSTROUTING ACCEPT [0:0]

# Forward traffic from br0.10 through eth0.
-A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE

# don't delete the 'COMMIT' line or these nat table rules won't be processed
COMMIT
```

Directly under the MASQUERADE rule, the rule for that table must have COMMIT. This is the case for each table in these rules.

We should be able to apply our netplan now..

```bash
sudo netplan --debug generate
sudo netplan generate
sudo netplan apply
```

We should be able to apply our firewall rules now as well

```bash
sudo ufw disable && sudo ufw enable
```

ufw reload might work, but the documentation from Ubuntu states the above. command.

If you want a birds eye view of your router, cockpit is great, and if you've configured your firewall like mine, you should be able to connect to it via port 9090

So i would install cockpit...

```bash
sudo apt install cockpit
```

For the UFW firewall, check out Ubuntu's website, they might cover some commands you'll find useful.

[Ubuntu UFW Docs](https://ubuntu.com/server/docs/security-firewall)