# CMPG 325 — Individual Semester Project · CMPG325-2026-089

**Client:** Kamogelo Insurance Brokers (Vryburg) · Client ID **CLI-089**
**Industry:** Professional Services
**Student:** M. Motsumi (41961366)
**Assigned addressing block:** `192.168.40.0/24`
**Assigned networking challenge:** Wireless LAN — AP integration & coverage (Foundational)
**Design constraint:** Boardroom requires dedicated wireless **and** wired presentation ports
**Change request:** CR15 — a second Internet connection is added for resilience and must be integrated

---

## 1. Project summary

Kamogelo Insurance Brokers is a single-site insurance brokerage in Vryburg, North West.
This project designs, simulates and documents a small-enterprise network in **Cisco Packet Tracer** that
provides segmented connectivity for four functional groups (Sales/Brokers, Admin & Management, Boardroom,
Server/Infrastructure), a boardroom with dedicated wired and wireless presentation connections, and
**dual Internet uplinks** for resilience.

The design is a three-tier layout (edge, core, access): VLAN-segmented access switches, router-on-a-stick
inter-VLAN routing on R3-CORE, a standalone access point (AP1-BOARD) in the boardroom, and two independent
edge routers with NAT, using floating static default routes for failover.

## 2. Repository structure

```
.
├── README.md                       ← this file
├── docs/
│   ├── Milestone1_Client_Design_Review.docx
│   ├── Milestone2_Client_Implementation_Review_41961366.docx   ← Milestone 2 written review
│   ├── Milestone2_Testing_Evidence_41961366.docx               ← all test screenshots, explained
│   ├── 01-client-requirements.md   ← Milestone 1 deliverable 1
│   ├── 02-physical-topology.md     ← Milestone 1 deliverable 2
│   ├── 03-logical-topology.md      ← Milestone 1 deliverable 3
│   ├── 04-ip-addressing-plan.md    ← Milestone 1 deliverable 4
│   ├── 05-design-decisions.md
│   ├── 06-wireless-lan-design.md   ← assigned networking challenge
│   ├── 07-cr15-dual-internet.md    ← client change request
│   └── 08-testing-plan.md
├── diagrams/
│   ├── physical-topology.svg / .png
│   └── logical-topology.svg / .png
├── ip-plan/
│   ├── ip-addressing-plan.csv      ← subnet allocation table
│   └── device-addressing.csv       ← per-interface addressing
├── configs/                        ← running-config export for every router and switch
├── packet-tracer/                  ← 41961366_CMPG325_M2.pkt
├── evidence/
│   ├── screenshots/                ← test evidence T01 … T16, named by test ID
│   └── troubleshooting/            ← issues found and how they were resolved
└── MILESTONES.md                   ← progress log against project milestones
```

## 3. Milestone status

| Milestone | Date | Scope | Status |
|---|---|---|---|
| Commencement | 14 Aug 2026 | Brief allocated, client analysed | Complete |
| **Milestone 1 — Client design review** | **28 Aug 2026** | Client requirements, physical topology, logical topology, IP addressing plan, initial GitHub repository | **Submitted** |
| **Milestone 2 — Client implementation review** | **2 Oct 2026** | Packet Tracer build, WLAN challenge configured & verified, CR15 integrated, testing evidence | **Complete** |
| Final submission | 16 Oct 2026 | .pkt file, full portfolio, 15–20 min video demonstration | Not started |

## 4. Quick reference — addressing summary

| VLAN | Name | Subnet | Gateway | Usable hosts |
|---|---|---|---|---|
| 10 | Sales / Brokers | 192.168.40.0/27 | 192.168.40.1 | 30 |
| 20 | Admin & Management | 192.168.40.32/28 | 192.168.40.33 | 14 |
| 30 | Boardroom | 192.168.40.48/28 | 192.168.40.49 | 14 |
| 99 | Server / Infrastructure | 192.168.40.64/29 | 192.168.40.65 | 6 |
| — | WAN Link 1 (R3-CORE ↔ R1-EDGE-PRI) | 192.168.40.72/30 | R3 .73 · R1 .74 | 2 |
| — | WAN Link 2 (R3-CORE ↔ R2-EDGE-BAK) | 192.168.40.76/30 | R3 .77 · R2 .78 | 2 |
| — | Router–ISP links | 203.0.113.0/30 (ISP-A), 198.51.100.0/30 (ISP-B) | ISP .1 · edge router .2 | 2 each |
| — | Unallocated (growth) | 192.168.40.80 – 192.168.40.255 | — | — |

Static assignments: Board-Presenter-PC `192.168.40.50`, File-Server `192.168.40.66`, Print-Server `192.168.40.67`.
DHCP pools on R3-CORE serve VLANs 10, 20 and 30 (boardroom leases `.51 – .62`).

Full detail: [`docs/Milestone2_Client_Implementation_Review_41961366.docx`](docs/Milestone2_Client_Implementation_Review_41961366.docx) (Section 3).

## 5. Milestone 2 — implementation summary

### 5.1 Devices

| Device | Model | Role |
|---|---|---|
| R1-EDGE-PRI | 2911 | Primary edge router (ISP-A), NAT overload |
| R2-EDGE-BAK | 2911 | Backup edge router (ISP-B), NAT overload |
| R3-CORE | 2911 | Inter-VLAN routing, DHCP, default routes, boardroom ACL |
| SW1-CORE | 3650-24PS | Core switch, 802.1Q trunks to the access switches |
| SW2-SALES, SW3-ADMIN, SW4-BOARD, SW5-SRV | 2960-24TT | Access switches |
| AP1-BOARD | Access Point-PT | Boardroom wireless access point |
| ISP-A, ISP-B | 2911 | Simulated ISPs; loopback `8.8.8.8` stands in for an Internet host |

### 5.2 Assigned challenge — Wireless LAN (AP integration & coverage)

- AP1-BOARD connects to SW4-BOARD Fa0/2 (access port, VLAN 30). The wired presenter PC is on SW4-BOARD Fa0/1, also VLAN 30, so the boardroom has dedicated wired and wireless presentation connections in one VLAN.
- SSID `KAMOGELO-BOARD`, WPA2-PSK, AES, 2.4 GHz channel 1.
- Board-Laptop, Board-Phone and Board-Tablet receive DHCP leases from R3-CORE (`192.168.40.51 – .53`) with gateway `192.168.40.49`.
- The wireless laptop reaches the wired presenter PC with TTL 128 (same VLAN, not routed), the File-Server (routed by R3-CORE) and `8.8.8.8`.
- A client with a wrong passphrase fails to associate (IPv4 `0.0.0.0`) and reconnects once the correct passphrase is entered.

### 5.3 CR15 — second Internet connection

- Two independent edge routers, one per ISP, both connected to R3-CORE.
- R3-CORE default routes: primary via R1 (`192.168.40.74`, AD 1) and floating static backup via R2 (`192.168.40.78`, AD 10).
- Failover test: with the primary path down, R3-CORE installs the backup default and traffic reaches `8.8.8.8` through R2; restoring the primary returns the default route to `.74`.

### 5.4 Boardroom isolation

Extended ACL `BOARD-IN`, applied inbound on R3-CORE Gi0/0.30, denies boardroom traffic to the Sales and Admin subnets and permits everything else. Boardroom devices keep access to the server VLAN and the Internet; because the boardroom's replies are also blocked, Sales and Admin cannot reach it either.

## 6. Testing summary

Screenshots are in [`evidence/screenshots/`](evidence/screenshots/), named by test ID, and explained in
[`docs/Milestone2_Testing_Evidence_41961366.docx`](docs/Milestone2_Testing_Evidence_41961366.docx).

| ID | Test | Result |
|---|---|---|
| T01 | VLANs created and access ports assigned on all five switches | Pass |
| T02 | Trunks between SW1-CORE and the access switches | Pass |
| T03 | DHCP leases issued by R3-CORE (wired and wireless clients) | Pass |
| T04a | AP1-BOARD wireless configuration (SSID, WPA2-PSK, AES, channel 1) | Pass |
| T04b | Wrong passphrase is rejected; client reconnects with the correct one | Pass |
| T05 | Wireless clients addressed 192.168.40.51-.53, gateway .49 | Pass |
| T06 | Wireless laptop to wired presenter PC (.50), same VLAN | Pass |
| T07 | Wireless laptop to File-Server (.66) | Pass |
| T08 | Wireless laptop to 8.8.8.8 | Pass |
| T09 | Boardroom isolation ACL `BOARD-IN` (blocked to/from Sales and Admin; server and Internet open) | Pass |
| T10 | Inter-VLAN routing from Sales-PC1 (before the ACL) | Pass |
| T11 | Internet path via the primary ISP (tracert via .74) | Pass |
| T12 | Failover: default route `[10/0]` via .78 when the primary path is down | Pass |
| T13 | Traffic over the backup path; R2 reaches ISP-B and 8.8.8.8 | Pass |
| T14 | Restore: default route back to `[1/0]` via .74 | Pass |
| T15 | NAT overload translations on R1-EDGE-PRI | Pass |
| T16 | All wireless clients associated with AP1-BOARD | Pass |

## 7. Notes and limitations

- **Gateways:** the Milestone 1 address table labels the gateways "SVI". The implementation follows the Milestone 1 routing design (R3-CORE performs all inter-VLAN routing), so the gateways are router subinterfaces (`Gi0/0.10`, `.20`, `.30`, `.99`) with the same addresses.
- **Beyond the Milestone 1 text:** the addressing of the two router–ISP links (outside the client block) and the boardroom isolation ACL.
- **Failover scope:** floating static routes detect loss of the R3–R1 link or of R1, but not an ISP outage beyond R1. IP SLA / route tracking was not available in the Packet Tracer version used.
- A single standalone AP covers the boardroom; there is no wireless LAN controller (as justified in Milestone 1).
- Packet Tracer's wireless range is a simulation setting, not an RF survey.
- The Internet is simulated by loopback `8.8.8.8` on each ISP router.
- No device hardening (enable secrets, SSH) was applied for this milestone.

## 8. Academic integrity

This repository contains my own individual work for project **CMPG325-2026-089**.
Where AI assistance was used it was used in line with the NWU AI Policy; I remain responsible for the
correctness, understanding and verification of everything submitted. No other student's project brief,
design or evidence is included in this repository.
