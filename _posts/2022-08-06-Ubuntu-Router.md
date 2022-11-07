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
sudo ufw limit from 10.10.10.0/24 to any port 22 proto tcp
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

## VLANs

update: 07/11/2022

Ok, I wanted to write about this earlier in the year, but my managed switch died on me, and I've only just got round to replacing it.

FYI, I am using a TP-Link smart managed switch here, nothing too fancy but this is the model number TL-SG1016DE. All I need is a little network segregation and this fits the bill for a reasonable price. No high speeds in this house, but gigabit LAN has been good to me over the years.

Due to me self hosting some services, I wanted to keep the self hosted network separate from my safer home network, and hence the vlan. I am going to continue with the bridge netplan discussed earlier, and include one VLAN in this configuration. This vlan network consists of an LXD cluster and some other goodies, but for container creation, i need DHCP as i'm using macVLAN with the LXD cluster, so in this section i'll have to include something for the VLAN DHCP server, so lets begin...

##  VLAN - Netplan YAML
```bash
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
        addresses:
          - 10.10.10.1/24]
        nameservers:
          addresses: [9.9.9.9, 149.112.112.112]
        interfaces:
          - enxa0cec8c0b0e2
          - enx000ec6dab3a7
    vlans:
      vlan2:
        id: 2
        link: br0.10
        addresses: [10.10.20.1/24]
    version: 2
    renderer: NetworkManager
```

## /etc/ufw/before.rules

This file now has this inclusion of the VLAN (10.10.20.0/24)masquerade..

```bash
# nat Table rules
*nat
:POSTROUTING ACCEPT [0:0]

# Forward traffic from vlan2 & br0.10 through a NAT on eth0.
-A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE
-A POSTROUTING -s 10.10.20.0/24 -o eth0 -j MASQUERADE

# don't delete the 'COMMIT' line or these nat table rules won't be processed
COMMIT
```

Ok this bit will be dependant on the whether you want one way traffic or any sort of communication between your VLAN and safer home network, and i do, so i'll include them and explain.

```bash
# When you find the rules below....

# Don't delete these required lines, otherwise there will be errors
*filter
:ufw-before-input - [0:0]
:ufw-before-output - [0:0]
:ufw-before-forward - [0:0]
:ufw-not-local - [0:0]
# End required lines

# ENTER FORWARD RULES NOW...

#### MY forward
# vlan2 to WAN - this could be included with the standard UFW command, but since i'm here (a few of these rules could probably be applied on the commandline, but this was easier to test the rules)...
-A ufw-before-forward -i vlan2 -o eth0 -j ACCEPT

## Firewall rules between vlan2 and specific device.
# Laptop to vlan2
-A ufw-before-forward -s 10.10.10.185/32 -o vlan2 -j ACCEPT
# phone to vlan2
-A ufw-before-forward -s 10.10.10.175/32 -o vlan2 -j ACCEPT
# NFS share for jellyfin - i've had to set it up this way, so the jellyfin container can mount the NFS share. 
# Funnily enough, I am using firewalld on this server, so i added the jellyfin to the public zone where only NFS is allowed. 
# Also, with NFS, i have provided these mount paths with read only access from /etc/exports, just to harden security. Jellyfin doesn't need write access to them.
-A ufw-before-forward -s 10.10.10.40/32 -o vlan2 -j ACCEPT
-A ufw-before-forward -i vlan2 -d 10.10.10.40/32 -j ACCEPT

# Leave this section at the bottom of the MY forward rules. This uses conntrack to prevent vlan2 from initiating new connections with my phone or laptop that i've let through the guard.
-A ufw-before-forward -o vlan2 -j ACCEPT
-A ufw-before-forward -i vlan2 -m state ! --state NEW -j ACCEPT
-A ufw-before-forward -i vlan2 -m state --state NEW -j REJECT

```
Once those rules are added, remember to...

```bash
sudo ufw reload
```

### UFW rules

Ok, now some standard DHCP and DNS access rules. It's worth pointing out. I'm using DNS 9.9.9.9 etc in my netplan example, but the reality is i'm using pihole, or i could use bind.

```bash
sudo ufw allow from 10.10.20.0/24 to any port 67 proto udp
sudo ufw allow from 10.10.20.0/24 to any port 68 proto udp
sudo ufw allow from 10.10.20.0/24 to any port 53 proto tcp
sudo ufw allow from 10.10.20.0/24 to any port 53 proto udp
```

## VLAN - DHCP Server

Ok, time to add the vlan2 to the DHCP server.
### /etc/dhcp/dhcpd.conf

```bash
default-lease-time 43200;
max-lease-time 86400;
authoritative;
## vlan2
subnet 10.10.20.0 netmask 255.255.255.0 {
    option domain-name "vlan2.lan";
    option routers 10.10.20.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 10.10.20.255;
    range 10.10.20.50 10.10.20.100;
    option domain-name-servers 9.9.9.9;
}
## home.lan
subnet 10.10.10.0 netmask 255.255.255.0 {
    option domain-name "home.lan";
    option routers 10.10.10.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 10.10.10.255;
    range 10.10.10.100 10.10.10.200;
    option domain-name-servers 9.9.9.9;
}
```

### /etc/default/isc-dhcp-server

```bash
INTERFACESv4="br0.10 vlan2"
INTERFACESv6=""
```
If there is anything that could be improved with this setup, please get in touch. I never included stateful rules with the ufw forward, simply because ufw-before-forward rules have this included by default...

```bash
# quickly process packets for which we already have a connection
-A ufw-before-input -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A ufw-before-output -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A ufw-before-forward -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```
And if you have set this up like I have, and scan iptables...

```bash
sudo iptables -vL | less
```

And look for...

```bash
Chain ufw-before-forward (1 references)
 pkts bytes target     prot opt in     out     source               destination         
1425K   78M ACCEPT     all  --  vlan2  eth0    anywhere             anywhere            
    0     0 ACCEPT     all  --  any    vlan2   10.10.10.144        anywhere            
  181 12442 ACCEPT     all  --  any    vlan2   10.10.10.125        anywhere            
20486 1846K ACCEPT     all  --  any    vlan2   nfs.home             anywhere            
27170 2328K ACCEPT     all  --  vlan2  any     anywhere             nfs.home            
3276K 4648M ACCEPT     all  --  any    vlan2   anywhere             anywhere            
 183K   81M ACCEPT     all  --  vlan2  any     anywhere             anywhere             ! state NEW
    5   372 REJECT     all  --  vlan2  any     anywhere             anywhere             state NEW reject-with icmp-port-unreachable
  43M   53G ACCEPT     all  --  any    any     anywhere             anywhere             ctstate RELATED,ESTABLISHED
    0     0 ACCEPT     icmp --  any    any     anywhere             anywhere             icmp destination-unreachable
    0     0 ACCEPT     icmp --  any    any     anywhere             anywhere             icmp time-exceeded
    0     0 ACCEPT     icmp --  any    any     anywhere             anywhere             icmp parameter-problem
 4836  342K ACCEPT     icmp --  any    any     anywhere             anywhere             icmp echo-request
59103   10M ufw-user-forward  all  --  any    any     anywhere             anywhere            

```
You can see the ctstate RELATED,ESTABLISHED processes the bulk of the packets (53G), and with my understanding... this accomplishes a stateful rule for the before-forward traffic. There's every chance i've missed or included something unnecessary, and I would be forever grateful if this was highlighted to me. Fingers crossed you have a new ubuntu router, with VLAN traffic.

## The Switch - TP Link TL-SG1016DE

Examples for making this simple smart managed switch to work with vlans. Just remember to set the vlan ID to the same ID as on your ubuntu router.

![Switch vlan settings](https://i.ibb.co/09Rcsyg/Screenshot-from-2022-11-07-21-37-05.png)
_bridge_

![Switch vlan settings](https://i.ibb.co/DG60R1j/Screenshot-from-2022-11-07-21-36-56.png)
_vlan2_

![Switch vlan settings](https://i.ibb.co/xzH3M2S/Screenshot-from-2022-11-07-21-37-27.png)
_default vlan-id_