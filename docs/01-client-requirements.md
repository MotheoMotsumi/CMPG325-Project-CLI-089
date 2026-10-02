# 01 - Client requirements

Project **CMPG325-2026-089** - Client **CLI-089** - Kamogelo Insurance Brokers (Vryburg), Professional Services.

## 1. Requirements from the approved brief

| Item | Requirement |
|---|---|
| Assigned organisation | Kamogelo Insurance Brokers (Vryburg) |
| Industry | Professional Services |
| Addressing block | 192.168.40.0/24 (single block, subnetted internally with VLSM) |
| Connectivity and services | A segmented LAN with inter-VLAN routing, a DHCP-ready address plan and reliable Internet access |
| Design constraint | The boardroom requires dedicated wired and wireless presentation ports |
| Change request (CR15) | A second Internet connection must be integrated for network resilience |
| Assigned networking challenge | Wireless LAN (AP integration and coverage), Foundational difficulty |
| Deliverable | A fully working, testable Cisco Packet Tracer implementation |

## 2. Assumptions

The brief does not describe the internal structure of the brokerage, so four functional groups were assumed:

- **Sales / Brokers** - the largest, client-facing group.
- **Admin and Management** - reception, administrative support and management.
- **Boardroom** - taken directly from the brief's design constraint.
- **Server / Infrastructure** - a small segment for shared file and print services.

## 3. How each requirement is met

| Requirement | Where it is met |
|---|---|
| Segmented LAN with inter-VLAN routing | VLANs 10, 20, 30 and 99; routing on R3-CORE ([03](03-logical-topology.md)) |
| DHCP-ready address plan | VLSM plan and DHCP pools on R3-CORE ([04](04-ip-addressing-plan.md)) |
| Reliable Internet access | Two edge routers, two ISPs, NAT overload ([07](07-cr15-dual-internet.md)) |
| Boardroom wired and wireless ports | SW4-BOARD with a wired port and AP1-BOARD in VLAN 30 ([02](02-physical-topology.md), [06](06-wireless-lan-design.md)) |
| CR15 second Internet connection | R2-EDGE-BAK and ISP-B with floating static routes ([07](07-cr15-dual-internet.md)) |
| Wireless LAN challenge | AP1-BOARD, WPA2-PSK, three wireless clients ([06](06-wireless-lan-design.md)) |
| Testable implementation | Packet Tracer file and test plan ([08](08-testing-plan.md)) |
