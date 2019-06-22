The html file was located in /var/www/html/index.nginx-debian.html

/usr/share/nginx/html/index.html

/


# Trying to get the second cluster to work

We have the master/management node. I plugged in the 2 ethernet cords to the existing cluster's
management and k8 (internet) network.

DHCP only runs on the K8 network (10.0.0.0/8). When I tried leaving out the blue wire to the K8 network,
DHCP would not run on boot. The netboot thing only works when you install it from the command line.

I booted up the new master as if it was another node, by typing `cli` into the command line
when DHCP prompted at `boot:` (pxe)

Then Ubuntu installed using the preseed.

On the console, which was connected to the new master, I typed `ip addr` to show the networking information.
It showed:

```
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:25:90:53:4e:ae brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.14/24 brd 10.0.0.255 scope global dynamic enp1s0
       valid_lft 553sec preferred_lft 553sec
    inet6 fe80::225:90ff:fe53:4eae/64 scope link
       valid_lft forever preferred_lft forever
```
This indicates that the 10.0.0.14 is the IP address of the new master, in the enp1s0 interface (the K8
network). 

I SSHed in to the new master: `ssh 10.0.0.14`, to make sure that it worked.

I used `scp` to transfer `/srv/tftp/pxelinux.cfg/default` and `/srv/tftp/preseed.cfg` to a new directory
in the new master called `tftpfiles`.

I installed git and cloned the metalc repo. Also installed vim.

In the network switch,
```
screen /dev/ttyS0

awplus>

awplus# enable 
awplus# config t
awplus    int vlan1
          ip address 192.168.1.1/24 # this is the management network


awplus>

awplus# enable 
awplus# configure terminal
awplus(config)# vlan database
awplus(config-vlan)# vlan 1-2 state enable
awplus# exit

awplus# interface port2.0.1-port2.0.12
awplus(config-if)# switchport access vlan 1

awplus# interface port2.0.13-port2.0.24
awplus(config-if)# switchport access vlan 2
```

The command `awplus# show vlan all` shows the configuration

copy running-config startup-config


Edit netplan config file

192.168.1.1/24 is the management
10.0.0.1/8 is the K8 network


Fill in the kubernetes and management networks. 
One of the interfaces will be connected to the Internet, for us it was enp1s0.
Leave that as is in the config file.

Kubernetes network: enp2s0
Management network: enp3s0

Then we follow: steps on rooster (i.e. commands to run on rooster to get netboot to work)
from the bare-metal.md

`sudo apt install policykit-1`
for when I was restarting dhcp-server after modifying config file

This doesn't work:
`sudo apt install ifupdown`
`sudo ifup wlan0`

Then `sudo netplan


