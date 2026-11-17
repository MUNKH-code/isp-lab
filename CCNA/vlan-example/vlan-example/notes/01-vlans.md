# 01 — VLANs and 802.1Q trunks

> Status: ✅ done · Lab: [p01-vlans](../p01-vlans/) · Last reviewed: YYYY-MM-DD

*Model example. Write your own version first, then compare — copying this teaches you nothing.*

---

## 1. What it is

A VLAN splits one physical switch network into several separate **broadcast domains**.
Hosts in different VLANs cannot talk at Layer 2 — traffic between them must go through a router or L3 switch.
A **trunk** is a link that carries many VLANs at once by adding an 802.1Q tag to each frame.

**Why it matters in an ISP:** aggregation networks separate services by VLAN (internet, IPTV, VoIP, management),
and BNGs identify subscribers by VLAN — often two tags stacked (QinQ: outer S-VLAN per area/OLT, inner C-VLAN per customer).

## 2. How it works

### Port types
| Port type | Carries | Tagged? | Typical use |
|---|---|---|---|
| Access | 1 VLAN | No — the switch adds/removes the tag internally | Host, server, CPE |
| Trunk | Many VLANs | Yes, except the native VLAN | Switch–switch, switch–router |
| Hybrid (Huawei) | Mix of tagged and untagged VLANs | Per VLAN | Huawei default on many switches |

### The 802.1Q tag (4 bytes, inserted after the source MAC)
| Field | Size | Meaning |
|---|---|---|
| TPID | 16 bits | `0x8100` = "this frame is tagged" |
| PCP | 3 bits | Priority (CoS 0–7), used by QoS |
| DEI | 1 bit | Drop eligible |
| VID | 12 bits | VLAN ID 0–4095 → usable **1–4094** (0 and 4095 reserved) |

Max Ethernet frame grows from 1518 to **1522 bytes** with one tag. QinQ adds another 4.

### Native VLAN
- Frames in the native VLAN cross a trunk **untagged**. Default = VLAN 1.
- Both ends **must agree**. If not: traffic leaks between VLANs, CDP logs a mismatch, PVST+ blocks the port.
- Best practice: set native to an unused VLAN, never VLAN 1.

### Frame walk (PC1 VLAN 10 → R1 gateway)
1. PC1 sends an untagged frame to SW1 Gi0/1 (access, VLAN 10)
2. SW1 forwards out trunk Gi0/0 with tag VID 10
3. SW2 receives on Gi0/0, keeps VLAN 10, forwards out trunk Gi0/2 still tagged 10
4. R1 Gi0/0.10 (`encapsulation dot1Q 10`) accepts it — the subinterface is the VLAN's gateway

### DTP (Cisco only)
| SW-A \ SW-B | access | trunk | dynamic auto | dynamic desirable |
|---|---|---|---|---|
| **dynamic auto** | access | trunk | **access** | trunk |
| **dynamic desirable** | access | trunk | trunk | trunk |

Production rule: hard-code `switchport mode trunk` or `access` + `switchport nonegotiate`.

### VLAN ranges (Cisco)
Normal 1–1005 (1002–1005 reserved legacy) · Extended 1006–4094.

## 3. Configuration

**Cisco IOS — VLANs, access, trunk**
```
vlan 10
 name USERS
!
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
 switchport nonegotiate
 spanning-tree portfast
!
interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q      ! only on platforms that also support ISL (e.g. IOSvL2)
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 switchport nonegotiate
```

**Cisco IOS — change the allowed list safely**
```
switchport trunk allowed vlan add 30       ! adds 30
switchport trunk allowed vlan remove 30    ! removes 30
! "switchport trunk allowed vlan 30" alone REPLACES the whole list → outage
```

**Cisco IOS — router-on-a-stick**
```
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.10.10.1 255.255.255.0
interface GigabitEthernet0/0.99
 encapsulation dot1Q 99 native
 ip address 10.10.99.1 255.255.255.0
```

**Cisco IOS — L3 switch alternative (SVIs)**
```
ip routing
interface Vlan10
 ip address 10.10.10.1 255.255.255.0
```

**Huawei VRP (switch)** — verify against the docs for your model/version
```
vlan batch 10 20 99
#
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 10
#
interface GigabitEthernet0/0/24
 port link-type trunk
 port trunk pvid vlan 99
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 10 20 99
```

## 4. Verification

| Command (Cisco) | Huawei | What to look for |
|---|---|---|
| `show vlan brief` | `display vlan` | VLAN exists, active, correct access ports |
| `show interfaces trunk` | `display port vlan` | Mode on, native VLAN, allowed **and** "allowed and active" lists |
| `show interfaces Gi0/1 switchport` | `display interface Gi0/0/1` | Admin vs operational mode, access/native VLAN |
| `show mac address-table vlan 10` | `display mac-address vlan 10` | Host MAC learned on the expected port |
| `show spanning-tree inconsistentports` | — | Native-VLAN (PVID) inconsistencies |
| `show ip interface brief` (router) | `display ip interface brief` | Subinterfaces/SVIs up/up |

## 5. Troubleshooting

| Symptom | Likely cause | How to check |
|---|---|---|
| One host can't reach its gateway, link is up, IP correct | Access port in wrong VLAN | `show vlan brief` |
| One VLAN down across a path, other VLANs fine | VLAN missing from a trunk allowed list, or VLAN not created on a transit switch | `show interfaces trunk` → "allowed and active" |
| `%CDP-4-NATIVE_VLAN_MISMATCH`, port blocked | Native VLAN differs per end | `show interfaces trunk` on both ends |
| Trunk never comes up | DTP auto + auto, or encapsulation mismatch | `show interfaces Gix switchport` |
| Inter-VLAN routing fails, same-VLAN works | Router subinterface VLAN ID wrong or `ip routing` missing on L3 switch | `show run int Gi0/0.10`, `show ip route` |

Real examples: [faults.md](../p01-vlans/faults.md)

## 6. Exam and interview points
- A VLAN = a broadcast domain = (normally) one IP subnet.
- Usable VLAN IDs 1–4094; tag = 4 bytes; TPID 0x8100.
- Native VLAN is untagged and must match; don't use VLAN 1.
- Security: **VLAN hopping** via DTP switch spoofing (fix: `nonegotiate`, static modes) and via double tagging on the native VLAN (fix: unused native VLAN, tag native). Shut unused ports and park them in an unused VLAN.
- ISP angle: be ready to explain how subscriber VLANs / QinQ reach the BNG in your network (sanitized).

## 7. Recall questions
1. What are the four fields of an 802.1Q tag and how big is the VID?
2. What happens, step by step, when the native VLAN differs on two ends of a trunk?
3. Why is `switchport trunk allowed vlan 30` dangerous on a live trunk? What do you type instead?
4. Two switches both set to `dynamic auto` — trunk or access? Why?
5. A host can ping hosts in its own VLAN on another switch but not its gateway on a router-on-a-stick. Name two likely causes.
