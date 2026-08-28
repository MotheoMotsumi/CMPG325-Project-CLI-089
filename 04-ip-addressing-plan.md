# 4. IP Addressing Plan

**Project:** CMPG325-2026-089 · Kamogelo Insurance Brokers (Vryburg)
**Assigned block:** `192.168.40.0/24` (255 usable addresses, private RFC 1918)
**Method:** VLSM — subnets sized to measured host counts, largest-first, with reserved space for growth

---

## 4.1 Host count analysis

| Segment | Current devices | Growth allowance (~40%) | Required usable | Chosen mask | Usable provided |
|---|---|---|---|---|---|
| Guest wireless (VLAN 60) | ~20 concurrent visitor devices | to 50+ | 50 | /26 | 62 |
| Brokers / sales (VLAN 10) | 18 wired + MFP | to 26 | 26 | /27 | 30 |
| Admin / finance (VLAN 20) | 12 wired + MFP + reception | to 22 | 22 | /27 | 30 |
| Staff wireless (VLAN 50) | ~14 laptops/tablets | to 26 | 26 | /27 | 30 |
| Servers / IT (VLAN 30) | 3 servers | to 8 | 8 | /28 | 14 |
| Boardroom (VLAN 40) | 5 wired ports + ~6 wireless | to 12 | 12 | /28 | 14 |
| Management (VLAN 99) | 9 infrastructure devices | to 12 | 12 | /28 | 14 |
| WAN links (×2) | 2 hosts each | — | 2 | /30 | 2 |

## 4.2 Subnet allocation (VLSM, largest first)

| Order | VLAN | Name | Network address | Mask | CIDR | Usable range | Broadcast | Gateway |
|---|---|---|---|---|---|---|---|---|
| 1 | 60 | WLAN-GUEST | 192.168.40.128 | 255.255.255.192 | /26 | .129 – .190 | 192.168.40.191 | 192.168.40.129 |
| 2 | 10 | BROKERS | 192.168.40.0 | 255.255.255.224 | /27 | .1 – .30 | 192.168.40.31 | 192.168.40.1 |
| 3 | 20 | ADMIN-FIN | 192.168.40.32 | 255.255.255.224 | /27 | .33 – .62 | 192.168.40.63 | 192.168.40.33 |
| 4 | 50 | WLAN-STAFF | 192.168.40.96 | 255.255.255.224 | /27 | .97 – .126 | 192.168.40.127 | 192.168.40.97 |
| 5 | 30 | SERVERS-IT | 192.168.40.64 | 255.255.255.240 | /28 | .65 – .78 | 192.168.40.79 | 192.168.40.65 |
| 6 | 40 | BOARDROOM | 192.168.40.80 | 255.255.255.240 | /28 | .81 – .94 | 192.168.40.95 | 192.168.40.81 |
| 7 | 99 | MGMT | 192.168.40.192 | 255.255.255.240 | /28 | .193 – .206 | 192.168.40.207 | 192.168.40.193 |
| — | — | **Reserved (future segment)** | 192.168.40.208 | 255.255.255.240 | /28 | .209 – .222 | 192.168.40.223 | — |
| — | — | **Reserved (future segment)** | 192.168.40.224 | 255.255.255.224 | /27 | .225 – .254 | 192.168.40.255 | — |

**Block utilisation:** 208 of 256 addresses allocated (81%); 48 addresses (a /28 + a /27) held in reserve for a
future VoIP or CCTV segment, satisfying TR-01.

### WAN / provider addressing (outside the assigned block — provider assigned)

| Link | Subnet | KIB side (`R1-EDGE`) | Provider side | Purpose |
|---|---|---|---|---|
| ISP-A (primary fibre) | 209.165.200.224/30 | 209.165.200.226 (Gi0/1) | 209.165.200.225 | Primary default route + PAT |
| ISP-B (secondary LTE) | 209.165.201.0/30 | 209.165.201.2 (Gi0/2) | 209.165.201.1 | Floating default route + PAT (CR15) |

*Simulated public ranges (Cisco documentation space) are used because Packet Tracer needs routable "public"
addressing on the outside of NAT.*

## 4.3 Address assignment policy per subnet

| Range within each user subnet | Use |
|---|---|
| First usable address | Default gateway (router sub-interface) |
| Next 8 addresses | Static infrastructure and shared devices (printers/MFPs, display hosts) |
| Middle of range | DHCP pool |
| Last 1–2 addresses | Held for temporary/diagnostic use |

## 4.4 Static assignments

| Device | VLAN | IP address | Mask | Default gateway | Notes |
|---|---|---|---|---|---|
| `R1-EDGE` Gi0/0.10 | 10 | 192.168.40.1 | /27 | — | Gateway, BROKERS |
| `R1-EDGE` Gi0/0.20 | 20 | 192.168.40.33 | /27 | — | Gateway, ADMIN-FIN |
| `R1-EDGE` Gi0/0.30 | 30 | 192.168.40.65 | /28 | — | Gateway, SERVERS-IT |
| `R1-EDGE` Gi0/0.40 | 40 | 192.168.40.81 | /28 | — | Gateway, BOARDROOM |
| `R1-EDGE` Gi0/0.50 | 50 | 192.168.40.97 | /27 | — | Gateway, WLAN-STAFF |
| `R1-EDGE` Gi0/0.60 | 60 | 192.168.40.129 | /26 | — | Gateway, WLAN-GUEST |
| `R1-EDGE` Gi0/0.99 | 99 | 192.168.40.193 | /28 | — | Gateway, MGMT |
| `R1-EDGE` Gi0/1 | — | 209.165.200.226 | /30 | — | ISP-A, `ip nat outside` |
| `R1-EDGE` Gi0/2 | — | 209.165.201.2 | /30 | — | ISP-B, `ip nat outside` |
| `MFP-BROKERS` | 10 | 192.168.40.5 | /27 | 192.168.40.1 | Static so print queues never break |
| `MFP-ADMIN` | 20 | 192.168.40.37 | /27 | 192.168.40.33 | |
| `SRV-DHCP-DNS` | 30 | 192.168.40.66 | /28 | 192.168.40.65 | DHCP + DNS (helper-address target) |
| `SRV-FILE` | 30 | 192.168.40.67 | /28 | 192.168.40.65 | Intranet / client documents (HTTP) |
| `SRV-BACKUP` | 30 | 192.168.40.68 | /28 | 192.168.40.65 | FTP / backup target |
| `BOARD-DISPLAY` | 40 | 192.168.40.82 | /28 | 192.168.40.81 | Wall display / presentation host |
| `SW-CORE` (SVI 99) | 99 | 192.168.40.194 | /28 | 192.168.40.193 | STP root, management |
| `SW-A1` (SVI 99) | 99 | 192.168.40.195 | /28 | 192.168.40.193 | |
| `SW-A2` (SVI 99) | 99 | 192.168.40.196 | /28 | 192.168.40.193 | |
| `SW-A3` (SVI 99) | 99 | 192.168.40.197 | /28 | 192.168.40.193 | Boardroom access switch |
| `WLC-1` management | 99 | 192.168.40.198 | /28 | 192.168.40.193 | AP join / CAPWAP endpoint |
| `AP1` | 99 | 192.168.40.199 | /28 | 192.168.40.193 | DHCP-reserved or static |
| `AP2` | 99 | 192.168.40.200 | /28 | 192.168.40.193 | |
| `AP3` | 99 | 192.168.40.201 | /28 | 192.168.40.193 | Boardroom AP |
| `ADMIN-PC` | 99 | 192.168.40.202 | /28 | 192.168.40.193 | Management workstation |

## 4.5 DHCP scopes (on `SRV-DHCP-DNS`, relayed by `ip helper-address 192.168.40.66`)

| Pool name | VLAN | Pool range | Mask | Gateway | DNS | Excluded |
|---|---|---|---|---|---|---|
| `POOL-BROKERS` | 10 | 192.168.40.10 – .29 | 255.255.255.224 | 192.168.40.1 | 192.168.40.66 | .1 – .9, .30 |
| `POOL-ADMIN-FIN` | 20 | 192.168.40.42 – .61 | 255.255.255.224 | 192.168.40.33 | 192.168.40.66 | .33 – .41, .62 |
| `POOL-BOARDROOM` | 40 | 192.168.40.84 – .93 | 255.255.255.240 | 192.168.40.81 | 192.168.40.66 | .81 – .83, .94 |
| `POOL-WLAN-STAFF` | 50 | 192.168.40.100 – .125 | 255.255.255.224 | 192.168.40.97 | 192.168.40.66 | .97 – .99, .126 |
| `POOL-WLAN-GUEST` | 60 | 192.168.40.135 – .189 | 255.255.255.192 | 192.168.40.129 | 192.168.40.66 | .129 – .134, .190 |

VLAN 30 (servers) and VLAN 99 (management) are **statically addressed only** — no DHCP scope.

Lease times: staff and wired VLANs 8 days; boardroom 4 hours; guest 2 hours (keeps the /26 from filling up
with stale leases from walk-in clients).

## 4.6 NAT / PAT plan

| Item | Configuration intent |
|---|---|
| Inside interfaces | All `Gi0/0.x` sub-interfaces (`ip nat inside`) |
| Outside interfaces | `Gi0/1` (ISP-A) and `Gi0/2` (ISP-B), both `ip nat outside` |
| Matching ACL | `access-list 10 permit 192.168.40.0 0.0.0.255` (whole assigned block) |
| Translation | Two route-map-based overload statements, one per outside interface, so translation follows whichever default route is active |
| Effect | Every internal VLAN reaches the Internet from the active provider's address; failover does not require a NAT reconfiguration |

## 4.7 Verification of the plan

- No subnet overlaps: each network/broadcast boundary in §4.2 is contiguous and non-overlapping from
  192.168.40.0 to 192.168.40.255.
- Every subnet's usable count ≥ required usable count from §4.1.
- Every user VLAN has a gateway that is the first usable address — a consistent, documentable rule.
- Machine-readable versions of these tables: [`../ip-plan/ip-addressing-plan.csv`](../ip-plan/ip-addressing-plan.csv)
  and [`../ip-plan/device-addressing.csv`](../ip-plan/device-addressing.csv).
