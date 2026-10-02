# 08 - Testing plan and results

Evidence screenshots are in [`evidence/screenshots/`](../evidence/screenshots/) and are explained one by one in [`Milestone2_Testing_Evidence_41961366.docx`](Milestone2_Testing_Evidence_41961366.docx). A single lost packet on the first ping to a new destination is normal (ARP resolution).

| ID | Objective | Procedure | Expected result | Result |
|---|---|---|---|---|
| T01 | VLANs and access ports | `show vlan brief` on all five switches | VLANs 10, 20, 30, 99 present; access ports in the correct VLAN | Pass |
| T02 | Trunks | `show interfaces trunk` on all five switches | Gi0/1 trunking on access switches; Gi1/0/2 to 5 on SW1-CORE | Pass |
| T03 | DHCP | `show ip dhcp binding` on R3-CORE | Leases inside the Sales, Admin and Boardroom pools | Pass |
| T04a | AP security | AP1-BOARD Config, Port 1 | SSID KAMOGELO-BOARD, WPA2-PSK, AES, channel 1 | Pass |
| T04b | Wrong passphrase | Enter a wrong passphrase on Board-Tablet; `ipconfig`; restore the correct one | 0.0.0.0 while wrong; 192.168.40.51 after correction | Pass |
| T05 | Wireless addressing | `ipconfig` on the three wireless clients | 192.168.40.51 to .53, mask 255.255.255.240, gateway .49 | Pass |
| T06 | Wireless to wired presenter PC | Ping 192.168.40.50 from Board-Laptop | Reply, TTL 128 (same VLAN) | Pass |
| T07 | Wireless to server | Ping 192.168.40.66 from Board-Laptop | Reply, TTL 127 (routed by R3-CORE) | Pass |
| T08 | Wireless to Internet | Ping 8.8.8.8 from Board-Laptop | Reply, TTL 253 | Pass |
| T09 | Boardroom isolation ACL | Pings between the boardroom and Sales/Admin; `show access-lists` | Blocked to and from Sales/Admin; server and Internet still reachable | Pass |
| T10 | Inter-VLAN routing | Ping Admin, server and boardroom hosts from Sales-PC1 (before the ACL) | Replies, TTL 127 | Pass |
| T11 | Primary Internet path | `tracert 8.8.8.8` from Sales-PC1 | 192.168.40.1, 192.168.40.74, 8.8.8.8 | Pass |
| T12 | Failover route | Shut R1-EDGE-PRI Gi0/0; `show ip route` on R3-CORE | Default route [10/0] via 192.168.40.78 | Pass |
| T13 | Backup path traffic | `tracert 8.8.8.8` from Sales-PC1; pings from R2-EDGE-BAK | Path via 192.168.40.78; R2 reaches ISP-B and 8.8.8.8 | Pass |
| T14 | Restore | Restore R1 Gi0/0; `show ip route` on R3-CORE | Default route [1/0] via 192.168.40.74 | Pass |
| T15 | NAT | `show ip nat translations` on R1-EDGE-PRI | Translations to 203.0.113.2 | Pass |
| T16 | Wireless association | Topology view of AP1-BOARD | Laptop, phone and tablet each show an active wireless link | Pass |

Related notes: [issues found and resolved](../evidence/troubleshooting/issues-and-resolutions.md).
