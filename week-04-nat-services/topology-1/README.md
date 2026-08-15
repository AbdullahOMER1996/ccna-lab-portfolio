# Week 4: NAT/PAT

**Date:** 14-Aug-2026 - 15-Aug-2026

---

## Topics Covered

- Why NAT exists — private addressing vs. public routability
- Static NAT (1-to-1 permanent mapping)
- Dynamic NAT (pool-based, per-host mapping)
- PAT / NAT Overload (many hosts, one public IP, port-based tracking)
- Before/after connectivity testing to prove NAT's purpose, not just its configuration

---

## Topology

```
Inside Network                    Router (NAT)                Outside Network (simulated ISP)
┌──────────────┐                  ┌─────────┐                  ┌──────────────┐
│  PC0          │                  │         │                  │  ISP Router   │
│  PC1          │──── Switch ──────│  G0/0    │───── G0/1 ───────│  (Router2)    │
│  Server0      │                  │ (inside) │    (outside)     │               │
│  (R1)         │                  │         │                  │  WebServer    │
└──────────────┘                  └─────────┘                  └──────────────┘
192.168.1.0/24                                                  198.51.100.0/24
                                                203.0.113.0/24 (R1↔ISP link)
```

**File:** `topology-1/topology-1-NAT-PAT.pkt`

| Device | Interface | IP Address | Role |
|--------|-----------|-----------|------|
| PC0 | — | 192.168.1.10/24 | Inside host |
| PC1 | — | 192.168.1.11/24 | Inside host |
| Server0 | — | 192.168.1.20/24 | Inside host — target for Static NAT |
| R1 | G0/0 | 192.168.1.1/24 | Inside interface (`ip nat inside`) |
| R1 | G0/1 | 203.0.113.1/24 | Outside interface (`ip nat outside`) |
| ISP | G0/0/1 | 203.0.113.2/24 | Simulated ISP router |
| ISP | G0/0/0 | 198.51.100.1/24 | Connects to outside web server |
| WebServer (Server1) | — | 198.51.100.10/24 | Represents a real internet destination |

**Routing:** R1 uses a default route (`0.0.0.0/0 → 203.0.113.2`) toward the ISP — matching how a real edge router points at its ISP rather than running a routing protocol with it.

---

## Lab 1: Proving the Problem Before Configuring NAT

Before touching NAT, the topology was deliberately tested with **no address translation and no return route** on the ISP side, to get a genuine "before" baseline.

**Test — PC0 (192.168.1.10) pinging the outside WebServer (198.51.100.10):**
```
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**Why it failed:** R1 forwarded the packet fine (it has a default route to the ISP), but the ISP router had no route back to 192.168.1.0/24 — because a private (RFC 1918) address has no business appearing on a public network, and no real ISP would ever route to it. The private source address was the reason the round trip broke, not a routing mistake.

**What I learned:** An earlier version of this test technically "succeeded," but only because a manual static route back to 192.168.1.0/24 had been added on the ISP router as a shortcut while building the topology — an artificial return path that doesn't exist in reality. Removing that route and re-testing produced the honest 100% loss result above, which is the actual baseline NAT is meant to solve.

**Screenshots:** See `topology-1/Screenshots/`

---

## Lab 2: Static NAT

**Use case:** A device that must always be reachable at the same fixed public address — here, Server0.

```
interface gigabitEthernet 0/0
 ip nat inside
exit

interface gigabitEthernet 0/1
 ip nat outside
exit

ip nat inside source static 192.168.1.20 203.0.113.10
```

**Verification — `show ip nat translations`:**
```
Pro   Inside global    Inside local     Outside local   Outside global
---   203.0.113.10     192.168.1.20     ---             ---
```

**Test — Server0 pinging WebServer:**
```
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss, after ARP)
```

**What I learned:** Static NAT creates a permanent mapping that never expires — it appears in the translation table immediately and stays there regardless of traffic, unlike Dynamic NAT and PAT entries (see Lab 3).

**Screenshots:** See `topology-1/Screenshots/`

---

## Lab 3: Dynamic NAT

**Use case:** A pool of public addresses shared on demand by inside hosts — no fixed mapping.

```
ip nat pool DYNAMIC_POOL 203.0.113.20 203.0.113.21 netmask 255.255.255.0

access-list 1 permit 192.168.1.0 0.0.0.255

ip nat inside source list 1 pool DYNAMIC_POOL
```

**Key finding — translation entries expire quickly:** The first attempt to check `show ip nat translations` after a completed ping showed only Server0's static entry — the dynamic entries had already timed out and cleared. Re-testing by checking the table *immediately* after pinging (rather than after the ping had already finished) captured the dynamic entries in time.

**Verification — captured immediately after a PC0 ping:**
```
Pro    Inside global      Inside local
icmp   203.0.113.20:1     192.168.1.10:1
icmp   203.0.113.20:2     192.168.1.10:2
...
icmp   203.0.113.20:8     192.168.1.10:8
---    203.0.113.10       192.168.1.20
```

**What I learned:** Dynamic NAT assigns one pool address per host for the duration of its active session — all of PC0's entries used the same pool address (`.20`), confirming per-host consistency rather than per-packet reassignment. The `:N` suffix on each entry is the ICMP identifier/sequence number, which Packet Tracer uses in place of a real port number since ICMP doesn't have ports — this is worth noting as a simulation detail rather than standard IOS NAT table behavior for ICMP.

**Screenshots:** See `topology-1/Screenshots/`

---

## Lab 4: PAT (NAT Overload)

**Use case:** Many inside hosts sharing a single public IP, differentiated by port number — the most common real-world NAT configuration (e.g., a home router).

```
ip nat inside source list 1 pool DYNAMIC_POOL overload
```

**Verification — after pinging from both PC0 and PC1, captured immediately:**
```
Pro    Inside global      Inside local
icmp   203.0.113.20:10    192.168.1.10:10   ← PC0
icmp   203.0.113.20:11    192.168.1.10:11   ← PC0
icmp   203.0.113.20:12    192.168.1.10:12   ← PC0
icmp   203.0.113.20:1     192.168.1.11:1    ← PC1
icmp   203.0.113.20:2     192.168.1.11:2    ← PC1
icmp   203.0.113.20:3     192.168.1.11:3    ← PC1
icmp   203.0.113.20:4     192.168.1.11:4    ← PC1
icmp   203.0.113.20:9     192.168.1.10:9    ← PC0
---    203.0.113.10       192.168.1.20      ← Server0 (static, unchanged)
```

**Key finding:** Both PC0 (192.168.1.10) and PC1 (192.168.1.11) were translated to the **exact same public address** (203.0.113.20) at the same time, distinguished only by the port/identifier number after the colon. The second pool address (203.0.113.21) was never used at all — proving PAT's entire purpose: serving many internal hosts from a single public IP, rather than needing one public address per host.

**Screenshots:** See `topology-1/Screenshots/`

---

## Comparison: Static NAT vs. Dynamic NAT vs. PAT

| | Static NAT | Dynamic NAT | PAT / Overload |
|---|---|---|---|
| Mapping | 1 private IP ↔ 1 fixed public IP | 1 private IP ↔ 1 pool IP, per session | Many private IPs ↔ 1 public IP, by port |
| Entry lifetime | Permanent | Expires quickly after session ends | Expires quickly after session ends |
| Public IPs needed | 1 per mapped host | Up to 1 per simultaneous host | 1 total, regardless of host count |
| Real-world use | Servers needing a fixed address | Rarely used alone in modern networks | Default behavior of nearly all home/office routers |

---

## Key Commands Learned

| Command | Purpose |
|---------|---------|
| `ip nat inside` / `ip nat outside` | Mark an interface as the inside (private) or outside (public) NAT boundary |
| `ip nat inside source static [private-ip] [public-ip]` | Configure Static NAT |
| `ip nat pool [name] [start-ip] [end-ip] netmask [mask]` | Define a pool of public addresses for Dynamic NAT/PAT |
| `access-list [number] permit [network] [wildcard]` | Define which inside hosts are eligible for translation |
| `ip nat inside source list [acl] pool [pool-name]` | Configure Dynamic NAT |
| `ip nat inside source list [acl] pool [pool-name] overload` | Configure PAT (adds port-based sharing to the same pool) |
| `ip route 0.0.0.0 0.0.0.0 [next-hop]` | Default route — used on R1 to point toward the simulated ISP |
| `show ip nat translations` | View the current NAT/PAT translation table |
| `show run \| section nat` | View all NAT-related configuration lines |
| `show access-lists` | Verify ACL contents used by Dynamic NAT/PAT |

---

## Challenges Faced

- **False-positive "before NAT" test:** An early connectivity test appeared to succeed without any NAT configured, because a manual static route on the ISP router back to the private network had been added as a topology-building shortcut. Removing that route produced the honest, expected 100% packet loss — the real evidence that NAT is necessary in the first place.
- **Empty NAT table after a successful ping:** Checking `show ip nat translations` right after a ping had completed showed nothing for the dynamic entries — only the static one. This was resolved by checking the table *immediately*, before the short-lived dynamic/PAT entries expired, confirming that dynamic and PAT translations are timer-based while static translations are permanent.
- **Packet Tracer's ICMP port numbering:** Dynamic NAT and PAT tables showed a `:N` suffix on ICMP entries even though ICMP has no real port concept — this is Packet Tracer substituting the ICMP identifier/sequence number in that field, which is a simulation detail worth noting rather than standard real-IOS output for ICMP traffic.

---

## What's Next

- IP Services: DHCP, NTP, Syslog, SNMP (remainder of Week 4)
- Then Week 5: Security — ACLs, port security, DHCP snooping, VPN theory

---

*Last updated: 15-Aug-2026
