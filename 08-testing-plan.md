# 8. Testing and Verification Plan

**Project:** CMPG325-2026-089 · Kamogelo Insurance Brokers (Vryburg)
Covers Brief §11 (Testing requirements). Results and screenshots are recorded in `evidence/` from Milestone 2.

---

## 8.1 Test matrix

| # | Category | Test | Method | Expected result | Acceptance criterion |
|---|---|---|---|---|---|
| T-01 | Layer 2 | VLANs exist and ports are assigned | `show vlan brief` on all switches | VLANs 10/20/30/40/50/60/99/999 present with correct ports | FR-04 |
| T-02 | Layer 2 | Trunks are up with correct allowed VLANs | `show interfaces trunk` | Trunks to router, access switches and WLC operational | TR-02 |
| T-03 | Layer 2 | STP root is `SW-CORE` | `show spanning-tree` | `SW-CORE` root for all VLANs | §3.3 |
| T-04 | Layer 3 | Sub-interfaces up with correct addresses | `show ip interface brief`, `show ip route` on `R1-EDGE` | 7 connected subnets present | TR-03 |
| T-05 | Addressing | DHCP works per VLAN | `ipconfig /renew` on a client in VLAN 10, 20, 40, 50, 60 | Address from the correct pool, correct gateway and DNS | TR-07 |
| T-06 | Intra-VLAN | Same-VLAN connectivity | Ping between two broker PCs | Success | FR-01 |
| T-07 | Inter-VLAN | Broker PC → file server | Ping/HTTP to 192.168.40.67 | Success | AC-01 |
| T-08 | Inter-VLAN | Admin PC → broker PC | Ping | Success | §3.6 |
| T-09 | Services | DNS resolution | Browse `intranet.kib.local` from a broker PC | Page loads via name | FR-06 |
| T-10 | Internet | Any staff VLAN → external host | Ping/browse external address | Success | AC-01 |
| T-11 | NAT | Translations built | `show ip nat translations` on `R1-EDGE` | Inside local ↔ inside global entries | TR-08 |
| T-12 | Wireless | All three APs joined the WLC | WLC *All APs* page | AP1, AP2, AP3 joined | W-T1 / AC-03 |
| T-13 | Wireless | Staff laptop on `KIB-STAFF` | Associate, `ipconfig` | VLAN 50 address | AC-03 |
| T-14 | Wireless | Roaming broker floor → reception | Move laptop, keep pinging server | No sustained loss, same address | W-T3 |
| T-15 | Wireless | Guest tablet on `KIB-GUEST` | Associate, `ipconfig` | VLAN 60 address | AC-02 |
| T-16 | Security | Guest → internal server | Ping 192.168.40.67 from guest | **Blocked** | AC-02 |
| T-17 | Security | Guest → Internet | Browse external host | Success | AC-02 |
| T-18 | Security | User VLAN → management VLAN | Ping 192.168.40.194 from a broker PC | **Blocked** | §3.6 |
| T-19 | Constraint | Boardroom SSID in boardroom | Associate boardroom laptop to `KIB-BOARD` | VLAN 40 address, reaches display + Internet | AC-04 |
| T-20 | Constraint | Boardroom wired presentation ports | Plug laptop into each of `SW-A3 Fa0/1–Fa0/4` | Link up, VLAN 40 address on every port | AC-04 |
| T-21 | CR15 | Primary path in use | `show ip route` | Default via ISP-A | AC-05 |
| T-22 | CR15 | Failover on primary loss | `shutdown` Gi0/1, retest T-10 | Internet still reachable via ISP-B | AC-05 |
| T-23 | CR15 | Recovery | `no shutdown` Gi0/1 | Traffic returns to ISP-A | AC-05 |
| T-24 | Management | SSH to all devices from ADMIN-PC | SSH to switches, WLC | Login succeeds | AC-06 |
| T-25 | Deliverable | `.pkt` reproducibility | Close and reopen the file, re-run T-05, T-10, T-19, T-22 | All pass on a fresh open | AC-07 |

## 8.2 Evidence to capture per test

For each test: the command or action, a screenshot of the output (Packet Tracer CLI, `ipconfig`, WLC page or
web browser), the pass/fail result, and a one-line comment. Screenshots are named
`T-<nn>-<short-description>.png` under `evidence/screenshots/`.

## 8.3 Troubleshooting log format

Issues found during the build are recorded in `evidence/troubleshooting/` as one file per issue:

```
Issue:        short title
Symptom:      what was observed
Diagnosis:    commands used and what they showed
Root cause:   the actual fault
Fix:          what was changed
Verification: which test now passes
```

Likely candidates on this design, recorded in advance so they are documented properly if they occur: APs not
joining the WLC (VLAN 99 addressing or DHCP), wireless clients getting no address (WLC dynamic interface mapped
to the wrong VLAN or trunk not allowing it), guest clients able to reach internal hosts (ACL direction or
placement), and failover not occurring (missing administrative distance on the secondary default route).
