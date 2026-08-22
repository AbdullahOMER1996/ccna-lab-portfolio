# Week 5: Security — ACLs, Port Security, DHCP Snooping, VPN Theory

**Date:** 19 Aug 2026 – 22 Aug 2026

**Topology files:**
- `topology-1/` — Standard and Extended ACLs
- `topology-2/topology-2-port-security.pkt` — Port Security
- `topology-3/topology-3-dhcp-snooping.pkt` — DHCP Snooping

---

## Topics Covered

- Standard ACLs (source-IP filtering)
- Extended ACLs (protocol/port-level filtering)
- Named vs numbered ACL syntax
- Port Security (sticky MAC learning, violation modes)
- DHCP Snooping (rogue DHCP server mitigation)
- VPN / IPsec — theory only (site-to-site vs remote-access, IPsec's two phases, symmetric vs asymmetric encryption)

---

## Lab 1: Standard ACL

**Setup:** Router with two subnets (192.168.1.0/24 LAN, 192.168.2.0/24 server segment). Goal: block one specific host (PC0) from reaching the server, while leaving PC1 and all other traffic unaffected.

**Config:**
```
ip access-list standard BLOCK-PC0
 deny host 192.168.1.10
 permit any
!
interface GigabitEthernet0/1
 ip access-group BLOCK-PC0 out
```

**Why placed here:** Standard ACLs filter on source IP only, so they must sit close to the **destination** — otherwise they risk blocking traffic that should still reach other parts of the network. Applied outbound on the interface facing the server.

**What I learned:**
- Named ACLs use `ip access-list standard [name]`, not `access-list standard` — the latter is the numbered-ACL command and doesn't accept a name argument
- Named ACL lines can be edited individually using sequence numbers (`no 10` then re-adding `10 deny ...`) instead of rebuilding the whole ACL
- An ICMP "Destination host unreachable" reply comes from the router interface that dropped the packet, not from the actual destination

**Verification:** PC0 → Server: 100% packet loss (`Destination host unreachable` from 192.168.1.1). PC1 → Server: 0% loss, full connectivity maintained.

**Screenshots:** See `topology-1/screenshots/`

**Setup:** Same topology. Goal: block only **Telnet** traffic from PC1 to the server, while still allowing ping (ICMP) from the same host.

**Config:**
```
ip access-list extended BLOCK-TELNET
 deny tcp host 192.168.1.11 host 192.168.2.2 eq 23
 permit ip any any
!
interface GigabitEthernet0/0
 ip access-group BLOCK-TELNET in
```

**Why placed here:** Extended ACLs can filter precisely on protocol and port, so they're placed close to the **source** and applied inbound — stopping unwanted traffic immediately rather than letting it cross the network first.

**What I learned:**
- Extended ACLs can distinguish between traffic types to the same destination — something a Standard ACL can never do
- IOS automatically translates well-known port numbers to service names in `show` output (`eq 23` displays as `eq telnet`)
- The `(N matches)` counter in `show access-lists` confirms a rule was actually triggered by real traffic, not just configured

**Verification:** PC1 → Server Telnet: connection timed out (blocked). PC1 → Server ping: 4/4 replies, TTL=127 (still allowed).

**Screenshots:** See `topology-1/screenshots/`

---

## Lab 3: Port Security

**Setup:** Switch port (Fa0/1) restricted to a single MAC address via sticky learning, violation mode set to Shutdown (default).

**Config:**
```
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

**What I learned:**
- Sticky MAC learning only triggers on **actual traffic**, not just configuration — the port shows 0 learned addresses until the connected device sends a frame (e.g., a ping)
- When a second device (different MAC) connects to a maximum-1 port, the switch immediately generates `PSECURE_VIOLATION` and `ERR_DISABLE` log messages and puts the port into **err-disabled** state — link light goes red, port stops passing all traffic
- Recovery requires manually cycling the interface (`shutdown` / `no shutdown`) — it does not recover on its own
- The `Last Source Address` field in `show port-security interface` reflects the most recently *seen* MAC (which could be a violating device), not necessarily the authorized sticky entry — `show mac address-table` is the more reliable source for confirming which MAC is actually bound to the port

**Verification:** Authorized device (PC0) passes traffic normally. Unauthorized device (PC1) connecting to the same port triggers a violation, port goes err-disabled, and recovers cleanly after manual intervention — with the original MAC confirmed still authorized via `show mac address-table` afterward.

**Screenshots:** See `topology-2/screenshots/`

---

## Lab 4: DHCP Snooping

**Setup:** One legitimate DHCP server (router, 192.168.1.0/24 pool) and one rogue DHCP server (Server-PT, deliberately misconfigured with a different subnet: 10.10.10.0/24) both connected to the same switch. Goal: prevent the rogue server from successfully leasing addresses to clients.

**Config:**
```
ip dhcp snooping
ip dhcp snooping vlan 1
no ip dhcp snooping information option
!
interface FastEthernet0/1
 ip dhcp snooping trust
```

**What I learned:**
- Before snooping, DHCP responses were a race condition — one client received a legitimate lease, another initially received an address from the rogue server before a clean release/renew corrected it
- All switch ports are **untrusted by default** once snooping is enabled — only the port facing the legitimate DHCP server needs to be explicitly trusted
- Enabling snooping initially **broke legitimate DHCP entirely** — diagnosed as Option 82 insertion (relay agent information automatically added to DHCP packets on untrusted ports) not being handled correctly by the simulated router's DHCP server. Disabling Option 82 insertion (`no ip dhcp snooping information option`) resolved it while leaving trust-based filtering fully intact
- `show ip dhcp snooping` was the key diagnostic command confirming trust configuration was correct before isolating Option 82 as the actual cause

**Verification:** After the fix, repeated `ipconfig /release` / `/renew` cycles on both clients consistently returned only legitimate 192.168.1.x addresses — the rogue server was never able to respond again.

**Screenshots:** See `topology-3/screenshots/`

---

## Lab 5: VPN / IPsec — Theory

Not hands-on labbed — CCNA 200-301 tests VPN/IPsec at a conceptual level, not configuration depth (that's CCNP-level scope). Covered as theory:

- **VPN purpose:** encrypted tunnel across an untrusted network (typically the internet)
- **Site-to-site vs remote-access:** network-to-network always-on tunnel vs. single user connecting in on-demand
- **IPsec's two phases:**
  - Phase 1 (ISAKMP/IKE) — routers authenticate each other and establish a secure channel using Diffie-Hellman key exchange
  - Phase 2 (IPsec SA) — using the secure channel from Phase 1, routers negotiate the actual data encryption parameters
- **Tunnel mode vs transport mode:** Tunnel mode encrypts the entire original packet (used for site-to-site); transport mode encrypts only the payload
- **Symmetric vs asymmetric encryption:** Diffie-Hellman (asymmetric-style exchange) is used once during Phase 1 to safely establish a shared secret without transmitting it directly; that shared secret then becomes the basis for fast symmetric encryption (e.g., AES) used for the actual bulk data traffic in Phase 2 — combining asymmetric security with symmetric speed

---

## Packet Tracer Limitations Found This Week

- Named ACL creation requires `ip access-list standard|extended [name]` — the plain `access-list standard` command does not exist and only accepts numeric ranges (1–99, 100–199).
- DHCP snooping's default Option 82 insertion causes the simulated router-as-DHCP-server to silently drop client requests relayed through untrusted ports. Resolved with `no ip dhcp snooping information option`; trust/untrust filtering remained fully functional throughout.

---

## Key Commands Learned

| Command | Purpose |
|---------|---------|
| `ip access-list standard [name]` | Create a named standard ACL |
| `ip access-list extended [name]` | Create a named extended ACL |
| `deny host [ip]` / `permit any` | Standard ACL rule syntax |
| `deny tcp host [src] host [dst] eq [port]` | Extended ACL rule syntax |
| `ip access-group [name] in\|out` | Apply an ACL to an interface |
| `show access-lists` | View ACL rules and match counters |
| `switchport port-security` | Enable port security on an interface |
| `switchport port-security mac-address sticky` | Auto-learn and lock the connected device's MAC |
| `switchport port-security violation [protect\|restrict\|shutdown]` | Set violation response mode |
| `show port-security interface [int]` | View port security status |
| `show mac address-table` | View actual MAC-to-port bindings |
| `ip dhcp snooping` / `ip dhcp snooping vlan [id]` | Enable DHCP snooping globally / per VLAN |
| `ip dhcp snooping trust` | Mark a port as trusted for DHCP server traffic |
| `show ip dhcp snooping` | View snooping configuration and trust status |

---

## What I Will Learn Next Week

- Network automation: Python and Netmiko for SSH-based configuration management

---

*Last updated: 22 Aug 2026*
