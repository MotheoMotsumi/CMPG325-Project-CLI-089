# 1. Client Requirements

**Project ID:** CMPG325-2026-089 · **Client ID:** CLI-089
**Organisation:** Kamogelo Insurance Brokers (Vryburg) · **Industry:** Professional Services
**Author:** M. Motsumi (41961366) · **Milestone 1 — Client Design Review (28 August 2026)**

---

## 1.1 Client profile

Kamogelo Insurance Brokers (KIB) is an independent short-term and life insurance brokerage operating from a
single leased office in Vryburg, North West. The business advises walk-in and telephonic clients, captures
policy applications, submits claims to underwriters, and stores client documentation (ID copies, policy
schedules, claim evidence) that must remain private and available during office hours.

The office is one storey with the following working areas, which form the basis of the network segmentation:

| # | Area | Users / devices | Notes |
|---|---|---|---|
| 1 | Reception & walk-in client area | 2 staff PCs, 1 shared printer, visitor seating | Visitors expect Wi-Fi while waiting |
| 2 | Broker / sales floor | 14 broker PCs, 4 laptops, 1 MFP | Highest user count; heavy use of insurer web portals |
| 3 | Administration & finance | 8 PCs, 1 MFP | Handles premium collections and payroll — must be isolated from guests |
| 4 | IT / server room | 3 servers, network equipment, 1 admin workstation | DHCP/DNS, file server, backup |
| 5 | Boardroom | Presentation display, 4 wired presentation ports, visiting underwriters/clients with laptops | Dedicated wireless coverage + dedicated wired ports required |
| 6 | Manager's office | 2 PCs | Placed with Administration & finance |

Peak concurrent devices: approximately **60–70**, of which roughly half are wireless.

## 1.2 Business drivers

| Driver | Why it matters to KIB |
|---|---|
| Continuous Internet availability | Quotations, policy submissions and claims are all done on insurer web portals. An outage stops billable work immediately. |
| Client data confidentiality | Personal information of policyholders is processed; guest and staff traffic must not share a broadcast domain (POPIA-aligned good practice). |
| Professional client experience | Walk-in clients and visiting underwriters expect working guest Wi-Fi and a boardroom that "just works" for presentations. |
| Low operating overhead | KIB has no full-time network engineer; the design must be centrally manageable and simple to support remotely. |
| Room to grow | The brokerage plans to add staff; addressing and switch capacity must allow growth without redesign. |

## 1.3 Functional requirements

| ID | Requirement | Source |
|---|---|---|
| FR-01 | Provide wired connectivity for all staff PCs, printers/MFPs and servers across the six working areas. | Brief §6 |
| FR-02 | Provide wireless connectivity for staff laptops and mobile devices with full coverage of the office floor. | Brief §9 |
| FR-03 | Provide a separate guest wireless service for walk-in clients and visitors, isolated from internal systems. | Client profile |
| FR-04 | Segment the network per business function (brokers, admin/finance, servers/IT, boardroom, staff Wi-Fi, guest Wi-Fi, management). | Brief §6, §7 |
| FR-05 | Route between segments so that authorised nodes can exchange data end-to-end. | Brief §5 |
| FR-06 | Provide core network services: DHCP for user and wireless VLANs, DNS, internal file/intranet server. | Brief §6 |
| FR-07 | Provide Internet access for all internal VLANs via address translation. | Brief §6 |
| FR-08 | **Boardroom:** provide dedicated wireless coverage (own SSID/segment) **and** dedicated wired presentation ports at the boardroom table. | Brief §8 (constraint) |
| FR-09 | **CR15:** integrate a second Internet connection so that the site keeps working if the primary link fails. | Brief §10 |
| FR-10 | Provide remote management access to all network devices from a dedicated management segment. | Low-overhead driver |

## 1.4 Technical requirements

| ID | Requirement |
|---|---|
| TR-01 | Base the entire IP addressing plan on the assigned block **192.168.40.0/24**, using VLSM so that subnet sizes match host counts and space is reserved for growth. |
| TR-02 | Implement Layer 2 segmentation with VLANs and 802.1Q trunking between switches and the router. |
| TR-03 | Implement inter-VLAN routing (router-on-a-stick sub-interfaces) on the edge router. |
| TR-04 | Implement a centrally managed WLAN: a wireless LAN controller with lightweight access points, three WLANs mapped to VLANs 40, 50 and 60. |
| TR-05 | Secure staff and boardroom WLANs with WPA2 (PSK in simulation); guest WLAN separated and Internet-only. |
| TR-06 | Plan AP placement and non-overlapping 2.4 GHz channels (1 / 6 / 11) to achieve ≥ −67 dBm in all working areas, including the boardroom. |
| TR-07 | Provide DHCP pools per user/wireless VLAN with static addressing for servers, network devices and printers. |
| TR-08 | Configure NAT/PAT on both Internet uplinks and floating static default routes so the secondary link takes over automatically. |
| TR-09 | Devices must be modelled with equipment available in Cisco Packet Tracer and the final `.pkt` file must open and reproduce the working solution. |
| TR-10 | Management VLAN 99 with SVI/interface addressing on all switches, APs and the WLC. |

## 1.5 Constraints

| ID | Constraint | Design impact |
|---|---|---|
| C-01 | **Assigned constraint:** the boardroom requires dedicated wireless and wired presentation ports. | Dedicated boardroom VLAN 40, a dedicated boardroom AP (AP3) with its own SSID, and four access-switch ports patched to floor boxes at the boardroom table. |
| C-02 | Single addressing block `192.168.40.0/24` for the whole internal network. | VLSM required; guest VLAN is the largest subnet, /28s used for small segments, one /28 held in reserve. |
| C-03 | Simulation must be built in Cisco Packet Tracer. | Device selection limited to Packet Tracer models (2911 router, 3560/2960 switches, WLC 2504, LAP-PT/AP, generic servers). |
| C-04 | Single-storey leased premises; limited cabling routes and one comms cabinet in the IT room. | Star topology from the IT room; access switches per area with fibre/copper uplinks to the core switch. |
| C-05 | No dedicated on-site network administrator. | Centralised WLC management, consistent naming, documented configurations. |
| C-06 | Small-business budget. | One edge router with two WAN interfaces rather than two routers; L2 access switches rather than a full L3 access layer. |

## 1.6 Assumptions

1. Both Internet services are ordered from different providers (fibre primary, LTE/wireless secondary) and each hands off a routed /30 to KIB.
2. Provider-supplied equipment is presented as a router/CPE in the simulation (`ISP-A`, `ISP-B`).
3. Structured cabling (Cat6 to desks, floor boxes in the boardroom) exists or is installed as part of the project.
4. VoIP telephony is out of scope for this project; QoS is therefore not a requirement.
5. Server operating systems and applications are out of scope — servers are modelled for DHCP, DNS, HTTP and file services only.
6. Physical security of the IT room is the client's responsibility.

## 1.7 Acceptance criteria (how the client will judge success)

| ID | Criterion | Verification |
|---|---|---|
| AC-01 | Any staff PC can reach the server VLAN and the Internet. | `ping` / web request from PCs in VLAN 10, 20, 40; browse to server and to an external host |
| AC-02 | Guest wireless clients reach the Internet but **cannot** reach VLAN 10/20/30/99. | `ping` from guest laptop to server (must fail) and to external host (must succeed) |
| AC-03 | Staff laptops associate to the staff SSID anywhere on the floor and receive a VLAN 50 address. | Wireless association + `ipconfig` on laptops at each area |
| AC-04 | Boardroom has its own SSID with signal in the boardroom, and four wired ports at the table that give a VLAN 40 address. | Association test in boardroom; link + DHCP test on each presentation port |
| AC-05 | Disabling the primary Internet link does not stop Internet access. | Shut `Gi0/1` on R1, confirm route change and continued external connectivity |
| AC-06 | All network devices are reachable for management from the management segment. | SSH/Telnet from admin workstation to switches, WLC and APs |
| AC-07 | The submitted `.pkt` file opens and reproduces all of the above. | Fresh open of the file and re-run of the test plan |

## 1.8 Out of scope

IP telephony and QoS · site-to-site VPN or remote-worker VPN · redundant internal switching (spanning-tree
redundancy beyond default) · server application build-out · public DNS hosting · firewall appliance (edge
filtering is done with router ACLs) · a second physical site.

---

### Traceability

| Brief item | Addressed by |
|---|---|
| §6 Client requirements | FR-01…FR-07, TR-01 |
| §7 Network design requirements | TR-01…TR-10, `docs/02`, `docs/03`, `docs/04` |
| §8 Design constraint (boardroom) | C-01, FR-08, `docs/06-wireless-lan-design.md` |
| §9 Networking challenge (Wireless LAN) | FR-02, FR-03, TR-04…TR-06, `docs/06-wireless-lan-design.md` |
| §10 Change request CR15 | FR-09, TR-08, `docs/07-cr15-dual-internet.md` |
| §11 Testing requirements | AC-01…AC-07, `docs/08-testing-plan.md` |
