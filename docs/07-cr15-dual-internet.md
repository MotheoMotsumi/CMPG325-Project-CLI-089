# 07 - CR15: second Internet connection

**Change request:** a second Internet connection must be integrated for resilience.

## 1. Design

- R1-EDGE-PRI connects to ISP-A and R2-EDGE-BAK connects to ISP-B; each edge router has its own /30 link to R3-CORE.
- Each edge router translates the internal block with NAT overload on its ISP-facing interface.
- R3-CORE holds two default routes: the primary via R1 and a floating static backup via R2.

| Route on R3-CORE | Next hop | Role |
|---|---|---|
| 0.0.0.0/0 (AD 1) | 192.168.40.74 (R1-EDGE-PRI) | Primary path via ISP-A |
| 0.0.0.0/0 (AD 10) | 192.168.40.78 (R2-EDGE-BAK) | Backup path via ISP-B |

| Edge router | Inside | Outside | Return route to the LAN |
|---|---|---|---|
| R1-EDGE-PRI | Gi0/0 192.168.40.74/30 | Gi0/1 203.0.113.2/30 | 192.168.40.0/24 via 192.168.40.73 |
| R2-EDGE-BAK | Gi0/0 192.168.40.78/30 | Gi0/1 198.51.100.2/30 | 192.168.40.0/24 via 192.168.40.77 |

## 2. Failover test

| Step | Action | Observed result | Evidence |
|---|---|---|---|
| 1 | tracert 8.8.8.8 from Sales-PC1 | 192.168.40.1, 192.168.40.74, 8.8.8.8 | T11 |
| 2 | Shut down R1-EDGE-PRI Gi0/0 (primary path down) | R3-CORE default becomes [10/0] via 192.168.40.78 | T12 |
| 3 | tracert from Sales-PC1; ping from R2-EDGE-BAK | Path via 192.168.40.78; R2 reaches ISP-B and 8.8.8.8 | T13 |
| 4 | Restore R1-EDGE-PRI Gi0/0 | R3-CORE default returns to [1/0] via 192.168.40.74 | T14 |
| 5 | `show ip nat translations` on R1-EDGE-PRI | Translations to 203.0.113.2 | T15 |

## 3. Limitation

Floating static routes withdraw the primary route when the R3 to R1 link or R1 itself fails. They do not detect an outage further upstream, such as ISP-A failing while R1 stays up. IP SLA route tracking was attempted but was not available in the Packet Tracer version used (`show track` and `show ip sla` were rejected), so this limitation is accepted.
