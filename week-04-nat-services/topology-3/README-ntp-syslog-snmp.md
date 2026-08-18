# Week 4 (Part 3): NTP, Syslog, and SNMP

**Date:** 16-18 - Aug-2026

**Topology file:** `topology-3-ntp-syslog-snmp.pkt`

**Base topology:** Reused the Week 3 OSPF multi-area topology (R1/R2/R3, 3 areas, serial WAN links) — no changes to routing config, only services added on top.

---

## Topics Covered

- NTP client/server synchronization across a multi-hop OSPF network
- Centralized Syslog logging from multiple routers to a server
- SNMPv2c configuration (community strings, read-only access)
- SNMPv3 — theory only (not configurable in Packet Tracer, see Limitations)

---

## Lab 1: NTP

**Setup:** R1 configured as NTP master (stratum source), R2 and R3 as NTP clients synced across the OSPF-routed WAN links.

**What I learned:**
- How to configure an NTP master and clients
- Root delay increases with hop count — R3 (2 hops from R1) shows a higher root delay than R2 (1 hop from R1), proving synchronization is happening across the actual routed path, not just locally

**Verification:** R2 and R3 both show synced status with correct root delay values reflecting their distance from the master.

**Screenshots:** [reference your NTP screenshots]

---

## Lab 2: Syslog

**Setup:** Added a switch and Server-PT to R1's existing LAN segment (10.0.0.0/24), alongside PC0. No new subnet or OSPF network statement needed. Server configured at 10.0.0.20/24, Syslog service enabled.

**Router config (all 3 routers):**
```
logging host 10.0.0.20
```

**What I learned:**
- Syslog pushes event messages from routers to a central server, unlike SNMP which is primarily poll-based
- Severity levels 0–7 control what gets logged; default sends level 6 (informational) and more severe
- Each router sources syslog traffic from whichever interface it uses to reach the destination — R1 showed as 10.0.0.1, R2 as 10.0.12.2, R3 as 10.0.23.2 on the server's HostName column, not a single fixed management IP
- Confirmed live event capture by shutting/no-shutting R3's Gig0/0 and watching `%LINK-5-CHANGED` and `%LINEPROTO-5-UPDOWN` messages arrive on the server in real time

**Verification:** All three routers' `LOGGINGHOST_STARTSTOP` and `CONFIG_I` messages appeared on Server0's Syslog table. Live interface flap on R3 was captured and visible on the server within seconds.

**Screenshots:** [reference your Syslog screenshots]

---

## Lab 3: SNMP

**Setup:** SNMPv2c community string configured on all three routers.

```
snmp-server community CCNALAB RO
```

**What I learned:**
- SNMP is primarily poll-based (a monitoring station queries the router), in contrast to Syslog's push model
- SNMPv2c uses a plaintext community string — no encryption or real authentication
- SNMPv3 adds authentication (SHA) and encryption (AES) via groups, users, and security levels (noAuthNoPriv, authNoPriv, authPriv)

**Verification:** `%SNMP-5-WARMSTART` message confirmed the SNMP agent activated on all three routers. Config confirmed via `show running-config | include snmp` on R1, R2, and R3.

**Screenshots:** [reference your SNMP screenshots]

---

## Packet Tracer Limitations Found This Session

- `logging trap` only accepts the `debugging` keyword — other severity keywords (`informational`, `warnings`, etc.) present on real IOS are not implemented. Verified via `logging trap ?`.
- Server-PT has no SNMP service — SNMP can be configured and verified on routers, but not polled end-to-end in-simulator.
- `snmp-server` accepts only the `community` subcommand — `location`, `contact`, `group`, `user`, and `view` are all absent.
- `show snmp` runs without error but produces no output (no agent stats or counters).
- `show snmp community` does not exist as a command; verification requires `show running-config | include snmp` instead.
- **SNMPv3 is not configurable anywhere in Packet Tracer.** Tested `snmp-server group ?` across four router platforms (Cisco 2901, 2911, ISR4321, ISR4331) spanning both ISR G2 and ISR 4000 series — all returned "Unrecognized command." This confirms the gap is platform-wide, not model-specific.

---

## Key Commands Learned

| Command | Purpose |
|---------|---------|
| `ntp server [ip]` | Configure NTP client to sync from a server |
| `ntp master` | Configure router as NTP master |
| `show ntp status` | Verify sync status and stratum |
| `show ntp associations` | View NTP peer relationships |
| `logging host [ip]` | Send syslog messages to a server |
| `show running-config \| include logging` | Verify logging config (PT workaround) |
| `snmp-server community [string] RO` | Configure SNMPv2c read-only community |
| `show running-config \| include snmp` | Verify SNMP config (PT workaround) |

---

## What I Will Learn Next Week

- Security fundamentals: ACLs, port security, DHCP snooping, VPN theory

---

*Last updated: 18-Aug-2026
