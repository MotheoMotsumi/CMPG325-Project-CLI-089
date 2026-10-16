# Device Configuration Reference (as built)

Kamogelo Insurance Brokers | CMPG325-2026-089 | Client CLI-089

This is the configuration applied in Packet Tracer, matching the Milestone 1
addressing plan. It replaces the earlier draft, which assumed router-on-a-stick.

## Differences from the earlier draft

- **R3-CORE uses SVIs, not router-on-a-stick.** The NIM-ES2-4 module on the 4331
  provides switch ports. Testing `no switchport` on Gi0/1/0 returned
  "% Incomplete command", so the port cannot be routed. R3-CORE therefore
  trunks to SW1-CORE over a module port (Gi0/2/0) and routes between VLANs
  through VLAN interfaces. The Milestone 1 logical diagram already allowed
  this ("router-on-a-stick OR SVIs"), so the design is unchanged.
- **R1 and R2 have a return route** to 192.168.40.0/24 via R3-CORE.
- **SW1-CORE** is a 3560-24PS. Uplinks to the access switches use Fa0/1 to Fa0/4.
- All configuration is entered from `Switch#`/`Router#` then `configure terminal`.

## R1-EDGE-PRI (Cisco 4331)

```
enable
configure terminal
no ip domain-lookup
hostname R1-EDGE-PRI
interface GigabitEthernet0/0/0
 description WAN link to ISP-A
 ip address 203.0.113.1 255.255.255.252
 no shutdown
interface GigabitEthernet0/0/1
 description Link to R3-CORE
 ip address 192.168.40.73 255.255.255.252
 no shutdown
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0/0
ip route 192.168.40.0 255.255.255.0 192.168.40.74
end
write memory
```

203.0.113.0/24 is a documentation range used as a placeholder for the ISP side.
Gi0/0/0 stays down because nothing is cabled to it.

## R2-EDGE-BAK (Cisco 4331)

```
enable
configure terminal
no ip domain-lookup
hostname R2-EDGE-BAK
interface GigabitEthernet0/0/0
 description WAN link to ISP-B (CR15 backup)
 ip address 203.0.113.5 255.255.255.252
 no shutdown
interface GigabitEthernet0/0/1
 description Link to R3-CORE
 ip address 192.168.40.77 255.255.255.252
 no shutdown
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0/0
ip route 192.168.40.0 255.255.255.0 192.168.40.78
end
write memory
```

## R3-CORE (Cisco 4331 with NIM-ES2-4) - inter-VLAN routing, DHCP, CR15

Cabled: Gi0/0/0 to R1 Gi0/0/1, Gi0/0/1 to R2 Gi0/0/1 (cross-over), module port
Gi0/2/0 to SW1-CORE.

```
enable
configure terminal
no ip domain-lookup
hostname R3-CORE
vlan 10
 name SALES
vlan 20
 name ADMIN
vlan 30
 name BOARDROOM
vlan 99
 name SERVER-INFRA
interface GigabitEthernet0/2/0
 description Trunk to SW1-CORE
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
 no shutdown
interface Vlan10
 ip address 192.168.40.1 255.255.255.224
 no shutdown
interface Vlan20
 ip address 192.168.40.33 255.255.255.240
 no shutdown
interface Vlan30
 ip address 192.168.40.49 255.255.255.240
 no shutdown
interface Vlan99
 ip address 192.168.40.65 255.255.255.248
 no shutdown
interface GigabitEthernet0/0/0
 description Link to R1-EDGE-PRI (ISP-A path)
 ip address 192.168.40.74 255.255.255.252
 no shutdown
interface GigabitEthernet0/0/1
 description Link to R2-EDGE-BAK (ISP-B path, CR15)
 ip address 192.168.40.78 255.255.255.252
 no shutdown
ip route 0.0.0.0 0.0.0.0 192.168.40.73
ip route 0.0.0.0 0.0.0.0 192.168.40.77 5
ip dhcp excluded-address 192.168.40.1 192.168.40.5
ip dhcp excluded-address 192.168.40.33 192.168.40.37
ip dhcp excluded-address 192.168.40.49 192.168.40.53
ip dhcp pool VLAN10-SALES
 network 192.168.40.0 255.255.255.224
 default-router 192.168.40.1
 dns-server 8.8.8.8
ip dhcp pool VLAN20-ADMIN
 network 192.168.40.32 255.255.255.240
 default-router 192.168.40.33
 dns-server 8.8.8.8
ip dhcp pool VLAN30-BOARD
 network 192.168.40.48 255.255.255.240
 default-router 192.168.40.49
 dns-server 8.8.8.8
end
write memory
```

CR15: the route via 192.168.40.73 (ISP-A) is active. The route via
192.168.40.77 has administrative distance 5, so it only enters the routing
table when the primary fails. VLAN 99 has no DHCP pool by design; FILE-SRV is
static.

## SW1-CORE (Cisco 3560-24PS)

Layer 2 distribution only. The trunk to R3-CORE is assumed to be Gi0/1. Confirm
with `show cdp neighbors` and change the port name if your cable differs.

```
enable
configure terminal
no ip domain-lookup
hostname SW1-CORE
vlan 10
 name SALES
vlan 20
 name ADMIN
vlan 30
 name BOARDROOM
vlan 99
 name SERVER-INFRA
interface GigabitEthernet0/1
 description Trunk to R3-CORE
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
 no shutdown
interface range FastEthernet0/1 - 4
 description Trunks to access switches
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
 no shutdown
end
write memory
```

## Access switches (Cisco 2960)

Uplink is Fa0/1 on each. Ports Fa0/2 to Fa0/24 are access ports in the
switch's VLAN.

| Switch | Hostname | VLAN | VLAN name | Devices |
|---|---|---|---|---|
| SW2-SALES | SW2-SALES | 10 | SALES | PC-S1, PC-S2, PC-S3 (Fa0/2-4) |
| SW3-ADMIN | SW3-ADMIN | 20 | ADMIN | PC-A1, PC-A2, Laptop-Mgr (Fa0/2-4) |
| SW4-BOARD | SW4-BOARD | 30 | BOARDROOM | Presenter-PC (Fa0/2), AP1-BOARD (Fa0/3) |
| SW5-SRV | SW5-SRV | 99 | SERVER-INFRA | FILE-SRV (Fa0/2) |

Template (replace `<HOST>`, `<ID>` and `<NAME>` from the table):

```
enable
configure terminal
no ip domain-lookup
hostname <HOST>
vlan <ID>
 name <NAME>
exit
interface range FastEthernet0/2 - 24
 switchport mode access
 switchport access vlan <ID>
exit
interface FastEthernet0/1
 description Uplink to SW1-CORE
 switchport mode trunk
 switchport trunk allowed vlan <ID>
end
write memory
```

SW4-BOARD satisfies the boardroom constraint: the wired presenter port
(Fa0/2) and the AP uplink (Fa0/3) are separate ports, both in VLAN 30.

## AP1-BOARD (Access Point-PT) - Wireless LAN challenge

Configured in the Config tab (no CLI). Cabled to SW4-BOARD Fa0/3.

- Config > Port 1: SSID `KIB-BOARD`, authentication WPA2-PSK, passphrase set
  and recorded by the student.
- The AP bridges onto VLAN 30. Clients receive 192.168.40.48/28 addresses from
  R3-CORE's VLAN30-BOARD pool.
- Tablet-W1: Config > Wireless0, same SSID, WPA2-PSK and passphrase, DHCP.
- Laptop-W1: wired module replaced with the wireless module (PT-LAPTOP-NM-1W),
  then Desktop > PC Wireless > Connect to `KIB-BOARD`, DHCP.

## End devices

- PC-S1/S2/S3, PC-A1/A2, Laptop-Mgr, Presenter-PC, Tablet-W1, Laptop-W1: DHCP.
- FILE-SRV: static 192.168.40.66, mask 255.255.255.248, gateway 192.168.40.65.
