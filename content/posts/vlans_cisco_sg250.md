---
title: "VLAN Setup Guide Cisco SG251"
date: 2021-05-23T21:39:29-05:00
draft: false
categories:
- homelab
---

[//]: # ( todo: create a table of contents )
[//]: # ( here, link to engineers worksop for how to create vlans on raspberry pi? )
[//]: # ( and the link for setting it up on proxmox??? )
[//]: # ( try making port 8 not able to use vlan 30 by going to vlan management -> port to vlan -> filter vlan id = 30 )


[//]: # ( to research: wtf is switchport mode layer 3 )
[//]: # ( and what is gneral/customer modes )
[//]: # ( -- on the vlan management -> interface settings )

[//]: # ( why can't I make traffic route from 1 vlan to another via weird interfaces???? )
[//]: # ( Why does ping from my laptop to 10.0.30.99 not work but 10.0.30.20 does ????? )

[//]: # ( big source of confusion: why do I need ACL's if my no networks can communicate with each other already?? )



# Some background on VLANs

## A world without VLANs
VLANs can be confusing and seem to be all-powerful, I find it helps to think about why they were needed in the first case: **to feasibly scale a network.**  
Before VLANs, each ethernet port on a network switch could only belong to **one** subnet :'(
This meant that a corporation would need many switches in order to have a segmented network.  
If this doesn't seem valuable or doesn't make sense yet, read the next section instead of re-reading this one.

## A better world (has VLANs)
VLANs have many benefits but the one that makes them essential is that they allow switch ports to handle traffic from many different subnetworks/VLANs (called a VLAN if there are multiple networks on one port).  
In other words, VLANs allow a machine with one ethernet port to host many VM's on many different networks which all share the same port.  
This was not feasible without VLANs since this machine would need an ethernet port for every different subnet used by the machine (and many switches!!).  

## Other VLAN benefits

1. Security through network isolation
2. Easily, feasibly segregate subnets
3. Helps with sanity (see previous point)


# Setup on Cisco SG 250
## Preamble
[//]: # ( ### What is the end goal here? )
[//]: # ( To create VLANs to seem smart to attract a foolish mate duh. )  
[//]: # ( JK nobody is that foolish except me ... )  

### Current Network Topology
- My laptop is connected to my switch which is connected to my router.
- I also have a raspberry pi connected to this switch.
- They are currently on the same network and can communicate with each other.
- All packets between these two machines are routed via my router.

### Goal Network Topology
- Laptop and raspberry pi each receive second IP address on to-be-created subnet: 10.0.30.0/24.
- Traffic on this subnet is routed through layer 3 cisco switch.
- Traffic on this subnet does not reach the router.

## Setup
1. **Access WebUI: https://switch_ip_address**  
**Note:** I found https to be significantly slower than http!
* * *
**2. Set laptop and raspberry pi ports on switch to trunk mode from access mode**  
\
This makes the port able to use multiple subnetworks (VLANs).
This is because access ports may only use the native VLAN, while trunk ports may use any number of configured VLANs.
- Click: VLAN Management then Interface Settings
- Click: Edit for each interface the raspberry pi and laptop are connected to.
![Create trunk port](/make_trunk_port.png)
* * *
**3. Click: VLAN Management then VLAN Settings then Add**
![Add VLAN](/add_vlan.png)
* * *
**4. Enter VLAN ID and the VLAN name in the popup window.**
![Add VLAN](/adding_vlan_popup.png)
* * *
**5. Enable IPV4 routing (Layer 3 functionality)**
- Click: IP Configuration -> IPv4 Management and Interfaces
- Make sure box is checked like in image below
![Enable IPV4](/enable_ipv4.png)
* * *
**6. Assign default gateway for this VLAN**  
From same screen as previous step  
- Click: Add
- Set the interface to VLAN 30
- Set the IP Address Type to static
- Define the VLAN subnet and default gateway by entering the IP address of the gateway and the subnet mask.  
Note that it is convention that VLAN XX be on network aa.bb.XX.0/24 or aa.XX.0.0/16.
![Create static route](/create_static_route.png)
* * *

## Testing
### Give devices IP addresses on new VLAN
First I add my raspberry pi to the new VLAN using [this guide](https://engineerworkshop.com/blog/raspberry-pi-vlan-how-to-connect-your-rpi-to-multiple-networks/)  
- It's IP address is 10.0.30.10

Do the same on laptop according to [arch wiki](https://wiki.archlinux.org/title/VLAN)
```
sudo ip link add link enp0s20u2u2 name enp0s20u2u2.30 type vlan id 30
sudo ip addr add 10.0.30.20/24 brd 10.0.30.99 dev enp0s20u2u2.30
sudo ip link set dev enp0s20u2u2.30 up
```
- It's IP address is 10.0.30.20

### Test the devices can communicate
Test laptop can reach raspberry pi:
```
ping -I enp0s20u2u2.30 10.0.30.10
```

Test raspberry pi can reach laptop:
```
ping -I eth0.30 10.0.30.20
```

# Q/A
### Aren't there plenty of cisco VLAN guides already?
Yes there are (strangely less than expected), but the SG 250 doesn't use the same command line interface as most cisco switches, which renders those guides not so helpful.  
I believe that most cisco switches have the same webUI however so this guide may work for other cisco switches.

[//]: # ( The reason I made this guide is because the SG 250 doesn't use the same command line interface as most cisco switches, which renders guides for those switches less helpful. )

### Why can't my devices connect to the internet via the VLAN interface?  
Because your router does not know where to send the return traffic for the subnetwork: 10.0.30.0/24.  
**Solution:** Add a static route on your router where all traffic destined for the subnet 10.0.30.0/24 is sent to the default gateway for this VLAN: 10.0.30.99

[//]: # ( ### How do I stop other subnetworks from communicating with each other? )
[//]: # ( Access Control Lists (ACL's))  
[//]: # ( **Note:** I am a noob) 

