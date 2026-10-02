# 06 - Wireless LAN design (assigned networking challenge)

**Challenge:** Wireless LAN - AP integration and coverage (Foundational).

## 1. Design

One standalone access point, AP1-BOARD, serves the boardroom. It connects to SW4-BOARD Fa0/2 (an access port in VLAN 30), so wireless clients share the VLAN, gateway and DHCP pool of the wired presentation port (SW4-BOARD Fa0/1). This provides dedicated wired and wireless presentation connections in one isolated segment.

## 2. Access point configuration

| Setting | Value |
|---|---|
| Device | AP1-BOARD (Access Point-PT) |
| Wired uplink | Port 0 to SW4-BOARD Fa0/2 (access, VLAN 30) |
| SSID | KAMOGELO-BOARD |
| Authentication / encryption | WPA2-PSK / AES |
| 2.4 GHz channel | 1 |
| Passphrase | Set on the AP and on every client; not reproduced in this repository |

![AP1-BOARD configuration](../evidence/screenshots/T04a_AP1-BOARD_wireless_config.png)

## 3. Clients

| Client | Address (DHCP, pool BOARD) | Gateway |
|---|---|---|
| Board-Tablet | 192.168.40.51/28 | 192.168.40.49 |
| Board-Phone | 192.168.40.52/28 | 192.168.40.49 |
| Board-Laptop | 192.168.40.53/28 | 192.168.40.49 |
| Board-Presenter-PC (wired) | 192.168.40.50/28 (static) | 192.168.40.49 |

## 4. Verification

| Check | Result | Evidence |
|---|---|---|
| Clients associate and obtain DHCP leases | Pass | T03, T05, T16 |
| Wireless laptop to wired presenter PC (TTL 128, same VLAN, not routed) | Pass | T06 |
| Wireless laptop to File-Server (routed by R3-CORE) | Pass | T07 |
| Wireless laptop to 8.8.8.8 through the primary ISP | Pass | T08 |
| Wrong passphrase: client fails to associate (0.0.0.0), then reconnects with the correct one | Pass | T04b |
| Boardroom isolation from Sales and Admin | Pass | T09 |

## 5. Coverage

The three wireless clients are placed in the boardroom area of the Packet Tracer workspace and each holds an active wireless link to AP1-BOARD (T16). Packet Tracer's coverage range is a simulation parameter and is not an RF survey; a single room needs a single AP.

## 6. Limitations

- One standalone AP and no wireless LAN controller, which suits a single room.
- A site survey, roaming and multiple APs are outside the scope of this Foundational task.
