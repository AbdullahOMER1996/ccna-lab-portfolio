# Week 3: OSPF Multi-Area, Security, Troubleshooting \& EIGRP Comparison

**Date:** 2-Aug-2026 - 12-Aug-2026

\---

## Topics Covered

* Multi-area OSPF design (Area 0 backbone + Area 1 + Area 2)
* OSPF router types, packet types, and neighbor states
* Route summarization at the ABR using `area range`
* OSPF MD5 authentication
* Break-and-troubleshoot methodology using only `show` commands
* EIGRP configuration and side-by-side comparison against OSPF

\---

## Topology

```
Area 0 (Backbone)          Area 1                    Area 2
┌─────────────┐            ┌─────────────┐            ┌─────────────┐
│  PC0        │            │  PC1        │            │  PC2        │
│  10.0.0.15  │            │  10.1.0.15  │            │  10.2.0.15  │
│  GW: 10.0.0.1│           │  GW: 10.1.0.1│           │  GW: 10.2.0.1│
└──────┬──────┘            └──────┬──────┘            └──────┬──────┘
       │                          │                          │
      Gig0/0                    Gig0/0                     Gig0/0
       │                          │                          │
    ┌──┴──┐                    ┌──┴──┐                    ┌──┴──┐
    │ R1  │───Serial0/0/0───────│ R2  │───Serial0/0/1───────│ R3  │
    │     │    10.0.12.0/30     │     │    10.0.23.0/30     │     │
    └─────┘                    └─────┘                    └─────┘
   Area 0 only                ABR (Area 0+1)              ABR (Area 0+2)
```

**File:** `topology-1/topology-1-OSPF-Deep-Dive.pkt`

|Router|Interface|IP Address|Area|
|-|-|-|-|
|R1|G0/0|10.0.0.1/24|0|
|R1|S0/0/0|10.0.12.1/30|0|
|R2|G0/0|10.1.0.1/24|1|
|R2|S0/0/0|10.0.12.2/30|0|
|R2|S0/0/1|10.0.23.1/30|0|
|R3|G0/0|10.2.0.1/24|2|
|R3|S0/0/1|10.0.23.2/30|0|

\---

## Lab 1: OSPF Router Types, Packet Types \& Neighbor States (Theory)

**OSPF Router Types:**

|Type|Role|
|-|-|
|Internal Router|All interfaces in one area (R1)|
|ABR (Area Border Router)|Connects backbone to another area (R2, R3)|
|ASBR|Connects OSPF to external networks (not used in this lab)|
|Backbone Router|At least one interface in Area 0 (all three routers)|

**OSPF Neighbor States progression:** Down → Init → 2-Way → ExStart → Exchange → Loading → **Full**

\---

## Lab 2: Multi-Area OSPF Configuration

**Router IDs:** R1 = 1.1.1.1, R2 = 2.2.2.2, R3 = 3.3.3.3

**Configuration (R2 example — ABR):**

```
router ospf 1
 router-id 2.2.2.2
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.23.0 0.0.0.3 area 0
 network 10.1.0.0 0.0.0.255 area 1
```

**Verification — `show ip ospf neighbor`:** All adjacencies reached **FULL** state (R1↔R2, R2↔R3).

**Verification — `show ip route ospf` on R1:**

```
O    10.0.23.0 \[110/128] via 10.0.12.2
O IA 10.1.0.0  \[110/65]  via 10.0.12.2
O IA 10.2.0.0  \[110/129] via 10.0.12.2
```

Note: `10.0.23.0/30` shows as intra-area (`O`), not interarea (`O IA`), because that link sits entirely within Area 0 — only the Area 1 and Area 2 LAN networks show as `IA`.

**Connectivity test — PC0 to PC2:**

```
tracert 10.2.0.15
1  10.0.0.1
2  10.0.12.2
3  10.0.23.2
4  10.2.0.15
```

3 hops, exactly matching the physical topology.

**Screenshots:** See `topology-1/Screenshots/`

\---

## Lab 3: Route Summarization

Configured on both ABRs to summarize their local area's networks before advertising into the backbone.

**R2 (Area 1 → Area 0):**

```
router ospf 1
 area 1 range 10.1.0.0 255.255.255.0
```

**R3 (Area 2 → Area 0):**

```
router ospf 1
 area 2 range 10.2.0.0 255.255.255.0
```

**What I learned:** Summarization is configured on the ABR using `area range`, at the boundary of the area being summarized *out of*. With only one subnet per area in this lab, the summary equals the original network — in a larger design with multiple subnets per area, this would collapse several routes into one, reducing routing table size and hiding local topology churn from the rest of the network.

**Screenshots:** See `topology-1/Screenshots/`

\---

## Lab 4: OSPF MD5 Authentication

Configured on every OSPF-facing interface, both ends of each link, using key ID 1.

```
interface serial 0/0/0
 ip ospf message-digest-key 1 md5 CCNA2026
 ip ospf authentication message-digest
```

**Key learning:** Cisco IOS will not allow overwriting an existing `message-digest-key` with the same key ID — attempting to do so returns `OSPF: Key 1 already exists`. The old key must be removed first with `no ip ospf message-digest-key 1 md5 \[old-key]` before adding a new one.

**Verification — `show ip ospf interface serial 0/0/0`:**

```
Message digest authentication enabled
  Youngest key id is 1
```

Confirmed on all three routers, both links. All neighbor adjacencies remained **FULL** after authentication was enabled on both sides of each link.

**Screenshots:** See `topology-1/Screenshots/`

\---

## Lab 5: Break-and-Troubleshoot Exercise A — MD5 Key Mismatch

**Fault introduced:** Changed the MD5 key on R1's Serial0/0/0 only (`WrongKey123`), leaving R2 with the original key (`CCNA2026`).

**Diagnostic process (using only `show` commands, cold):**

|Step|Command|Finding|
|-|-|-|
|1|`show ip interface brief`|Interface up/up — ruled out physical/IP layer issue|
|2|`show ip ospf neighbor`|R1: empty. R2: neighbor 1.1.1.1 missing. Adjacency confirmed down on both sides|
|3|`show logging`|No explicit rejection message at default logging level|
|4|`debug ip ospf adj`|No additional detail surfaced|
|5|Root cause|Isolated by elimination — the only change made was the MD5 key|

**Key finding:** Packet Tracer's OSPF simulation does not expose MD5 authentication mismatch details through `show logging` or `debug ip ospf adj` at the informational level, unlike real IOS or GNS3/EVE-NG, where this would typically log explicitly. The fault was still fully diagnosable through elimination and configuration review — a legitimate real-world diagnostic path when logging doesn't provide full visibility.

**Fix:**

```
interface serial 0/0/0
 no ip ospf message-digest-key 1 md5 WrongKey123
 ip ospf message-digest-key 1 md5 CCNA2026
```

**Verification:** Both R1 and R2 confirmed adjacency back to **FULL**.

**Screenshots:** See `topology-1/Screenshots/`

\---

## Lab 6: Break-and-Troubleshoot Exercise B — Wrong OSPF Area

**Fault introduced:** Changed R2's network statement for the R1-facing link from Area 0 to Area 5, while R1 kept that same link in Area 0.

**Diagnostic process:**

|Step|Command|Finding|
|-|-|-|
|1|Log (automatic)|R2 immediately and repeatedly logged: `%OSPF-4-ERRRCV: Received invalid packet: mismatch area ID, from backbone area must be virtual-link but not found from 10.0.12.2, Serial0/0/0`|
|2|`show ip ospf neighbor`|R1: neighbor 2.2.2.2 gone — log showed `Dead timer expired`. R2: neighbor 1.1.1.1 gone|

**Key finding — contrast with Exercise A:** Area mismatches fail loudly and explicitly at the default logging level, naming the exact cause on the router that detects the conflict (R2, since it received a Hello claiming Area 0 while its own interface expected Area 5). MD5 mismatches, by contrast, fail silently under the same logging level. This is a meaningful operational difference: not all OSPF faults present the same way, and log severity varies by fault type.

**Fix:**

```
router ospf 1
 no network 10.0.12.0 0.0.0.3 area 5
 network 10.0.12.0 0.0.0.3 area 0
```

**Verification:** Recovery logged explicitly — `Nbr 1.1.1.1 on Serial0/0/0 from LOADING to FULL, Loading Done`. Both routers confirmed FULL.

**Screenshots:** See `topology-1/Screenshots/`

\---

## Lab 7: EIGRP Configuration \& Comparison Against OSPF

Configured EIGRP AS 100 on all three routers, on the same networks already running OSPF, to observe how two IGPs coexist and compare their route selection.

```
router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.23.0 0.0.0.3
 network 10.1.0.0 0.0.0.255
 no auto-summary
```

**Verification — `show ip eigrp neighbors`:** All three EIGRP adjacencies formed cleanly on the first attempt (no faults introduced in this lab).

**Routing table impact — `show ip route` on R1 after EIGRP came up:**

```
D    10.0.23.0/30 \[90/2681856] via 10.0.12.2
D    10.1.0.0/24   \[90/2172416] via 10.0.12.2
D    10.2.0.0/24   \[90/2684416] via 10.0.12.2
```

Every OSPF route (`O`/`O IA`) was silently replaced by an EIGRP route (`D`) — not because OSPF failed, but because EIGRP's Administrative Distance (90) is lower than OSPF's (110). OSPF continued calculating routes correctly in the background; it simply lost the tiebreak for which route gets installed in the forwarding table.

**Metric comparison:**

|Destination|OSPF cost (AD 110)|EIGRP metric (AD 90)|Installed in RIB|
|-|-|-|-|
|10.0.23.0/30|128|2,681,856|EIGRP|
|10.1.0.0/24|65|2,172,416|EIGRP|
|10.2.0.0/24|129|2,684,416|EIGRP|

**Key learning:** OSPF cost and EIGRP metric are not directly comparable numbers — OSPF cost is a simple bandwidth-based sum per hop, while EIGRP's default metric is a weighted composite of minimum bandwidth along the path and cumulative delay, which produces much larger numbers. The only value that determines which protocol's route wins when both are present is **Administrative Distance**, not the metric itself. From `show ip route 10.2.0.0`: EIGRP used a minimum bandwidth of 1544 Kbit (the T1-speed serial link, correctly identified as the bottleneck) and a total delay of 40,100 microseconds across 2 hops.

**Cleanup:** Removed EIGRP from all three routers (`no router eigrp 100`) to return to a clean, single-protocol OSPF topology.

**Verification after removal:**

* `show ip eigrp neighbors` — returned empty, process fully removed
* `show ip route` — OSPF routes reappeared instantly with the exact same costs as before (`10.0.23.0 \[110/128]`, `10.1.0.0 \[110/65]`, `10.2.0.0 \[110/129]`), confirming OSPF's database never stopped converging — it was simply unused while EIGRP was preferred

**Screenshots:** See `topology-1/Screenshots/`

\---

## Key Commands Learned

|Command|Purpose|
|-|-|
|`router ospf \[process-id]`|Start OSPF process|
|`router-id \[id]`|Manually set OSPF router ID|
|`network \[ip] \[wildcard] area \[id]`|Assign interface to OSPF area|
|`area \[id] range \[network] \[mask]`|Summarize routes at the ABR|
|`ip ospf message-digest-key \[id] md5 \[key]`|Configure MD5 auth key|
|`ip ospf authentication message-digest`|Enable MD5 auth on interface|
|`show ip ospf neighbor`|View adjacency states|
|`show ip route ospf`|View OSPF-learned routes|
|`show ip ospf interface \[interface]`|View per-interface OSPF detail, including auth status|
|`show ip ospf 1`|View OSPF process-level summary|
|`router eigrp \[AS-number]`|Start EIGRP process|
|`no auto-summary`|Disable classful auto-summarization in EIGRP|
|`show ip eigrp neighbors`|View EIGRP adjacencies|
|`show ip route \[network]`|View detailed route entry, including AD and metric breakdown|

\---

## Challenges Faced

* **MD5 key overwrite:** Attempting to directly overwrite an existing `message-digest-key` with the same key ID failed with `OSPF: Key 1 already exists`. Resolved by removing the old key first with `no ip ospf message-digest-key`, then adding the new one.
* **Limited log/debug visibility in Packet Tracer:** Neither `show logging` nor `debug ip ospf adj` exposed MD5 authentication mismatch details, unlike real IOS. Diagnosis was completed through elimination and configuration review instead — a valid real-world approach when tooling doesn't provide full visibility.
* **`show ip ospf route` unsupported:** This real-IOS command returned `Invalid input` in Packet Tracer. Used `show ip route ospf` and `show ip ospf 1` as supported alternatives to gather equivalent information.

\---

## Revised Plan: Exam Before End of September 2026

The original 8-week schedule assumed more runway than is actually available. With the exam now targeted for **before September 30, 2026**, the remaining content is being compressed rather than followed week-for-week:

|Priority|Topic|Change from Original Plan|
|-|-|-|
|Keep, full depth|NAT/PAT, DHCP, NTP, Syslog, SNMP|Unchanged — heavily tested|
|Keep, full depth|Security: ACLs, port security, DHCP snooping, VPN theory|Unchanged — heavily tested|
|**Cut**|Python/Netmiko automation, DevNet Sandbox|Dropped — smallest-weighted CCNA topic, not worth the remaining time|
|**Shrunk**|Practice exams|Reduced from a dedicated multi-day block to ongoing practice folded into remaining sessions, rather than a separate week|

**Next session:** NAT/PAT and IP Services labs, at the same hands-on depth as Weeks 1–3.

\---

*Last updated: 12-Aug-2026*

