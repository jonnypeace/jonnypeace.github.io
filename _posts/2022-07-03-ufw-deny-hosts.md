---
title: UFW & Hosts Deny/Allow
date: 2022-07-03 22:00:00 -500
categories: [ubuntu,hosts.deny,hosts.allow,firewall]
tags: [firewall,ufw,hosts.deny,hosts.allow]
---
# Quick way to Strengthen Ubuntu Security

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