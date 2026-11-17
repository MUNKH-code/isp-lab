# P0 — OSPF warm-up (template)

## Objective
Two FRR routers form an OSPF adjacency and exchange loopbacks.
Copy this folder for every new lab: `cp -r _template p01-aggregation-ring`

## Diagram
```
 r1 (lo 10.255.0.1) eth1 ---- 10.0.12.0/31 ---- eth1 r2 (lo 10.255.0.2)
                      area 0
```

## How to run
```bash
sudo containerlab deploy -t topology.clab.yml
docker exec -it clab-p00-ospf-r1 vtysh -c "show ip ospf neighbor"
docker exec -it clab-p00-ospf-r1 vtysh -c "show ip route ospf"
sudo containerlab destroy -t topology.clab.yml
```

## Expected result
- `show ip ospf neighbor` on r1 shows r2 in state `Full`
- r1 has an OSPF route to `10.255.0.2/32`; `ping 10.255.0.2 -I 10.255.0.1` works

## Faults I injected
See [faults.md](faults.md)

## What I learned
(3–5 lines, in your own words, written after the lab)
