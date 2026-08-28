# 2. Physical Topology

**Project:** CMPG325-2026-089 · Kamogelo Insurance Brokers (Vryburg)

![Physical topology](../diagrams/physical-topology.png)
*Source file: [`diagrams/physical-topology.svg`](../diagrams/physical-topology.svg)*

---

## 2.1 Design approach

The site is a single-storey office, so the physical design is an **extended star (hierarchical) topology**
with one comms cabinet in the IT/server room:

- **Edge layer** — one Cisco 2911 router (`R1-EDGE`) with two WAN interfaces, one per Internet provider (CR15).
- **Core/distribution layer** — one Cisco 3560 switch (`SW-CORE`) in the IT room, the single aggregation point
  for all access switches, the WLC and the server farm.
- **Access layer** — three Cisco 2960 switches placed close to the user areas they serve, each with a single
  uplink to `SW-CORE`.
- **Wireless** — one WLC 2504 in the cabinet, three lightweight APs cabled back to the nearest access switch.

A star keeps cable runs short, isolates faults to one area, and lets KIB add an access switch later without
touching the rest of the network.

## 2.2 Device inventory

| Device | Packet Tracer model | Location | Role |
|---|---|---|---|
| `R1-EDGE` | Cisco 2911 | IT room cabinet | Inter-VLAN routing (802.1Q sub-interfaces), NAT/PAT, dual-ISP default routing, DHCP helper-address relay |
| `SW-CORE` | Cisco 3560-24PS | IT room cabinet | Layer 2 core; trunks to all access switches, WLC and router; server access ports |
| `SW-A1` | Cisco 2960-24TT | Broker floor cabinet | Access for broker/sales PCs, MFP, AP1 |
| `SW-A2` | Cisco 2960-24TT | Admin corridor cabinet | Access for admin/finance, reception, manager, AP2 |
| `SW-A3` | Cisco 2960-24TT | IT room (serves boardroom) | Boardroom presentation ports and AP3 |
| `WLC-1` | Cisco WLC 2504 | IT room cabinet | Central control of all APs, 3 WLANs |
| `AP1` | LAP (lightweight) | Broker floor ceiling, centre | Staff + guest coverage, broker area |
| `AP2` | LAP (lightweight) | Reception/admin ceiling | Staff + guest coverage, reception & admin |
| `AP3` | LAP (lightweight) | Boardroom ceiling | Dedicated boardroom coverage (constraint C-01) |
| `SRV-DHCP-DNS` | Server-PT | IT room rack | DHCP for VLAN 10/20/40/50/60, DNS |
| `SRV-FILE` | Server-PT | IT room rack | Client documents / intranet (HTTP) |
| `SRV-BACKUP` | Server-PT | IT room rack | Backup target / FTP |
| `ADMIN-PC` | PC-PT | IT room | Management workstation (VLAN 99) |
| `ISP-A` | Cisco 1941 / cloud | Provider | Primary fibre uplink |
| `ISP-B` | Cisco 1941 / cloud | Provider | Secondary LTE/wireless uplink |
| End devices | PC-PT, Laptop-PT, Printer-PT, TabletPC-PT | All areas | ~30 wired PCs, 3 MFPs, ~10 laptops/tablets, boardroom display PC |

## 2.3 Cabling / interconnection plan

| From | Interface | To | Interface | Media | Link type |
|---|---|---|---|---|---|
| `ISP-A` | Gi0/0 | `R1-EDGE` | Gi0/1 | Copper (serial/Ethernet) | WAN — primary |
| `ISP-B` | Gi0/0 | `R1-EDGE` | Gi0/2 | Copper | WAN — secondary (CR15) |
| `R1-EDGE` | Gi0/0 | `SW-CORE` | Fa0/1 | Cat6 straight-through | 802.1Q trunk (router-on-a-stick) |
| `SW-CORE` | Fa0/2 | `SW-A1` | Gi0/1 | Cat6 | 802.1Q trunk |
| `SW-CORE` | Fa0/3 | `SW-A2` | Gi0/1 | Cat6 | 802.1Q trunk |
| `SW-CORE` | Fa0/4 | `SW-A3` | Gi0/1 | Cat6 | 802.1Q trunk |
| `SW-CORE` | Fa0/5 | `WLC-1` | Gi0/1 | Cat6 | 802.1Q trunk (WLAN → VLAN mapping) |
| `SW-CORE` | Fa0/6–Fa0/8 | Servers | Fa0 | Cat6 | Access, VLAN 30 |
| `SW-CORE` | Fa0/9 | `ADMIN-PC` | Fa0 | Cat6 | Access, VLAN 99 |
| `SW-A1` | Fa0/1–Fa0/18 | Broker PCs / MFP | Fa0 | Cat6 to desk outlets | Access, VLAN 10 |
| `SW-A1` | Fa0/24 | `AP1` | Gi0 | Cat6 (ceiling) | Access, VLAN 99 (CAPWAP to WLC) |
| `SW-A2` | Fa0/1–Fa0/12 | Admin/finance, reception, manager PCs, MFP | Fa0 | Cat6 | Access, VLAN 20 |
| `SW-A2` | Fa0/24 | `AP2` | Gi0 | Cat6 (ceiling) | Access, VLAN 99 |
| `SW-A3` | Fa0/1–Fa0/4 | Boardroom floor-box presentation ports | — | Cat6 to boardroom floor boxes | Access, VLAN 40 |
| `SW-A3` | Fa0/5 | Boardroom display PC / presentation host | Fa0 | Cat6 | Access, VLAN 40 |
| `SW-A3` | Fa0/24 | `AP3` | Gi0 | Cat6 (ceiling) | Access, VLAN 99 |

**Port reservations:** on every access switch, ports 19–23 are left unpatched for growth; port 24 is reserved
for the access point; the Gigabit uplink port is reserved for the trunk to `SW-CORE`.

## 2.4 Access point placement and coverage intent

| AP | Physical position | Primary coverage area | 2.4 GHz channel |
|---|---|---|---|
| `AP1` | Ceiling, centre of broker/sales floor | Broker floor, corridor, part of admin | 1 |
| `AP2` | Ceiling above reception, facing admin | Reception, walk-in waiting area, admin/finance, manager's office | 6 |
| `AP3` | Ceiling, centre of boardroom | Boardroom only (dedicated per constraint C-01) | 11 |

Placement rationale, channel/power planning and the coverage target (≥ −67 dBm in all working areas, with
~15–20% cell overlap for roaming) are detailed in [`06-wireless-lan-design.md`](06-wireless-lan-design.md).

## 2.5 Boardroom detail (design constraint C-01)

The boardroom is deliberately served twice, in two independent ways:

1. **Dedicated wireless** — `AP3` is mounted in the boardroom ceiling and broadcasts the boardroom SSID
   (`KIB-BOARD`) mapped to VLAN 40, so visiting underwriters and staff can present wirelessly without joining
   the staff or guest network.
2. **Dedicated wired presentation ports** — four Cat6 runs from `SW-A3` terminate in two floor boxes at the
   boardroom table, plus one run to the wall-mounted display host. All five ports are access ports in VLAN 40
   with PortFast, so a laptop plugged in gets an address and reaches the projector/display and the Internet
   immediately.

## 2.6 Power, environmental and physical notes

- All network equipment lives in a single lockable 12U wall cabinet in the IT room, on a UPS sized for the
  router, switches, WLC and servers (Internet resilience is worthless if the cabinet loses power).
- APs are powered over Ethernet from the switch ports (`SW-CORE`/3560-24PS and PoE-capable access ports),
  removing the need for power outlets in the ceiling.
- The secondary provider's LTE/wireless CPE is mounted for best signal and cabled back to `R1-EDGE Gi0/2`.
- Horizontal cabling is Cat6 to RJ45 wall outlets/floor boxes; runs are kept under 90 m, which the site
  footprint allows comfortably.
