---
title: "VLAN Setup Guide cisco SG250"
date: 2021-05-23T21:39:29-05:00
draft: true
---

[//]: # ( todo: create a table of contents )
[//]: # ( here, link to engineers worksop for how to create vlans on raspberry pi? )
[//]: # ( and the link for setting it up on proxmox??? )

# A world without VLANs
The traditional explanation of a VLAN requires explaining why they were needed in the first case: to feasibly scale a physical network.  
Before VLANs, each ethernet port on a network switch could belong to only one subnet, hence the inability to scale.
This meant that a corporation would need many switches in order to have a segmented network.  
If this doesn't seem valuable or doesn't make sense yet, read the next section instead of re-reading this one.

# A better world (has VLANs, which you love)
VLANs have many benefits but the one that makes them essential is that they allow each port on a switch to handle traffic from many different subnetworks/VLANs (called a VLAN if there are multiple networks on one port).  
In other words, a machine with one ethernet port can host many VM's on many different networks which all share the same port.
This was unfeasible in a world without VLANs since this machine would need an ethernet port for every different subnet used by the machine (need many switches also).

# Other VLAN benefits

1. Security through network isolation
2. Easily, feasibly segregate subnets
3. Helps with sanity (see previous point)


# Setup on Cisco SG 250
## Preamble
### Why this model? Aren't there plenty of cisco VLAN guides already?
The reason I made this guide is because the SG 250 doesn't use the same command line interface as most cisco switches, which renders guides for those switches less helpful.  
I believe that most cisco switches have the same webUI however so this guide may work for other cisco switches.

### What is the end goal here?
To create VLANs to seem smart to attract a foolish mate duh.  
JK nobody is that foolish except me ...  

### Current Network Topology
- My laptop is connected to my switch which is connected to my router.
- I also have a raspberry pi connected to this switch.
- They are currently on the same network and can communicate with each other
- All packets between these two machines are routed via my router.

### Goal Network Topology
- Laptop and raspberry pi each receive second IP address on to-be-created subnet: 10.0.30.0/24
- Traffic on this subnet is routed through layer 3 cisco switch.
- Traffic does not reach the router

