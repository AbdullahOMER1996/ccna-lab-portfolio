# Week 1: Network Fundamentals

**Date:** 25-July-2026

---

## Topics Covered

- OSI Model and TCP/IP Model overview
- IPv4 addressing and subnetting
- Basic LAN design and device connectivity
- Static routing between multiple subnets
- IP configuration on end devices (PCs, servers)
- Basic router interface configuration

---

## Lab 1: Basic LAN Topology

**File:** `Topology 1/topology-1-basic-lan.pkt`

**Devices:** 2 PCs, 1 Switch (2960), 1 Router (2911), 1 Server

**Network:** 192.168.1.0/24

| Device | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|-------------|-----------------|
| PC0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC1 | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| Server0 | 192.168.1.20 | 255.255.255.0 | 192.168.1.1 |
| Router G0/0 | 192.168.1.1 | 255.255.255.0 | — |

**What I Learned:**
- How to connect devices with correct cable types (Copper Straight-Through for PC-to-Switch, Switch-to-Router)
- How to assign static IP addresses, subnet masks, and default gateways
- How to configure router interfaces with `ip address` and `no shutdown`
- How to verify connectivity using `ping` and `tracert`
- In a single subnet, all devices communicate directly through the switch — the router is only the gateway

**Verification:** All devices can ping each other bidirectionally with 0% packet loss.

**Screenshots:** See `Topology 1/Screenshots/`

---

## Lab 2: Multi-Subnet Topology with Static Routing

**File:** `Topology 2/topology-2-multi-subnet.pkt`

**Devices:** 4 PCs, 2 Switches (2960), 2 Routers (2911), 1 Server

**Networks:**

| Subnet | Network | Devices |
|--------|---------|---------|
| Subnet 1 | 192.168.1.0/24 | PC0 (.15), PC1 (.16), R1 G0/0 (.1) |
| Subnet 2 | 192.168.2.0/24 | PC2 (.15), PC3 (.16), R1 G0/1 (.1) |
| Subnet 3 | 192.168.3.0/24 | Server0 (.15), R2 G0/0 (.1) |
| WAN Link | 10.0.0.0/30 | R1 G0/2 (.1), R2 G0/2 (.2) |

**Router Configurations:**

**R1 Static Route:**
ip route 192.168.3.0 255.255.255.0 10.0.0.2

**R2 Static Routes:**
ip route 192.168.1.0 255.255.255.0 10.0.0.1
ip route 192.168.2.0 255.255.255.0 10.0.0.1


**What I Learned:**
- How to design a network with multiple subnets
- How to configure static routes on routers to reach remote networks
- How traffic flows between subnets through multiple routers
- How to use `show ip route` to verify routing tables (C = Connected, S = Static)
- How to use `tracert` to trace the full path across routers

**Verification:** PC in Subnet 1 can ping Server in Subnet 3 via 2 routers. Tracert shows 3 hops: R1 → R2 → Server.

**Screenshots:** See `Topology 2/Screenshots/`

---

## Subnetting Practice

**Tool used:** subnettingpractice.com

**Status:** In progress

**Key concepts mastered:**
- Block size calculation: 256 - mask value
- Network address: count by block size until passing the IP
- Broadcast address: next network - 1
- Usable hosts: network + 1 to broadcast - 1

---

## Key Commands Learned

| Command | Purpose |
|---------|---------|
| `enable` | Enter privileged EXEC mode |
| `configure terminal` | Enter global configuration mode |
| `hostname [name]` | Set device hostname |
| `interface [type] [number]` | Enter interface configuration |
| `ip address [ip] [mask]` | Assign IP to interface |
| `no shutdown` | Enable interface |
| `ip route [network] [mask] [next-hop]` | Add static route |
| `show ip interface brief` | View interface status |
| `show ip route` | View routing table |
| `ping [ip]` | Test connectivity |
| `tracert [ip]` | Trace route path |
| `copy running-config startup-config` | Save configuration |

---

## Challenges Faced

- **Topology 1:** Initially tried to put both router interfaces in the same subnet (192.168.1.1 and 192.168.1.2), which caused an IP overlap error. Fixed by using only G0/0 and connecting the server to the switch instead.
- **Topology 2:** First ping attempts showed 25% packet loss due to ARP resolution. Second attempts were successful with 0% loss — this is normal behavior.

---

## What I Will Learn Next Week

- VLANs and trunking (802.1Q)
- Inter-VLAN routing (Router-on-a-stick and Layer 3 switching)
- Spanning Tree Protocol (STP/RSTP)
- EtherChannel (LACP)

---

*Last updated: [Date]*