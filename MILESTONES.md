# Milestones

CMPG325-2026-089 | Kamogelo Insurance Brokers (CLI-089)

| Milestone | Due | Status |
|---|---|---|
| Project commencement | 14 Aug 2026 | Complete |
| Milestone 1: Client Design Review | 28 Aug 2026 | Submitted |
| Milestone 2: Client Implementation Review | 2 Oct 2026 | Packet Tracer build completed |
| Final submission | 16 Oct 2026 | In progress |

## Milestone 1: Client Design Review

Delivered the five required items:

1. **Client requirements:** `01-client-requirements/requirements.md`
2. **Physical topology:** `02-network-design/topology.md`
3. **Logical topology:** `02-network-design/topology.md`
4. **IP addressing plan:** `03-ip-addressing/addressing-plan.md`
5. **Initial GitHub repository:** this repository

Key decisions: four VLANs from a single /24 using VLSM, two edge routers for
CR15, a standalone access point for the Foundational wireless challenge, and the
boardroom on its own VLAN so the wired and wireless presentation ports are
dedicated.

## Milestone 2: Client Implementation Review

Required items and where they are:

1. **Working Packet Tracer file:** `04-packet-tracer/CMPG325-Milestone2-Cisco.pkt`
2. **Assigned feature implemented:** AP1-BOARD with SSID `KIB-BOARD` and WPA2-PSK
   on VLAN 30, with wireless clients receiving addresses from the boardroom DHCP
   pool. Configuration: `05-configuration/device-configuration.md`
3. **Testing evidence:** `06-testing-evidence/` (plan and screenshots)
4. **Updated GitHub portfolio:** this repository

Changes from the Milestone 1 design, found during the build:

- The NIM-ES2-4 module added to R3-CORE provides switch ports, which cannot be
  routed (`no switchport` returned "% Incomplete command"). Inter-VLAN routing
  therefore uses SVIs on R3-CORE with a trunk to SW1-CORE, instead of
  router-on-a-stick. The Milestone 1 logical diagram already allowed SVIs.
- R1 and R2 gained a return route to 192.168.40.0/24.
- SW1-CORE (3560-24PS) has only two Gigabit ports, so Fa0/1 to Fa0/4 carry the
  access-switch uplinks.

## Final submission

Required items:

| Item | Where |
|---|---|
| Packet Tracer project (.pkt) | `04-packet-tracer/` |
| GitHub portfolio of evidence | this repository |
| Technical report | submitted on eFundi |
| 15-20 minute inset video demonstration | submitted on eFundi |

The final `.pkt`, screenshots and reflection are added to this repository
before the 16 October 2026 deadline.
