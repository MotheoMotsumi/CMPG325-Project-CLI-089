# 7. Client Change Request CR15 — Second Internet Connection for Resilience

**CR15 (Brief §10):** *A second Internet connection is added for resilience and must be integrated.*

---

## 7.1 What the change means for KIB

Brokers cannot quote, submit policies or lodge claims without reaching insurer web portals. A single fibre
failure therefore stops revenue-generating work. CR15 adds a second, independent Internet service so that the
office keeps working through a primary-link or provider failure. The requirement is **resilience**, not extra
bandwidth — so the design provides automatic failover on a clearly defined primary/secondary basis rather than
load sharing.

## 7.2 How it is integrated into the existing design

| Aspect | Before CR15 | After CR15 |
|---|---|---|
| WAN interfaces on `R1-EDGE` | Gi0/1 to ISP-A only | Gi0/1 to ISP-A (primary fibre), **Gi0/2 to ISP-B (secondary LTE/wireless)** |
| WAN addressing | 209.165.200.224/30 | plus **209.165.201.0/30** |
| Default routing | One static default | **Two static defaults, second one floating (AD 5)** |
| NAT | PAT on Gi0/1 | **PAT on both outside interfaces, selected by route-map** |
| Edge filtering | Inbound ACL on Gi0/1 | **Same inbound ACL applied to Gi0/2** |
| LAN side | Unchanged | Unchanged — no VLAN, addressing or wireless change is needed |

Because both links terminate on the same router and all internal VLANs already default to that router, the
change is contained entirely at the edge. That is the main reason a single-router, two-WAN design was chosen
over adding a second router (see DD-01, DD-06).

## 7.3 Routing behaviour

```
ip route 0.0.0.0 0.0.0.0 209.165.200.225        ← via ISP-A, administrative distance 1 (installed, preferred)
ip route 0.0.0.0 0.0.0.0 209.165.201.1 5        ← via ISP-B, administrative distance 5 (floating, standby)
```

- Normal state: only the ISP-A route is in the routing table; all Internet traffic uses the fibre.
- Failure state: when `Gi0/1` loses line protocol (or the tracked object goes down), the ISP-A route is
  withdrawn and the AD 5 route is installed automatically; traffic continues via ISP-B.
- Recovery: when `Gi0/1` comes back, the AD 1 route reinstalls and preempts the standby path.

**Production improvement (documented, partially simulated):** in a real deployment the primary route would be
tied to `ip sla` + `track` reachability of an off-net address, so that a link that is physically up but
logically broken also triggers failover. Packet Tracer's support for IP SLA tracking is limited, so the
simulation demonstrates interface-based failover and this refinement is recorded as a recommendation.

## 7.4 NAT behaviour across two exits

Two overload statements are used, each keyed to its own outside interface via a route-map, so translations are
built with whichever provider address is currently in use:

```
access-list 10 permit 192.168.40.0 0.0.0.255
route-map TO-ISPA permit 10
 match ip address 10
 match interface GigabitEthernet0/1
route-map TO-ISPB permit 10
 match ip address 10
 match interface GigabitEthernet0/2
ip nat inside source route-map TO-ISPA interface GigabitEthernet0/1 overload
ip nat inside source route-map TO-ISPB interface GigabitEthernet0/2 overload
```

Existing translations are cleared on failover (`clear ip nat translation *` in testing) because the outside
address changes; users may need to reload a page once, which is acceptable for a resilience-only requirement.

## 7.5 Failover test plan (evidence to capture in Milestone 2)

| # | Test | Action | Expected result |
|---|---|---|---|
| CR-T1 | Steady-state path | `show ip route` on `R1-EDGE` | Single default via 209.165.200.225 (ISP-A) |
| CR-T2 | Steady-state reachability | Ping/browse external host from a broker PC | Success, traced via ISP-A |
| CR-T3 | Translation check | `show ip nat translations` | Inside global addresses = 209.165.200.226 |
| CR-T4 | Primary failure | `interface Gi0/1` → `shutdown` | ISP-A route withdrawn; default via 209.165.201.1 appears |
| CR-T5 | Failover reachability | Repeat CR-T2 during outage | Success via ISP-B (traceroute shows the new next hop) |
| CR-T6 | Translation after failover | `show ip nat translations` | Inside global addresses = 209.165.201.2 |
| CR-T7 | Wireless clients during outage | Guest and staff Wi-Fi clients browse externally | Success — failover is transparent to the WLAN |
| CR-T8 | Recovery | `no shutdown` on Gi0/1 | Primary route reinstalls, traffic returns to ISP-A |

## 7.6 Scope discipline

CR15 is implemented exactly as worded — one additional Internet connection, integrated for resilience. No
additional scope (second router, VPN, SD-WAN, load balancing, provider BGP) is introduced. Alternatives that
were considered and deliberately excluded are recorded in DD-06.
