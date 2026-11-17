# isp-lab

Hands-on labs for ISP network engineering: routing, MPLS, and network automation.
Every lab is built, broken on purpose, fixed, and documented.

**Author:** Munkh — Network Planning Engineer (ISP core & aggregation, Huawei/Cisco)

## Projects

| # | Project | Topics | Status |
|---|---|---|---|
| P0 | [OSPF warm-up](_template/) | 2-router OSPF, containerlab, FRR | ✅ template |
| P1 | Aggregation ring | VLAN, RSTP, LACP, IPv6 | ⬜ |
| P2 | OSPF + IPv6 core | Multi-area OSPF, VRRP | ⬜ |
| P3 | IS-IS ISP core | OSPF → IS-IS migration | ⬜ |
| P4 | Multi-AS ISP edge | BGP policy, route reflectors, RPKI | ⬜ |
| P5 | MPLS L3VPN | LDP, VRF, VPNv4 | ⬜ |
| P6 | EVPN/VXLAN fabric | Spine/leaf, EVPN | ⬜ |
| P7 | Inventory collector | Python, Netmiko, TextFSM | ⬜ |
| P8 | Backup + templates | Ansible, Jinja2, Git diff | ⬜ |
| P9 | NetBox source of truth | NetBox, Ansible | ⬜ |
| P10 | Monitoring + flows | Prometheus, Grafana, Akvorado | ⬜ |
| P11 | Mini-ISP as code (capstone) | All of the above | ⬜ |

## Every lab folder contains

| File | Purpose |
|---|---|
| `README.md` | Objective, diagram, how to run, what I learned |
| `topology.clab.yml` | containerlab topology |
| `configs/` | Device configs |
| `faults.md` | Break/fix log: symptom → cause → fix |
| `notes.md` | My explanation of the topic, in my own words |

## Lab environment

- Linux VM, 16–32 GB RAM
- [containerlab](https://containerlab.dev) with FRRouting, Nokia SR Linux, Arista cEOS
- Cisco Modeling Labs Free / EVE-NG for Cisco syntax

## Rules

- All addresses use private (RFC 1918) or documentation ranges (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`, `2001:db8::/32`) and private ASNs (`64512–65534`).
- No production configs, credentials or customer data from any employer — ever.
