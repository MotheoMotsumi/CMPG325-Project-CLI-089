# 6. Wireless LAN Design — AP Integration & Coverage

**Assigned networking challenge (Brief §9):** Wireless LAN (AP integration & coverage) — *Foundational*
**Related constraint (Brief §8):** Boardroom requires dedicated wireless and wired presentation ports

This document is the design intent for Milestone 1. Configuration output and verification screenshots are
added under `configs/` and `evidence/` during Milestone 2.

---

## 6.1 Wireless requirements restated

| ID | Requirement |
|---|---|
| W-01 | Staff wireless coverage across the whole office floor with roaming between areas |
| W-02 | Separate guest wireless for walk-in clients, Internet-only |
| W-03 | Dedicated wireless service and coverage in the boardroom |
| W-04 | Each SSID mapped to its own VLAN/subnet |
| W-05 | Central management of all APs (no per-AP configuration) |
| W-06 | Signal ≥ −67 dBm in all working areas, non-overlapping channels between neighbouring cells |

## 6.2 Architecture — split-MAC with a controller

```
Client ──802.11──► LAP (AP1/AP2/AP3)  ──CAPWAP tunnel (VLAN 99)──►  WLC-1 (192.168.40.198)
                                                                       │ 802.1Q trunk
                                                                  SW-CORE ──► R1-EDGE Gi0/0.40/.50/.60 ──► routed
```

- APs are **lightweight**: they do radio work, the WLC owns configuration, WLAN-to-VLAN mapping and security.
- APs sit in the management VLAN 99 and reach the WLC's management interface to join it.
- The WLC's uplink to `SW-CORE` is an 802.1Q trunk carrying VLANs 40, 50, 60 (client traffic) and 99 (management).
- Adding a fourth AP later means cabling it and letting it join the controller — no new configuration.

## 6.3 WLAN (SSID) plan

| WLAN ID | SSID | Audience | Mapped VLAN / subnet | Security | Broadcast | Notes |
|---|---|---|---|---|---|---|
| 1 | `KIB-BOARD` | Boardroom presenters, visiting underwriters | 40 / 192.168.40.80/28 | WPA2-PSK (AES) | Yes | Advertised on **AP3 only** — dedicated boardroom service (C-01) |
| 2 | `KIB-STAFF` | KIB employees (laptops, tablets, phones) | 50 / 192.168.40.96/27 | WPA2-PSK (AES) | Yes | Advertised on AP1 + AP2 (and AP3 for staff continuity) |
| 3 | `KIB-GUEST` | Walk-in clients and visitors | 60 / 192.168.40.128/26 | Open with client isolation (WPA2-PSK printed at reception in production) | Yes | Advertised on AP1 + AP2; Internet-only by ACL |

Guest clients are denied to all internal subnets at `R1-EDGE Gi0/0.60`; only DHCP and DNS to
192.168.40.66 plus outbound Internet are permitted.

## 6.4 AP placement, channels and power

| AP | Location | Coverage responsibility | Band / channel (2.4 GHz) | 5 GHz | Tx power | Uplink |
|---|---|---|---|---|---|---|
| `AP1` | Ceiling, centre of broker/sales floor | Broker floor, corridor, spill into admin | Channel 1 | 36 | Medium (3) | `SW-A1 Fa0/24`, VLAN 99, PoE |
| `AP2` | Ceiling above reception, facing admin | Reception, waiting area, admin/finance, manager's office | Channel 6 | 40 | Medium (3) | `SW-A2 Fa0/24`, VLAN 99, PoE |
| `AP3` | Ceiling, centre of boardroom | Boardroom (dedicated) | Channel 11 | 44 | Low–medium (4) | `SW-A3 Fa0/24`, VLAN 99, PoE |

**Channel plan rationale:** 1 / 6 / 11 are the only non-overlapping 20 MHz channels in the 2.4 GHz band, so
adjacent cells do not interfere. `AP3` uses channel 11 and a reduced transmit power so the boardroom cell stays
in the boardroom rather than bleeding into the broker floor and competing with `AP1`.

**Coverage intent:** APs are spaced so that neighbouring cells overlap by roughly 15–20% at the −67 dBm
contour, which is enough for a laptop to roam from the broker floor to reception without dropping a session,
but not so much that co-channel contention appears. The boardroom's internal walls attenuate `AP1`/`AP2`
signal, which is precisely why the constraint requires a dedicated AP rather than relying on spill-over.

## 6.5 Boardroom: how the constraint is satisfied

| Element | Implementation |
|---|---|
| Dedicated wireless | `AP3` mounted in the boardroom ceiling, broadcasting `KIB-BOARD` (WLAN 1) mapped to VLAN 40. It is the only AP advertising that SSID. |
| Dedicated wired presentation ports | `SW-A3 Fa0/1–Fa0/4` cabled to two floor boxes at the boardroom table + `Fa0/5` to the wall display host. All access ports in VLAN 40, PortFast enabled. |
| Same experience either way | Both media land in VLAN 40, get DHCP from `POOL-BOARDROOM`, gateway 192.168.40.81, and the same permissions (HTTP/file to the server VLAN, plus Internet). |
| Independence | A failure of `AP3` leaves the wired ports working, and vice versa — the boardroom always has a working presentation path. |

## 6.6 Configuration outline (to be executed in Milestone 2)

1. **Switching prerequisites** — create VLANs 40/50/60/99 on `SW-CORE`, `SW-A1/A2/A3`; trunk to WLC with those
   VLANs allowed; AP switchports as access VLAN 99 with PoE enabled.
2. **WLC bring-up** — management interface 192.168.40.198/28 on VLAN 99, gateway 192.168.40.193, admin
   credentials, DHCP/DNS pointers.
3. **Dynamic interfaces on the WLC** — one per client VLAN: `int-board` (VLAN 40), `int-staff` (VLAN 50),
   `int-guest` (VLAN 60), each with an address in its subnet and the correct gateway.
4. **WLANs** — create the three SSIDs from §6.3, bind each to its dynamic interface, set WPA2-PSK on
   `KIB-BOARD`/`KIB-STAFF`, enable client isolation on `KIB-GUEST`, and restrict `KIB-BOARD` to `AP3`
   via an AP group.
5. **AP join** — cable AP1–AP3, confirm each obtains a VLAN 99 address and appears under *Wireless → All APs*.
6. **Radio settings** — set 2.4 GHz channels 1/6/11 and transmit power per §6.4.
7. **Client onboarding** — configure laptops/tablets to associate to the correct SSID with the PSK.

## 6.7 Verification plan (evidence to capture)

| # | Test | Expected result | Evidence |
|---|---|---|---|
| W-T1 | `show ap summary` / *All APs* page on WLC-1 | AP1, AP2, AP3 all listed as joined | WLC screenshot |
| W-T2 | Staff laptop associates to `KIB-STAFF` on the broker floor | Address from 192.168.40.100–.125, gateway .97 | `ipconfig` screenshot |
| W-T3 | Same laptop moved to reception | Stays associated, keeps VLAN 50 address (roaming AP1→AP2) | screenshot + note |
| W-T4 | Visitor tablet associates to `KIB-GUEST` | Address from 192.168.40.135–.189, gateway .129 | `ipconfig` screenshot |
| W-T5 | Guest tablet pings `SRV-FILE` (192.168.40.67) | **Fails** (ACL) | ping output |
| W-T6 | Guest tablet reaches external host / web server | Succeeds | browser/ping output |
| W-T7 | Boardroom laptop associates to `KIB-BOARD` in the boardroom | Address from 192.168.40.84–.93, gateway .81 | `ipconfig` screenshot |
| W-T8 | `KIB-BOARD` not visible from the broker floor | SSID absent / signal too weak | wireless scan screenshot |
| W-T9 | Laptop on boardroom floor-box port (`SW-A3 Fa0/1`) | VLAN 40 address, reaches display host and Internet | ping + `ipconfig` |
| W-T10 | Channel check on all three APs | Channels 1, 6, 11 respectively | WLC radio page |

## 6.8 Known limitations in simulation

- Packet Tracer does not model RF propagation, walls or real signal strength, so the −67 dBm coverage target is
  a design statement supported by placement and channel/power choices rather than a measured survey result.
- 802.1X / WPA2-Enterprise with RADIUS is only partially modelled; WPA2-PSK is used in the simulation while the
  documented production recommendation for `KIB-STAFF` is WPA2-Enterprise.
- Band steering, RRM auto-channel and client load balancing are described here as design intent but are set
  manually in the simulation.
