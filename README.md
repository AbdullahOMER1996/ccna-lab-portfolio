# CCNA Lab Portfolio

**Name:** Abdullah Omer Hamza
**Location:** Riyadh, Saudi Arabia
**Background:** Bachelor's in Network Technology | Former Network Engineer
**Current Status:** CCNA 200-301 in progress (exam targeted before the end of September 2026)

---

## About This Portfolio

This repository documents hands-on lab work covering CCNA 200-301 exam topics — built in Cisco Packet Tracer, with full configuration files, verification screenshots, and written breakdowns of what was built and why.

Beyond passing the exam, the goal here is to keep a documented record of real, working configurations and real troubleshooting — including the mistakes, not just the clean results. Each week's write-up includes what broke, how it was diagnosed, and how it was fixed.

---

## Background

I hold a Bachelor's degree in Network Technology and have hands-on experience configuring enterprise routers, switches, and firewalls. My background also includes experience in Governance, Risk & Compliance, which shapes how I think about network design — I try to build networks that are not just functional, but auditable and secure by design.

- **Technical foundation:** Bachelor's in Network Technology + hands-on Network Engineer experience
- **Compliance lens:** Familiarity with ISO 31000:2018 ERM
- **Languages:** Native Arabic, fluent English

---

## Lab Progress

| Week | Topic | Status | Link |
|------|-------|--------|------|
| 1 | Network Fundamentals — subnetting, static routing | ✅ Complete | [week-01](./week-01-network-fundamentals) |
| 2 | VLANs, Trunking, STP/RSTP, EtherChannel | ✅ Complete | [week-02](./week-02-vlan-stp) |
| 3 | OSPF Multi-Area, MD5 Auth, Break-and-Troubleshoot, EIGRP Comparison | ✅ Complete | [week-03](./week-03-ospf-eigrp) |
| 4 | NAT/PAT, IP Services (DHCP, DNS, NTP, SNMP, Syslog) | 🔄 In Progress | [week-04](./week-04-nat-services) |
| 5 | Security — ACLs, Port Security, DHCP Snooping, VPN Theory | ⏳ Planned | [week-05](./week-05-security) |
| 6 | Automation — Python, Netmiko | ⏳ Planned | [week-06](./week-06-automation) |
| 7 | Practice Exams & Weak Area Review | ⏳ Planned | [week-07](./week-07-practice-exams) |
| 8 | CCNA Exam | ⏳ Planned | — |

---

## Tools Used

| Tool | Purpose | Cost |
|------|---------|------|
| Cisco Packet Tracer | Network simulation (routers, switches, firewalls, servers) | Free |
| Python + Netmiko | SSH automation and configuration management | Free |
| Wireshark | Protocol analysis and traffic capture | Free |

---

## Weekly Lab Structure

Each week folder contains:

- `.pkt` files — Packet Tracer topology files
- `Screenshots/` — Configuration verification and `show` command outputs
- `README.md` — What was built, what was learned, key commands used, and any faults diagnosed

---

## Highlighted Work

### Multi-Area OSPF with Security and Fault Diagnosis (Week 3)

Three-router, three-area OSPF topology with route summarization at both ABRs and MD5 authentication on every link. Includes two deliberate break-and-troubleshoot exercises — an authentication key mismatch and a misconfigured area ID — each diagnosed from symptoms using only `show` commands, then documented end-to-end. Also includes a side-by-side EIGRP comparison showing how Administrative Distance determines which protocol's routes get installed. [View Week 3](./week-03-ospf-eigrp)

### VLANs, Inter-VLAN Routing, and Layer 2 Redundancy (Week 2)

VLAN segmentation with Router-on-a-Stick inter-VLAN routing, STP/RSTP root bridge election (including manually forcing a root bridge change), and EtherChannel with LACP to eliminate STP blocking while preserving full bandwidth. [View Week 2](./week-02-vlan-stp)

---

## Certifications & Education

| Item | Status | Date |
|------|--------|------|
| Cisco Certified Network Associate (CCNA 200-301) | 🔄 In Progress | Targeted before Sep 30, 2026 |
| Bachelor of Science in Network Technology | ✅ Completed | 2020 |

---


---

*Last updated: August 2026*
