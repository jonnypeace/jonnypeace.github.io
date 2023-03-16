---
title: UFW & Hosts Deny/Allow (More for VPS)
date: 2022-07-03 22:00:00 -500
categories: [ubuntu,hosts.deny,hosts.allow,firewall]
tags: [firewall,ufw,hosts.deny,hosts.allow]
---
# Possible way to Strengthen Ubuntu Security

_Update 16/03/2023_ This never did break anything, but worth pointing out that i decided to remove this server (well actually, i left it dormant for about 4 months, with no search engine finding it (so nobody should find it) and let it gather ip addresses that shouldn't be focusing on it - i then sent the data to crowdsec). Also worth mentioning, this will only work with programmes that work with the tcp wrapper hosts deny/allow. I usually have UFW set up with rate limiting on open ports, and the rest are blocked. So, the script below does no further analysis but filter IP addresses which have broken the UFW firewall rules, set by yourself. I know firewalld allows the use of a ban ip text file which would be better. I might also update with an iptables solution using ipset... to be continued...

Something i'm noticing in my UFW logs on my VPS (internet facing), are a number of syn blocks from what i suspect will be hackers / bots, so many on my contabo server (~ 4000 i've got logged just now), it's difficult to keep up. A cross section of the traffic reveals Russia, Bulgaria, USA and so on. I will leave this blog up, for this use case. It might be overkill, and i might break something, in which case i'll take it down.



Check the number of ip's in your UFW logs that have been blocked.

```bash
sudo egrep -o "SRC=[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" /var/log/ufw.log | sort -u | wc -l
```

Create a ban list

```bash
sudo egrep -o "SRC=[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" /var/log/ufw.log | sort -u | cut -d= -f2 > ~/banning
```

Copy ban list to /etc

```bash
sudo cp ~/banning /etc/ban.list
```

## Before updating hosts.deny

Add one of these lines with your details to the /etc/hosts.allow in case you lock yourself out.

```bash
ALL: mydomain.co.uk
ALL: my.ip.address
ALL: my.dynamic.dns.service
```

## Updating hosts.deny

Add this line to /etc/hosts.deny

```bash
ALL: /etc/ban.list
```

The ALL: syntax refers to all daemons so you could write sshd, or something else if you want to be specific.

## To update this ban.list file

Run this again..

```bash
sudo egrep -o "SRC=[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" /var/log/ufw.log | sort -u | cut -d= -f2 > ~/banning
```

Concatinate with previous list, and sort unique ip's

```bash
sudo cat /etc/ban.list banning | sort -u > ~/ban.list 
```
Copy the new ban.list file to /etc

```bash
sudo cp ~/ban.list /etc/ban.list 
```

## Lets script this...

```bash
#!/bin/bash

# This can be an ip address or domain, which will be used to ensure you don't lock yourself out. 
# Uncomment below to add your ip address
#my_domain=my.ip.address
my_domain=$(host example.com | cut -d' ' -f4)

#Check if your domain has been added to the hosts.allow

if ! sudo grep "$my_domain" /etc/hosts.allow > /dev/null
then
	echo "ALL: $my_domain # Home" | sudo tee -a /etc/hosts.allow
fi

#Check if your ban.lisst has been added to hosts.deny

if ! sudo grep "/etc/ban.list" /etc/hosts.deny > /dev/null
then
	echo "ALL: /etc/ban.list" | sudo tee -a /etc/hosts.deny
fi

#Check if the file banning exists, if not treat as if first run. If file exists, concatenate the new banning file with the existing ban.list, sort unique and move the new list to the correct location.

if [[ ! -f "$HOME"/banning  ]]
then
	sudo egrep -o "SRC=[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" /var/log/ufw.log | sort -u | cut -d= -f2 > "$HOME"/banning
	sudo cp "$HOME"/banning /etc/ban.list
else
	sudo egrep -o "SRC=[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" /var/log/ufw.log | sort -u | cut -d= -f2 > "$HOME"/banning
	sudo cat /etc/ban.list banning | sort -u > "$HOME"/ban.list
	sudo cp "$HOME"/ban.list /etc/ban.list
fi

```
