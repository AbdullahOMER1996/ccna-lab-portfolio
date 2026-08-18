# Week 4: DHCP and NAT/PAT

**Date:** 16-Aug-2026

**Topology file:** `topology-2-dhcp-services.pkt`

---

## Topics Covered

- DHCP server configuration on a Cisco router
- DHCP pool, network, default-router, and DNS server setup
- Static NAT (one-to-one inside/outside mapping)
- Dynamic NAT with PAT (overload) for internal LAN internet access

---

## Lab 1: DHCP Server Configuration

**Router config:**
```
ip dhcp pool INSIDE_POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
```

**What I learned:**
- How `network` and `default-router` combine to define the scope and gateway for a pool
- How to switch a PC from static to DHCP and verify it receives an address, mask, and gateway automatically
- How to verify active leases and pool status from the router side

**Verification:**
```
show ip dhcp binding
show ip dhcp pool
```

PC(s) confirmed to receive correct IP, subnet mask, default gateway, and DNS server automatically via Packet Tracer's Desktop → IP Configuration showing DHCP-assigned values.

**Screenshots:** [reference your DHCP screenshots — pool config, PC IP config showing DHCP, `show ip dhcp binding` output]

---

## Lab 2: NAT / PAT

**Topology context:** Inside router connects to an ISP router. ISP's Gig0/0/1 (203.0.113.2/24) is the inside router's next hop — this is the address used as the default route target and NAT outside gateway.

**ISP Router config:**
```
interface GigabitEthernet0/0/0
 ip address 198.51.100.1 255.255.255.0
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/1
 ip address 203.0.113.2 255.255.255.0
 duplex auto
 speed auto
```

**Inside Router config:**
```
ip nat pool DYNAMIC_POOL 203.0.113.20 203.0.113.21 netmask 255.255.255.0
ip nat inside source list 1 pool DYNAMIC_POOL overload
ip nat inside source static 192.168.1.20 203.0.113.10
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

**What this does:**

| Line | Purpose |
|---|---|
| `ip nat pool DYNAMIC_POOL 203.0.113.20 203.0.113.21` | Defines a small pool of public IPs for dynamic NAT |
| `ip nat inside source list 1 pool DYNAMIC_POOL overload` | PAT — overloads the pool so many inside hosts share those public IPs via port translation |
| `ip nat inside source static 192.168.1.20 203.0.113.10` | Static one-to-one NAT — always maps 192.168.1.20 to 203.0.113.10, e.g. for a server needing a fixed public address |
| `ip route 0.0.0.0 0.0.0.0 203.0.113.2` | Default route pointing to the ISP router's Gig0/0/1 (203.0.113.2) |

**Network layout:**

| Segment | Network | Role |
|---|---|---|
| Inside LAN | 192.168.1.0/24 | Private, NAT'd |
| NAT outside pool | 203.0.113.20–.21/24 | Public IPs for dynamic PAT |
| NAT static mapping | 192.168.1.20 ↔ 203.0.113.10 | Fixed 1:1, e.g. internal server |
| Inside-to-ISP link | 203.0.113.0/24 | Between inside router and ISP Gig0/0/1 |
| ISP outside link | 198.51.100.0/24 | ISP's "internet-facing" side (Gig0/0/0) — simulates the public internet |

**What I learned:**
- The difference between static NAT (fixed 1:1 mapping, good for servers) and dynamic PAT/overload (many-to-few, good for general LAN internet access)
- How the inside router's default route and NAT outside pool both point toward the ISP router, and how 198.51.100.0/24 on the ISP's other interface represents the simulated "internet" side beyond NAT
- NAT/PAT table entries expire almost immediately after a ping in Packet Tracer — must check `show ip nat translations` right after generating traffic to catch the entry before it clears

**Verification:**
```
show ip nat translations
show ip nat statistics
```

**Screenshots:** [reference your NAT/PAT screenshots]

---

## Packet Tracer Limitations Found

- The `lease` command (used on real IOS to set custom DHCP lease duration) is not supported in Packet Tracer's DHCP pool configuration.
- Dynamic NAT/PAT table entries expire almost immediately after a ping — verification requires checking `show ip nat translations` immediately after generating traffic.

---

## Key Commands Learned

| Command | Purpose |
|---------|---------|
| `ip dhcp pool [name]` | Create a named DHCP pool |
| `network [addr] [mask]` | Define the scope/subnet for the pool |
| `default-router [ip]` | Gateway handed out to DHCP clients |
| `dns-server [ip]` | DNS server handed out to DHCP clients |
| `show ip dhcp binding` | View active DHCP leases |
| `ip nat pool [name] [start-ip] [end-ip] netmask [mask]` | Define a NAT address pool |
| `ip nat inside source list [acl] pool [name] overload` | Dynamic NAT with PAT |
| `ip nat inside source static [inside-ip] [outside-ip]` | Static 1:1 NAT |
| `show ip nat translations` | View active NAT table entries |
| `show ip nat statistics` | View NAT translation counters |

---

## What I Will Learn Next

- NTP, Syslog, and SNMP (see separate README)

---

*Last updated: 16-Aug-2026
