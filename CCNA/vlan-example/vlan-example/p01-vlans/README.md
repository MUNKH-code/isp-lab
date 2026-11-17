# P01 — VLANs, 802.1Q trunks and router-on-a-stick

> Notes: [../notes/01-vlans.md](../notes/01-vlans.md) · Faults: [faults.md](faults.md)

## Objective
Two hosts in different VLANs, on different switches, reach each other through
802.1Q trunks and one router doing inter-VLAN routing.

## Diagram
```
 PC1 (VLAN 10)                                     PC2 (VLAN 20)
 10.10.10.11/24                                    10.10.20.12/24
      |                                                  |
   Gi0/1                                              Gi0/1
  [ SW1 ] Gi0/0 ===== trunk 10,20,99 ===== Gi0/0 [ SW2 ]
                      native 99                    Gi0/2
                                                     ||  trunk 10,20,99, native 99
                                                   Gi0/0
                                                  [ R1 ]  .10 / .20 / .99 subinterfaces
```

## Addressing
| VLAN | Name | Subnet | Gateway (R1) | Members |
|---|---|---|---|---|
| 10 | USERS | 10.10.10.0/24 | Gi0/0.10 → .1 | PC1 .11 |
| 20 | SERVERS | 10.10.20.0/24 | Gi0/0.20 → .1 | PC2 .12 |
| 99 | NATIVE-MGMT | 10.10.99.0/24 | Gi0/0.99 → .1 | SW1 .11, SW2 .12 |
| 999 | PARKING | — | — | unused ports, shut down |

## Platform
- Cisco Modeling Labs: SW1, SW2 = IOSvL2 · R1 = IOSv · PC1, PC2 = Alpine/Desktop
- 5 nodes total, so it fits CML Free (check the current node limit)

## How to run
1. Build the topology above in CML and paste the configs from `configs/`
2. On the hosts:
   ```
   PC1: ip addr add 10.10.10.11/24 dev eth0 && ip route add default via 10.10.10.1
   PC2: ip addr add 10.10.20.12/24 dev eth0 && ip route add default via 10.10.20.1
   ```

## Expected result
| Check | Command | Expected |
|---|---|---|
| VLANs exist, ports assigned | `show vlan brief` (SW1, SW2) | VLAN 10 on SW1 Gi0/1, VLAN 20 on SW2 Gi0/1, Gi0/3 etc. in 999 |
| Trunks up | `show interfaces trunk` | Gi0/0 (and Gi0/2 on SW2) mode `on`, encapsulation `802.1q`, native `99`, allowed `10,20,99` |
| Same subnet across a trunk | PC1 → `ping 10.10.10.1` | Replies (crosses 2 trunks) |
| Inter-VLAN routing | PC1 → `ping 10.10.20.12` | Replies, `traceroute` shows hop 10.10.10.1 |
| Management VLAN | SW1 → `ping 10.10.99.12` | Replies |
| MAC learning | `show mac address-table dynamic` (SW2) | PC1's MAC in VLAN 10 learned on Gi0/0 |

## Faults injected
1. VLAN 20 removed from a trunk's allowed list
2. Native VLAN mismatch
3. Access port in the wrong VLAN

## What I learned
(Write 3–5 lines in your own words after the lab.)
