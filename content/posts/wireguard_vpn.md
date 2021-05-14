---
title: "Wireguard vpn"
date: 2021-05-05T16:56:58-05:00
categories:
- homelab
tags:
- wireguard
- vpn
- networks
series:
- idk
draft: false
---
[//]: # ( {{< series "idk" >}} )

# Setup

## Create keys
Run the following commands for every peer
```bash
wg genkey | tee privatekey | wg pubkey > publickey
```
Then set perms so that only the user that created them can read/write them
```bash
sudo chmod 600 *key
```

## Allow ip forwarding on VPN server
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
**(untested) Note:** To make the change permanent, add `net.ipv4.ip_forward = 1` to /etc/sysctl.d/99-sysctl.conf.


## Create config files for interface using these keys

**1) Create the following files on each machine**
```bash
sudoedit /etc/wireguard/wg0.conf
```

<details>
  <summary>VPN Client's wg0.conf</summary>
<p>
  
```
[Interface]
# The IP address that this client will have on the WireGuard network.
# Note the /32 means this client can only have one address within the tunnel
# Whereas the server has a /24 since it can use any 
Address = 10.101.0.2/32

# The private key you generated for the client previously.
PrivateKey = :)


[Peer]
# VPN Server's public key.
PublicKey = public !

# The WireGuard server to connect to.
# Note: this address must be reachable on the client's network
Endpoint = 192.168.0.20:51821

# The subnets this WireGuard VPN is in control of.
# Implication ==> this client's (client who has this file) ping's to these subnets will go through this peer's endpoint instead of another gateway
# Implication ==> This client   (meaning ^) can access these subnet's if this wireguard vpn server can access them !
AllowedIPs = 10.101.0.0/24, 10.0.50.0/24

# Ensures that your home router does not kill the tunnel, by sending a ping
# every 25 seconds.
PersistentKeepalive = 25
```

</p>
</details>

<details>
  <summary>VPN Server's wg0.conf </summary>
<p>
  
```
[Interface]
# Configuration for the server
# Set the IP subnet that will be used for the WireGuard network.
Address = 10.101.0.1/24

# The port that will be used to listen to connections. 51820 is the default.
ListenPort = 51821

# The output of `wg genkey` for the server.
PrivateKey = :)


# FROM: https://wiki.archlinux.org/index.php/WireGuard#Server
# Apperently (from ^^) if the server is behind a router and receives traffic via NAT, these iptables rules are not needed
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o CHANGE_ME:eth0.50 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptable s -t nat -D POSTROUTING -o CHANGE_ME:eth0.50 -j MASQUERADE



[Peer]

# Connecting client's public key
PublicKey = public !

# The IP address that this peer has WITHIN the tunnel
AllowedIPs = 10.101.0.2/32

# Ensures that your home router does not kill the tunnel, by sending a ping every 25 seconds.
PersistentKeepalive = 25
```

</p>
</details>

**2) `wg-quick up wg0` on both machines and you're done**

# Q/A
[//]: # ( **Q: I'm confused, I keep hearing the terms clients and servers and peers interchanged but WHO IS WHO ARENT THEY ALL PEERS??** )
### Q: I'm confused, I keep hearing the terms clients and servers and peers interchanged but WHO IS WHO ARENT THEY ALL PEERS??
- **A:** Yes they are all peers but we can mimic a client-server model using wireguard which is what is shown here :)

### Q: What would I use this dang thing for anyways
- **A:** For example, people who want remote access to some computer in their house might have:
1) A machine with the servers config file on their private (no open ports) home network **(tweaked so this server also has the clients endpoint in its config file)**
2) And a publicly acessible Amazon EC2 instance with the client configuration.
- As long as PersistentKeepalive is set and a connection is established by the "server" to the EC2 "client" (established by ping or any communication then PersistentKeepalive makes the connection stay active) then we can connect to the publicly innaccessable "server" by connecting to the public EC2 "client" :)

### Q: Where can I find further resources for wireguard or VPN's in general
- **A:** [The arch wiki](https://wiki.archlinux.org/title/WireGuard#Specific_use-case:_VPN_server) is always a great resource, [the talk](https://www.youtube.com/watch?v=88GyLoZbDNw&t=523s) that Jason Donenfeld gave to introduce wireguard provides an interesting not-overly-technical analysis of wireguard. Lastly, the [wireguard website](https://www.wireguard.com/#conceptual-overview) has lots of information for those who wish to deep dive into the inner workings of wireguard.
