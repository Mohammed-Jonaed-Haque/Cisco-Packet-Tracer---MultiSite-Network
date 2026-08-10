# Multi-Site Enterprise Network Design

A fully functional multi-site enterprise network designed and implemented in Cisco Packet Tracer. The network connects 7 geographically distributed branch sites to a centralised Data Centre using VLSM subnetting, static routing, and a redundant triangle core topology.

---

## What the Project Includes

### Network Architecture
- **3-router triangle core** — MainRouter (Cisco 2911), Router1 (ISR4321), Router2 (Cisco 2911) interconnected via GigabitEthernet and Serial point-to-point links for path redundancy
- **7 branch sites** — each with a dedicated site router, access switch, and 3 representative PCs
- **Centralised Data Centre** — Server1 and Server2 on a dedicated /29 subnet connected via Router2
- **2 aggregation switches** — Switch1 aggregating Sites 1–4, Switch0 aggregating Sites 5–7
- **10 routers, 10 switches, 21 PCs, 2 servers**

### IP Addressing — VLSM Design
The entire network runs on a single `172.31.0.0/19` block (8,190 usable addresses), divided into 13 non-overlapping subnets using Variable Length Subnet Masking:

| # | Location | Prefix | Network | Gateway | Hosts |
|---|---|---|---|---|---|
| 1 | Site 7 | /25 | 172.31.0.0 | 172.31.0.1 | 126 |
| 2 | Site 5 | /26 | 172.31.0.128 | 172.31.0.129 | 62 |
| 3 | Site 3 | /26 | 172.31.0.192 | 172.31.0.193 | 62 |
| 4 | Site 2 | /27 | 172.31.1.0 | 172.31.1.1 | 30 |
| 5 | Site 6 | /27 | 172.31.1.32 | 172.31.1.33 | 30 |
| 6 | Site 1 | /27 | 172.31.1.64 | 172.31.1.65 | 30 |
| 7 | Site 4 | /27 | 172.31.1.96 | 172.31.1.97 | 30 |
| 8 | Data Centre | /29 | 172.31.1.128 | 172.31.1.129 | 6 |
| 9 | Transit SW1 | /29 | 172.31.1.152 | 172.31.1.153 | 6 |
| 10 | Transit SW0 | /29 | 172.31.1.160 | 172.31.1.161 | 6 |
| 11 | R0 ↔ R1 | /30 | 172.31.1.136 | — | 2 |
| 12 | R0 ↔ R2 | /30 | 172.31.1.140 | — | 2 |
| 13 | R1 ↔ R2 | /30 | 172.31.1.144 | — | 2 |

Total addresses consumed: **~420 out of 8,190** — leaving over 7,700 in reserve.

### Routing
- **Static routing** configured on all 3 core routers — each covering all 13 subnets via specific next-hops
- **Default routes** on all 7 site routers — single upstream route to the aggregation router
- Triangle topology provides **path redundancy** — if any single link fails, traffic reroutes automatically through the remaining two links

### Switch Management
All 10 switches configured with VLAN 1 management IPs from their respective subnets, enabling remote management via Telnet or SSH.

---

## How It Works

### Packet Flow — Site 1 PC to Data Centre Server
1. **Site1_PC1** (172.31.1.68) sends a packet to **Server1** (172.31.1.131)
2. PC sends frame to **Site1_Router** (gateway 172.31.1.65) — MAC changes, IP stays the same
3. Site1_Router forwards via default route to **Router1** (172.31.1.153) through Switch1 transit subnet
4. Router1 matches static route `172.31.1.128/29 → 172.31.1.146` and forwards to **Router2** via serial link
5. Router2 delivers directly to Server1 on its connected Data Centre subnet (Gi0/1)
6. Total: **3 router hops** — confirmed by TTL=125 (128 − 3)

### Why VLSM
Site user counts range from 14 to 80 users. Using a flat /25 for every site would waste 87% of addresses at smaller sites and consume 896 addresses for LANs alone. VLSM allocates exactly what each site needs — a /25 for 80 users, /26 for 29 and 28 users, /27 for the rest — consuming only 420 addresses total.

### Why Triangle Core
A linear chain (R0 → R1 → R2) would isolate all Sites 5–7 and the Data Centre if the R1–R2 link failed. The triangle topology ensures no single link failure can partition the network — traffic always has an alternative path.

---

## Connectivity Testing

All 8 test cases pass with 0% packet loss:

| Test | Type | Source | Destination | Result |
|---|---|---|---|---|
| 1 | Intra-site | Site1_PC1 | Site1_PC2 | ✓ Pass |
| 2 | Intra-site | Site5_PC1 | Site5_PC2 | ✓ Pass |
| 3 | Inter-site | Site1_PC1 | Site5_PC1 | ✓ Pass |
| 4 | Inter-site | Site3_PC1 | Site7_PC1 | ✓ Pass |
| 5 | Inter-site | Site4_PC1 | Site6_PC1 | ✓ Pass |
| 6 | PC → Server | Site1_PC1 | Server1 | ✓ Pass |
| 7 | PC → Server | Site5_PC1 | Server1 | ✓ Pass |
| 8 | PC → Server | Site7_PC1 | Server2 | ✓ Pass |

---

## What I Learned

- **VLSM subnetting** — how to calculate subnet sizes based on actual host requirements including gateway and management IPs, and why largest-first allocation prevents gaps and overlaps
- **Static routing** — how to build routing tables that cover every subnet across a multi-router topology, and how to verify them with `show ip route`
- **Layer 2 vs Layer 3 addressing** — how IP addresses stay constant end-to-end while MAC addresses change at every router hop
- **Redundant topology design** — the trade-off between a linear chain (simple routing) and a triangle core (fault tolerance but more complex routing)
- **Cisco IOS CLI** — configuring interfaces, static routes, switch management IPs, and saving configurations with `write memory`
- **Troubleshooting** — using `show ip interface brief`, `show ip route`, and `ping` to verify connectivity at every layer

---

## Tools Used

- **Cisco Packet Tracer** — network simulation and CLI configuration
- **Cisco IOS** — router and switch configuration
- **Devices** — Cisco 2911, Cisco ISR4321, Cisco 2960-24TT

---

## Files

| File | Description |
|---|---|
| `network.pkt` | Complete Packet Tracer simulation file |
| `report.pdf` | Full network design report with VLSM working, configs, and test results |

---

## Network Topology

```
                    [MainRouter]
                   /            \
              (Gi/30)        (Gi/30)
             /                    \
        [Router1] ---serial/30--- [Router2]
            |                         |        \
       (Gi/29)                   (Gi/29)    (Gi/29)
            |                         |        |
        [Switch1]               [Switch0]  [Switch2]
       /  |  |  \              /   |   \       |
      S1  S2  S3  S4          S5   S6   S7  Server1
                                            Server2
```
