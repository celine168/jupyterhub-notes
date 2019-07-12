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

http://192.168.1.1/gui/ showed a nice graphical interface of the switch.
It needed Java and could only open on Internet Explorer though.

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

Then `sudo netplan apply`

To find entry for gateway4, do `ip route | grep default` and replace the IP address.
I.e., 
ip route gives
```
default via 128.120.136.1 dev enp2s0 proto static
```
The default gateway is the private IP address assigned to the router that is used to 
communicate with the local network. It lets a device on one network communicate
with devices on another (such as the Internet), i.e., it's a *gateway*.
https://www.lifewire.com/what-is-a-default-gateway-817771
https://www.lifewire.com/how-to-find-your-default-gateway-ip-address-2626072

```
gateway4: 128.120.136.1
```

https://netplan.io/reference Netplan reference

I have come to the conclusion that the WiFi USB adapter is not compatible with 
the Linux kernel. Plan on getting a new one.

So I got a new USB adapter.
When you type ifconfig, there's a new interface that pops up.
```
wlx9cefd5fd7ff9: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 9c:ef:d5:fd:7f:f9  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

I figured out how to download packages using a USB, thanks to 
[Stack Overflow](https://askubuntu.com/questions/711890/installing-packages-from-usb-to-ubuntu-server-14-04).

I downloaded the `wireless-tools` package.

I used [these instructions](https://ubuntuforums.org/showthread.php?t=1106353) to connect to the WiFi. (instrs
are in the second post of that thread. I didn't know how to get dhclient3 to work, but I don't 
think I really needed it). 
1. `iwconfig`: You should see a new interface pop up. Mine was called `wlx9cefd5fd7ff9`.
1. `sudo iwlist wlx9cefd5fd7ff9 scan`: Find the network you want to use.
1. `sudo iwconfig wlx9cefd5fd7ff9 essid <Network name you want to use>

Also in `/etc/netplan/01-netcfg.yaml`, according to the 
[netplan reference](https://netplan.io/examples#connecting-to-an-open-wireless-network)
I added this:
```
wifis:
  wlx9cefd5fd7ff9:
    dhcp4: yes
    access-points:
      SVPMeterConnectWiFi: {}
```

`SVPMeterConnectWiFi` is the name of the open WiFi network I am connected to.

Also, for the vlans, I added this to `/etc/netplan/01-netcfg.yaml`, according to the 
[netplan reference](https://netplan.io/examples#attaching-vlans-to-network-interfaces)
```
vlans:
  vlan1:
    id: 1
    link: enp1s0
    addresses: ["192.168.1.1/24"]
  
  vlan2:
    id: 2
    link: enp2s0
    addresses: ["10.0.0.1/8"]
```

So it turns out in `/etc/default/isc-dhcp-server`, I had to change b/c I made the management network 
to `enp1s0`. 
```
INTERFACESv4="enp1s0"
```
This is the interface that the DHCP server serves DHCP requests to. I guess it's the management network
because that's how all the nodes are connected to the master, so they get assigned IP addresses that way.

It's not enp3s0 because I don't think I have enp3s0. Just doesn't show up on `ifconfig`.

So now I can finally use the `ping` command and the wifi adapter lights up. The only problem is that
when I ping, nothing comes back.

