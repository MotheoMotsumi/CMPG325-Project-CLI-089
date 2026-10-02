# configs

One configuration file per network device, written from the commands applied in Packet Tracer (final state).
These are configuration references, not raw `show running-config` captures, so cosmetic lines such as
default interface settings, `version`, `service` and `spanning-tree` lines that IOS adds are not shown.

| File | Device |
|---|---|
| `R1-EDGE-PRI.txt` | Primary edge router (ISP-A) |
| `R2-EDGE-BAK.txt` | Backup edge router (ISP-B) |
| `R3-CORE.txt` | Core router: inter-VLAN routing, DHCP, default routes, ACL |
| `ISP-A.txt`, `ISP-B.txt` | Simulated ISPs |
| `SW1-CORE.txt` | Core switch |
| `SW2-SALES.txt`, `SW3-ADMIN.txt`, `SW4-BOARD.txt`, `SW5-SRV.txt` | Access switches |
| `AP1-BOARD_and_end-devices.txt` | Access point and end-device settings (GUI configured) |
