# 3. Logical Topology

**Project:** CMPG325-2026-089 · Kamogelo Insurance Brokers (Vryburg)

![Logical topology](../diagrams/logical-topology.png)
*Source file: [`diagrams/logical-topology.svg`](../diagrams/logical-topology.svg)*

---

## 3.1 Logical model

The logical design is a **single-router, VLAN-segmented Layer 2 domain with router-on-a-stick inter-VLAN
routing and two default paths to the Internet**:

```
Internet
  ├── ISP-A (primary)   209.165.200.224/30 ── R1-EDGE Gi0/1  [static default, AD 1]
  └── ISP-B (secondary) 209.165.201.0/30   ── R1-EDGE Gi0/2  [floating static default, AD 5]
                                                   │
                                    R1-EDGE Gi0/0 (802.1Q trunk)
                    Gi0/0.10 .20 .30 .40 .50 .60 .99  ← default gateways / SVI equivalents
                                                   │
                                              SW-CORE (L2 core, VTP/VLAN owner, trunks)
                     ┌──────────────┬────────────┬─────────────┬──────────────┐
                   SW-A1          SW-A2        SW-A3         WLC-1        Servers (VLAN 30)
                 VLAN 10        VLAN 20      VLAN 40      WLAN→VLAN 40/50/60
                   + AP1          + AP2        + AP3
```

Every user segment is a separate VLAN = separate IP subnet = separate broadcast domain. All routing between
segments happens once, on `R1-EDGE`, which is also the single point where Internet-bound traffic is translated
and where inter-VLAN policy (ACLs) is enforced.

## 3.2 VLAN plan

| VLAN | Name | Purpose | Subnet | Gateway (R1 sub-if) | Where it appears |
|---|---|---|---|---|---|
| 10 | `BROKERS` | Broker/sales PCs and MFP | 192.168.40.0/27 | 192.168.40.1 (Gi0/0.10) | SW-A1 access ports |
| 20 | `ADMIN-FIN` | Administration, finance, reception, manager | 192.168.40.32/27 | 192.168.40.33 (Gi0/0.20) | SW-A2 access ports |
| 30 | `SERVERS-IT` | DHCP/DNS, file, backup servers | 192.168.40.64/28 | 192.168.40.65 (Gi0/0.30) | SW-CORE access ports |
| 40 | `BOARDROOM` | Boardroom wired presentation ports **and** boardroom SSID | 192.168.40.80/28 | 192.168.40.81 (Gi0/0.40) | SW-A3 access ports + WLAN `KIB-BOARD` |
| 50 | `WLAN-STAFF` | Staff wireless (laptops, tablets, phones) | 192.168.40.96/27 | 192.168.40.97 (Gi0/0.50) | WLAN `KIB-STAFF` via WLC trunk |
| 60 | `WLAN-GUEST` | Visitor / walk-in client wireless, Internet-only | 192.168.40.128/26 | 192.168.40.129 (Gi0/0.60) | WLAN `KIB-GUEST` via WLC trunk |
| 99 | `MGMT` | Switch/AP/WLC management, admin workstation | 192.168.40.192/28 | 192.168.40.193 (Gi0/0.99) | SVIs, AP ports, WLC mgmt, ADMIN-PC |
| 999 | `BLACKHOLE` | Unused-port parking VLAN (no IP) | — | — | All unused access ports |

VLAN 1 is not used for user data or management. The native VLAN on all trunks is set to an unused VLAN (`1000`)
rather than left at VLAN 1.

## 3.3 Layer 2 design

| Item | Decision |
|---|---|
| Trunking | 802.1Q, statically configured (`switchport mode trunk`), allowed-VLAN lists pruned per link |
| VLAN propagation | VLANs created manually on each switch (VTP transparent) — small network, avoids VTP accidents |
| Spanning tree | PVST+ (default); `SW-CORE` configured as root bridge for all VLANs with a low priority |
| Edge port protection | PortFast + BPDU Guard on all access ports (including boardroom presentation ports) |
| Unused ports | Administratively shut and assigned to VLAN 999 |
| Port security | Sticky MAC with a low maximum on admin/finance and boardroom ports (optional hardening) |

## 3.4 Layer 3 design

| Item | Decision |
|---|---|
| Inter-VLAN routing | Router-on-a-stick: seven 802.1Q sub-interfaces on `R1-EDGE Gi0/0`, one per VLAN, each the default gateway for its subnet |
| Internal routing protocol | None — all networks are directly connected to `R1-EDGE`, so no dynamic routing is required |
| Default routing | Two static default routes: `0.0.0.0/0` via ISP-A (default AD 1) and `0.0.0.0/0` via ISP-B with AD 5 (floating) — see [`07-cr15-dual-internet.md`](07-cr15-dual-internet.md) |
| Address translation | NAT overload (PAT) on `R1-EDGE`, one pool/interface per ISP, matched by ACL on the internal 192.168.40.0/24 space |
| DHCP | Central server `SRV-DHCP-DNS` (192.168.40.66) with a scope per user VLAN; `ip helper-address 192.168.40.66` on sub-interfaces 10, 20, 40, 50, 60 |
| DNS | `SRV-DHCP-DNS` resolves internal names and forwards externally |
| Static addressing | Servers, printers/MFPs, switches, WLC, APs and all router interfaces are statically addressed |

## 3.5 Wireless logical integration

```
Wireless client ── SSID ── WLC WLAN ── mapped VLAN ── R1 sub-interface ── routed
KIB-BOARD  (WPA2-PSK) → WLAN 1 → VLAN 40 → 192.168.40.80/28  → boardroom, Internet + internal
KIB-STAFF  (WPA2-PSK) → WLAN 2 → VLAN 50 → 192.168.40.96/27  → full internal + Internet
KIB-GUEST  (open/PSK, isolated) → WLAN 3 → VLAN 60 → 192.168.40.128/26 → Internet only
```

APs are lightweight and live in VLAN 99; they build CAPWAP tunnels to `WLC-1` and carry client traffic to the
controller, which places it into the mapped VLAN on its trunk to `SW-CORE`. Full detail and the coverage plan
are in [`06-wireless-lan-design.md`](06-wireless-lan-design.md).

## 3.6 Traffic policy (logical security boundaries)

| Source | Destination | Policy |
|---|---|---|
| VLAN 10, 20 | VLAN 30 (servers) | Permit (file, DHCP, DNS, HTTP, FTP) |
| VLAN 10 | VLAN 20 | Permit (business workflow between brokers and finance) |
| VLAN 40 (boardroom) | VLAN 30 | Permit HTTP/file only — presentations pull material from the file server |
| VLAN 50 (staff Wi-Fi) | VLAN 10, 20, 30 | Permit — same staff, different medium |
| VLAN 60 (guest) | VLAN 10, 20, 30, 40, 99 | **Deny** (client-data protection, AC-02) |
| VLAN 60 (guest) | Internet | Permit, after DHCP/DNS from the server |
| Any user VLAN | VLAN 99 (management) | **Deny**, except the ADMIN-PC in VLAN 99 itself |
| Internet | Inside | Deny unsolicited inbound (stateful behaviour of PAT + inbound ACL on both WAN interfaces) |

Policy is applied as extended ACLs inbound on the relevant `R1-EDGE` sub-interfaces, so a single device holds
the entire policy — appropriate for a client with no on-site network staff.

## 3.7 Naming and management conventions

| Element | Convention | Example |
|---|---|---|
| Routers | `R<n>-ROLE` | `R1-EDGE` |
| Switches | `SW-CORE`, `SW-A<n>` | `SW-A2` |
| Wireless | `WLC-1`, `AP<n>` | `AP3` |
| Servers | `SRV-<function>` | `SRV-FILE` |
| SSIDs | `KIB-<audience>` | `KIB-GUEST` |
| Management | All devices addressed from VLAN 99, SSH enabled, banner + local admin user, `.1xx` host range | `SW-A1` = 192.168.40.195 |
