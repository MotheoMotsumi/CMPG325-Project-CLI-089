# 04 - IP addressing plan

The assigned block **192.168.40.0/24** is subnetted with VLSM so that each segment is sized to its host count.

## 1. Subnet allocation

| Segment | Network | Mask | Prefix | Usable range | Gateway | Broadcast |
|---|---|---|---|---|---|---|
| VLAN 10 Sales / Brokers | 192.168.40.0 | 255.255.255.224 | /27 | .1 - .30 (30) | .1 | .31 |
| VLAN 20 Admin and Management | 192.168.40.32 | 255.255.255.240 | /28 | .33 - .46 (14) | .33 | .47 |
| VLAN 30 Boardroom | 192.168.40.48 | 255.255.255.240 | /28 | .49 - .62 (14) | .49 | .63 |
| VLAN 99 Server / Infrastructure | 192.168.40.64 | 255.255.255.248 | /29 | .65 - .70 (6) | .65 | .71 |
| WAN Link 1 (R3-CORE to R1-EDGE-PRI) | 192.168.40.72 | 255.255.255.252 | /30 | .73 - .74 (2) | n/a | .75 |
| WAN Link 2 (R3-CORE to R2-EDGE-BAK) | 192.168.40.76 | 255.255.255.252 | /30 | .77 - .78 (2) | n/a | .79 |
| Unallocated | 192.168.40.80 onward | | | 176 addresses | | |

Router-to-ISP links sit outside the client block, in documentation ranges:

| Link | Network | Addresses |
|---|---|---|
| R1-EDGE-PRI to ISP-A | 203.0.113.0/30 | ISP-A .1, R1-EDGE-PRI .2 |
| R2-EDGE-BAK to ISP-B | 198.51.100.0/30 | ISP-B .1, R2-EDGE-BAK .2 |

## 2. Allocation reasoning

- Sales / Brokers was allocated first as the largest segment; a /27 gives 30 usable addresses.
- Admin and Management and the Boardroom each received a /28 (14 usable addresses), enough for current needs with room to grow.
- Server / Infrastructure needs only a /29 (6 usable addresses).
- Both WAN links use a /30, the most efficient size for a point-to-point link.
- The gateway is the first usable address in each subnet.

## 3. Host assignments

| Host | Address | Method |
|---|---|---|
| Sales PCs | From 192.168.40.2 | DHCP (pool SALES, .1 excluded) |
| Admin PCs | From 192.168.40.34 | DHCP (pool ADMIN, .33 excluded) |
| Board-Presenter-PC | 192.168.40.50 | Static |
| Boardroom wireless clients | 192.168.40.51 - .62 | DHCP (pool BOARD, .49 and .50 excluded) |
| File-Server | 192.168.40.66 | Static |
| Print-Server | 192.168.40.67 | Static |

All pools hand out the gateway of their own VLAN and DNS server 8.8.8.8.

## 4. Per-interface addressing

Router, switch, access-point and end-device addressing is listed in [`ip-plan/device-addressing.csv`](../ip-plan/device-addressing.csv). The subnet table is also available as [`ip-plan/ip-addressing-plan.csv`](../ip-plan/ip-addressing-plan.csv).
