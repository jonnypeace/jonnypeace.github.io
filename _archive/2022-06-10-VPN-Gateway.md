---
title: Ubuntu 20.04 VPN Gateway
date: 2022-06-10 19:30 -500
categories: [homelab, project, vpn, routing]
tags: [servers,homelab,bash,vpn,routing,nordvpn,iptables]
---

# VPN-gateway

The purpose of these scripts is to connect devices on my LAN to a VPN gateway using wireguard.

Originally my plan was to set up a raspberry pi with open-wrt and connect the devices over wifi
with a stronger wifi dongle, but I couldn't resist exploring this option first.
I'm aware that I could use static ip gateway for something like this, but to keep things easier
to switch on and off (the wireguard client), I feel this is more user friendly than switching gateways.

The benefit of doing it this way, if we have a server running at home anyway, you don't need
additional hardware (wifi access points for the server), just the wireguard client/server application. I've got this running in an 
Ubuntu VM, on a proxmox server, and this ubuntu VM is dedicated to serve only this function. I will be exploring the use of LXC containers
as well, but VM's are easier.

WARNING : 
This runs strictly iptables only, and doesn't take into account nftables or UFW or firewalld. Also, As much as i think this works, I am just as capable as the next person to overlook something, so just be wary whatever your activity is.

So, a quick flow diagram....

Phone/pc device with wireguard client >> VPN Gateway, with wireguard server traffic coming in >> VPN gateway with NORD VPN traffic going out.

The firewall used for iptables is a stateless firewall, something i've been debating with myself over the months and settled with. Feel free to send me a message with any pro's and con's on this.

############################################################################

## - Clone Repo

First of all, clone my repo for this.

```bash
git clone https://github.com/jonnypeace/VPN-gateway.git
```

## - WIREGUARD

Log into your server and Switch user to root. install wireguard and change directory
```bash
sudo su -
apt install wireguard
cd /etc/wireguard/
```
Create server Keys
```bash
umask 077; wg genkey | tee privatekey | wg pubkey > publickey
```
Create new file..
```bash
nano /etc/wireguard/wg0.conf
```
Paste this into it
```bash
[Interface]
Address = 10.6.0.1/24
ListenPort = 51820
```
Below command will include the private key in the wgo.conf.
```bash
echo "PrivateKey = $(cat privatekey)" >> wg0.conf
```
We need to uncomment (#) this line in /etc/sysctl.conf and update systctl. No edit necessary
```bash
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sysctl -p
```
By now, we can enable& start wireguard service.
```bash
systemctl enable wg-quick@wg0
systemctl start wg-quick@wg0
systemctl status wg-quick@wg0
```
This is a script from my bashscripts repository, which will help us add new users. 
Also change permissions so the script can run.
```bash
wget https://raw.githubusercontent.com/jonnypeace/bashscripts/main/wireguardadduser.sh
chmod 700 wireguardadduser.sh 
```
In this sed command we can update our DNS to use NordVPN (help with dns leaks) - this is client side only.
```bash
sed -i 's|DNS = 9.9.9.9|DNS = 103.86.96.100, 103.86.99.100|g' wireguardadduser.sh
```
To update the NordVPN dns on the server, i use netplan yaml to configure.
```bash
nano /etc/netplan/00-installer-config.yaml
```
(your yaml might be named differently)

Spacing is quite important in yaml, and it should look something like this. Update your gateway & addresses. Nameservers included here are of NordVPN.

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens3:
      dhcp4: no
      addresses:
        - 192.168.1.111/24
      gateway4: 192.168.1.1
      nameservers:
          addresses: [103.86.96.100, 103.86.99.100]
  version: 2
```
apply the netplan config
```bash
netplan apply
```
Check the DNS has been applied
```bash
systemd-resolve --status | grep 'DNS Servers' -A1
```
Below should provide the ip address for wireguard client config.
```bash
ip route list default | cut -d " " -f9
```
if this doesn't work, try 
```bash
ip a
```
Replace "HOSTNAMEorIPofGATEWAY" with the IP above (the above ip is for home use, if you're using a VPS or dynamic dns, it will be the hostname or public ip you use to SSH into the server, BUT i'm not covering VPS/cloud in this readme)
```bash
sed -i "s|Endpoint = MYDNS.ORMY.IP|Endpoint = HOSTNAMEorIPofGATEWAY|g" wireguardadduser.sh 
```

Ok, the script should be ready to run, just folllow the prompts.
```bash
./wireguardadduser.sh 
```
Now that the script has finished, we should see an entry in wg0.conf
```bash
cat /etc/wireguard/wg0.conf
```
And we should have a config for the client/desktop/mobile device.
```bash
cat /etc/wireguard/configs/NAMEofYOUR.conf
```
This .conf needs copied to the desktop or mobile device. You can copy/paste the contents if you have no other means of getting
the file off the server.

## - OPENVPN

Install openvpn, unzip, net-tools and curl and change directories
```bash
apt install openvpn unzip net-tools git curl
cd /etc/openvpn
```
Grab the server list from nordvpn. Check NordVPN website incase this link changes.
```bash
wget https://downloads.nordcdn.com/configs/archives/servers/ovpn.zip
```
unzip the server list and remove the zip file
```bash
unzip ovpn.zip
rm ovpn.zip
```
move into the udp server directory and list all uk servers, lots to choose from, and uk is just an example.
for the purposes of this readme, i'll select uk2161.nordvpn.com.udp.ovpn
```bash
cd /etc/openvpn/ovpn_udp
ls uk*
```

In this tutorial, we could use nano/vi/vim to edit files, but for a change, i figured lets do it the sed way for most of it. Feel free to edit with the editor of your choice.

We're looking for text containing "auth-user-pass" and replacing it with "auth-user-pass /etc/openvpn/auto-auth.txt" in this file uk2161.nordvpn.com.udp.ovpn
```bash
sed -i 's|auth-user-pass|auth-user-pass /etc/openvpn/auto-auth.txt|' uk2161.nordvpn.com.udp.ovpn
```
Enter your credentials into /etc/openvpn/auto-auth.txt i.e. These credentials will be found in your nordvpn account dashboard.
```
nano /etc/openvpn/auto-auth.txt

jonny
password123
```
and change the permissions so only root can read it:
```bash
chmod 400 /etc/openvpn/auto-auth.txt
```
we need to move uk2161.nordvpn.com.udp.ovpn into a config in the etc/openvpn directory
```bash
mv /etc/openvpn/ovpn_udp/uk2161.nordvpn.com.udp.ovpn /etc/openvpn/uk2161.conf
```
edit the default openvpn config to autostart with uk2161 (or which ever server you chose) /etc/default/openvpn
```bash
sed -i '/AUTOSTART="all"/a AUTOSTART="uk2161"' /etc/default/openvpn
```
Lastly, reboot
```bash
reboot
```
on boot, lets check our ip and country and see if our vpn has connected. If this hasn't worked, try again with another server (some of the sed commands will need modified, as the files will have changed from default, i.e. /etc/default/openvpn)
```bash
curl ifconfig.co ; curl ifconfig.co/city ; curl ifconfig.co/country
```

## - FIREWALL RULES AND KILLSWITCH

Ok, i'll get this out of the way first. Wireguard clients (mobile devices/desktops) themselves don't have a killswitch but the wireguard application tends to hang your connection BUT there are ways to check. I've never tested how long this hangs for, but it seems permanent until intervention occurs.

Android devices : Connections > More connection settings > VPN > settings icon > Always-on VPN & Block connections without VPN.

Windows devices : Not sure, but like i say, the wireguard app already hangs your connection. You could run wireshark and run some packet capture, or a traceroute alternative for windows, or ping 8.8.8.8 should confirm 100% packet loss.

Linux Desktop : Once you're finished this tutorial, and have linux desktop connected. Shutdown the vpn-gateway, run the below command, nothing appears to escape. If you run this command before and after you'll see what it should look like. You can also try ping, and it will confirm 100% packet loss, and also tcpdump.
```bash
traceroute 8.8.8.8
```
or
```bash
ping -c1 8.8.8.8
```
or
```bash
sudo tcpdump
```
I usually just get a bunch of arp requests looking for the vpn-gateway.

Ok back to the VPN-Gateway Firewall rules & Killswitch

Make sure your root
```bash
sudo su -
```
Lets clone this repo on the gateway, change directory, and provide necessary permissions for the scripts.
```bash
mkdir -p $HOME/git && cd $HOME/git
git clone https://github.com/jonnypeace/VPN-gateway.git && cd VPN-gateway
chmod 700 ipRes.sh openvpn.sh
```

I've put in some firewall rules that can be copied to /etc/iptables
This might not work well with UFW, i find it better to keep things simple and choose one over the other.
If you are using ufw, you might want to disable it, and flush the iptable rules, but for the purpose of this tutorial,
i'll assume fresh install of Ubuntu with ufw disabled (and never been run).

We'll need to edit the rules to match your system and not lock you out, so...
replace ens18 with your network interface, which can be found using...

run this command:
```bash
ip route list table main default
```

snippet of the output:
```
default via 192.168.1.1 dev <b>ens7</b> proto dhcp src <b>192.168.1.111</b> metric 100
```

if this doesn't work, use..
```bash
ip a
```
snippet of ip a
```
2: <b>ens7</b>: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:12:34:56 brd ff:ff:ff:ff:ff:ff
    inet <b>192.168.1.111/24</b> brd 192.168.2.255 scope global ens7
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe12:3456/64 scope link 
       valid_lft forever preferred_lft forever
```

replace "EDITME" with the correct interface on your system.. i.e. the output from above command was ens7
```bash
sed -i 's|ens18|<b>EDITME</b>|g' etc.iptables.rules.v4
```
If my interface is ens7
```bash
sed -i 's|ens18|ens7|g' etc.iptables.rules.v4
```

WARNING: This is important to get right, as this rule whitelists your Lan from the firewall, and will allow SSH/wireguard access

SUBNET HELP: A subnet from the ip address identified from the command above (in bold), would equate to...
```
192.168.1.111/24 = 192.168.1.0/24 or 192.168.0.0/16 (/16 will provide more addresses than /24)

For a better understanding, maybe have a look here. https://www.cloudflare.com/en-gb/learning/network-layer/what-is-a-subnet/

Home lans usually fall somewhere in the 192.168.0.0/16 range.
```
Now lets replace the "EDITME" (in this sed command) with your lan address subnet. If you're happy with 192.168.0.0/16, then skip this part, as no sed edit will be necessary
```bash
sed -i 's|192.168.0.0/16|<b>EDITME</b>|g' etc.iptables.rules.v4
```

We need the nordvpn IP address we're connecting to in our uk config file. Remember to use the name of the config you chose.
```bash
grep "^remote" /etc/openvpn/uk2161.conf | awk 'NR==1{print $2}'
```

Now use this ip to replace "EDITME" 
```bash
sed -i 's|123.123.123.123|<b>EDITME</b>|g' etc.iptables.rules.v4
```

rules should be good now, so copy them to the correct directory, and correctly labelled.
```bash
mkdir -p /etc/iptables
cp etc.iptables.rules.v4 /etc/iptables/rules.v4
```

If we make these rules live, we can make them persistent afterwards, so, lets run the script provided and choose the
default N (or just press enter without typing anything).
WARNING - If your LAN address is wrong, this will lock you out of SSH
```bash
./ipRes.sh
```
Check rules are in place
```bash
iptables -vL --line-numbers
```

Try pinging google & checking our IP
```bash
ping -c1 google.com
curl ifconfig.co ; curl ifconfig.co/city ; curl ifconfig.co/country
```

If the ping was successful install iptables-persistent & follow the screen prompt. It will ask to save your current iptables, and it should be fine to do so.
```bash
apt install iptables-persistent
```

## - CRONTAB TO MONITOR CONNECTION AND RESTART SERVICE IF IT DROPS

Ok, the openvpn service connection can drop from time to time on the VPN-gateway, so lets monitor it and restart the service to keep it live.
First, lets copy the script into root home directory, we should still be in the vpn-gateway github directory and still be root.
```bash
cp openvpn.sh /root/
```
Modify the systemctl service in this script to match yours "EDITME". Ignore if you've decided to use the same config as me.
```bash
sed -i 's|openvpn@uk2161.service|openvpn@EDITME.service|g' /root/openvpn.sh
```
lets open crontab
```bash
crontab -e
```
At the bottom, type in something like this so the script checks if the connection is live each minute.
```bash
* * * * * /root/openvpn.sh
```
Look here https://crontab.guru/every-5-minutes for an easy way to schedule your cron jobs.

## - WIREGUARD CLIENT LINUX DESKTOP

Make sure wireguard and resolvconf are installed
```bash
sudo apt install wireguard resolvconf
```
With your user config that you copied/pasted the contents, or were able to copy from the server directly, lets move this into the Desktop wireguard config directory and get this going. Lets assume you copied this .conf file into your desktop home directory.
```bash
sudo mv $HOME/EDITME.conf /etc/wireguard/wg0.conf
```
An easy way to bring the connection up or down
```bash
sudo wg-quick down wg0
sudo wg-quick up wg0
```
This command shows the status
```bash
sudo wg
```

If everything is working up to this point, we should be good.
You can test further for DNS leaks. For the client side, I often use https://www.dnsleaktest.com/

There are ways which i'll let you search online, for checking dnsleaks in the terminal.

- VPN-Gateway SERVER SIDE TROUBLESHOOTING

First thing to try out of everything else, is a system reboot
```bash
reboot
```
We are going to use ping a lot, so best to start off with checking DNS so you know what to check as we troubleshoot.

If this works
```bash
ping -c1 8.8.8.8
```
and this doesn't
```bash
ping -c1 google.com
```

then it's probably DNS

If its your DNS and all you've done is follow this readme, take a look at the yaml / netplan part of this readme. Worst comes to worst, return the yaml to DHCP, and remove the nameservers, addresses and gateway
```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens3:
      dhcp4: yes
  version: 2
```

Check your VPN-gateway systemctl service. Replace uk2161 with the server config you chose above.
Some commands to try below
```bash
systemctl status openvpn@uk2161.service
systemctl enable openvpn@uk2161.service
systemctl start openvpn@uk2161.service
systemctl restart openvpn@uk2161.service
```
If the service is active & enabled it could be your firewall rules.

This will flush your rules.
```bash
iptables -F
```
Try ping now
```bash
ping -c1 google.com
ping -c1 8.8.8.8
```
If there are still issues after iptables flushed, then there's maybe a problem with the openvpn config/credentials, or nordvpn server
Try stopping and disabling the service while the firewall rules are flushed.
```bash
systemctl stop openvpn@uk2161.service
systemctl disable openvpn@uk2161.service
ping -c1 google.com
ping -c1 8.8.8.8
```
If this works, you could go through the start of this readme and try again, with another nordvpn server. 

If problems persist we can also flush iptables then save using the script provided (enter y at the prompt) so you start over, and upon reboot, you will have no firewall rules.
```bash
systemctl disable openvpn@uk2161.service
iptables -F
./ipRes.sh
reboot
```
This should be enough to clear everything and bring back your internet connection.

Hope all went well & Happy VPN-Gatewaying :)
