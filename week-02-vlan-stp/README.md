# Week 2: VLANs, Inter-VLAN Routing, STP/RSTP, EtherChannel

**Date:** 26-07-2026 - 01-08-2026

---

## Topics Covered

- VLAN creation and port assignment
- Inter-VLAN routing using Router-on-a-Stick
- Spanning Tree Protocol (STP) and Rapid STP (RSTP)
- EtherChannel with LACP

---

## Lab 1: VLAN Basics

**File:** `topology-1/topology-2-vlan-basics.pkt`

**VLANs configured:**
- VLAN 10: Helpdesk
- VLAN 20: Network
- VLAN 30: Sales

**What I learned:**
- How to create VLANs and assign access ports
- Same VLAN = direct communication
- Different VLAN = blocked at switch level

**Screenshots:** See `topology-1/Screenshots/`

---

## Lab 2: Inter-VLAN Routing

**File:** `topology-2/topology-2-inter-vlan-routing.pkt`

**Method:** Router-on-a-Stick

**Router subinterfaces:**
- G0/0.10: 192.168.10.1 (VLAN 10)
- G0/0.20: 192.168.20.1 (VLAN 20)
- G0/0.30: 192.168.30.1 (VLAN 30)

**What I learned:**
- Trunk ports carry multiple VLANs
- Router subinterfaces with 802.1Q encapsulation
- Inter-VLAN routing verification

**Screenshots:** See `topology-2/Screenshots/`

---

## Lab 3: STP and RSTP

**File:** `topology-3/topology-3-stp.pkt`

**What I learned:**
- STP prevents Layer 2 loops
- Root Bridge election (lowest priority/MAC)
- Port roles: Root, Designated, Alternate/Blocked
- RSTP for faster convergence

**Screenshots:** See `topology-3/Screenshots/`

---

## Lab 4: EtherChannel

**File:** `topology-4/topology-4-etherchannel.pkt`

**Protocol:** LACP

**What I learned:**
- Bundle multiple links into one logical link
- No STP blocking with EtherChannel
- Full bandwidth utilization + instant failover

**Screenshots:** See `topology-4/Screenshots/`

---

## Key Commands Learned

| Command | Purpose |
|---------|---------|
| `vlan [id]` | Create VLAN |
| `name [name]` | Name VLAN |
| `switchport access vlan [id]` | Assign port to VLAN |
| `switchport mode trunk` | Configure trunk port |
| `interface g0/0.[id]` | Create subinterface |
| `encapsulation dot1Q [id]` | 802.1Q tagging |
| `spanning-tree vlan [id] priority [value]` | Set STP priority |
| `spanning-tree mode rapid-pvst` | Enable RSTP |
| `channel-group [id] mode active` | LACP EtherChannel |
| `show etherchannel summary` | Verify bundle |

---

## What I Will Learn Next Week

- OSPF multi-area deep dive
- EIGRP
- BGP basics

---

*Last updated: 01-08-2026*