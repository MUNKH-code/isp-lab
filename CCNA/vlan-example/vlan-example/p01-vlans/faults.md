# Fault log — P01 VLANs

> Example entries showing the expected depth. Times are examples — record your real ones.

## Fault 1 — VLAN 20 removed from the SW2 → R1 trunk

- **What I broke:** on SW2 Gi0/2: `switchport trunk allowed vlan remove 20`
- **Symptom:** PC2 cannot ping its gateway 10.10.20.1. PC1 still reaches 10.10.10.1.
- **Hypotheses (before checking):**
  1. PC2's access port is in the wrong VLAN
  2. VLAN 20 is not carried on a trunk between PC2 and R1
  3. R1 subinterface Gi0/0.20 is down or misconfigured
- **Checks:**
  - SW2 `show vlan brief` → Gi0/1 is in VLAN 20 ✔ (rules out 1)
  - SW2 `show interfaces trunk` → Gi0/2 "Vlans allowed on trunk: 10,99" ✘ — 20 missing
  - R1 `show ip interface brief` → Gi0/0.20 up/up ✔ (rules out 3)
- **Root cause:** VLAN 20 not in the allowed list on SW2 Gi0/2
- **Fix:** `switchport trunk allowed vlan add 20`
- **Verification:** `show interfaces trunk` lists 10,20,99 · PC2 → `ping 10.10.20.1` OK
- **Time to fix:** 6 min
- **Lesson:** single-VLAN outage + other VLANs fine on the same path → check the trunk allowed list first. Always use `add`/`remove`, never retype the list (that's how production outages happen).

## Fault 2 — Native VLAN mismatch

- **What I broke:** on SW1 Gi0/0: `no switchport trunk native vlan 99` (native falls back to VLAN 1)
- **Symptom:** SW1 cannot ping 10.10.99.12 or 10.10.99.1. Console logs on both switches:
  `%CDP-4-NATIVE_VLAN_MISMATCH` and a spanning-tree "inconsistent peer vlan" message; trunk port blocked for the affected VLANs.
- **Hypotheses:**
  1. SVI 99 down on SW1
  2. Native VLAN mismatch on the SW1–SW2 trunk
  3. VLAN 99 not allowed on the trunk
- **Checks:**
  - SW1 `show ip interface brief` → Vlan99 up/up ✔
  - `show interfaces trunk` on both ends → native 1 on SW1, 99 on SW2 ✘
  - `show spanning-tree inconsistentports` → Gi0/0 listed
- **Root cause:** untagged frames from SW2 (VLAN 99) arrive on SW1 into VLAN 1, and the reverse. PVST+ detects the mismatch and blocks the port for those VLANs.
- **Fix:** `switchport trunk native vlan 99` on SW1 Gi0/0
- **Verification:** no new CDP mismatch logs · `show spanning-tree inconsistentports` empty · ping OK
- **Time to fix:** 9 min
- **Lesson:** the native VLAN must match on both ends. Read the console logs before running show commands — the switch told me the answer.

## Fault 3 — Access port in the wrong VLAN

- **What I broke:** on SW1 Gi0/1: `switchport access vlan 20`
- **Symptom:** PC1 (10.10.10.11) cannot ping 10.10.10.1. Link is up.
- **Hypotheses:**
  1. Wrong IP or gateway on PC1
  2. Port assigned to the wrong VLAN
  3. VLAN 10 not carried across the trunks
- **Checks:**
  - PC1 `ip addr`, `ip route` → correct ✔
  - SW1 `show vlan brief` → Gi0/1 in VLAN 20 ✘
  - `show interfaces trunk` → VLAN 10 allowed and forwarding ✔
- **Root cause:** PC1 is a VLAN-10 addressed host sitting in VLAN 20 — its ARP for 10.10.10.1 goes to the wrong broadcast domain, nobody answers.
- **Fix:** `switchport access vlan 10`
- **Verification:** `show vlan brief` · `ping 10.10.10.1` OK · `show mac address-table` shows PC1 MAC in VLAN 10
- **Time to fix:** 4 min
- **Lesson:** "link up, can't reach gateway, IP is correct" → check which VLAN the port is in before anything else.
