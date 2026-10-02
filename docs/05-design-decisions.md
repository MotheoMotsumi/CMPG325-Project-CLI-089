# 05 - Design decisions

| # | Decision | Alternatives considered | Reason |
|---|---|---|---|
| 1 | Three-tier layout (edge, core, access) | A flat or two-tier design | Separates the WAN edge from internal routing and keeps the design simple enough for the Foundational challenge |
| 2 | Two independent edge routers, one per ISP | One router with two WAN interfaces | A single router would stay a single point of failure; two routers address the resilience required by CR15 |
| 3 | One access switch per department | A single switch with VLAN-tagged ports | Each VLAN stays on its own hardware; the boardroom switch gives the wired and wireless presentation ports their own path |
| 4 | Four VLANs sized with VLSM | Equal-size subnets | An even split would waste addresses on small segments or starve Sales |
| 5 | Router-on-a-stick on R3-CORE | SVIs on SW1-CORE | Matches the design in which R3-CORE performs all inter-VLAN routing and is the convergence point of both Internet paths |
| 6 | Standalone AP (AP1-BOARD) | Wireless LAN controller with lightweight APs | Practical and sufficient for a single room at Foundational level |
| 7 | AP and wired presenter port in the same VLAN (30) | Separate wireless VLAN | Satisfies the dedicated wired and wireless constraint in one isolated boardroom segment |
| 8 | WPA2-PSK with AES on the boardroom SSID | Open network | Protects boardroom traffic; tested with a wrong-passphrase attempt |
| 9 | DHCP server on R3-CORE | Separate DHCP server | R3-CORE is already the gateway of every VLAN, so no relay is needed |
| 10 | Static addresses for servers and the presenter PC | DHCP reservations | Fixed, predictable addresses for shared resources |
| 11 | NAT overload on each edge router | NAT on R3-CORE | Keeps translation at the ISP boundary of each path |
| 12 | Floating static default routes on R3-CORE (AD 1 and AD 10) | IP SLA route tracking | IP SLA was not available in the Packet Tracer version used; floating statics detect loss of the R3 to R1 link or R1 itself |
| 13 | ACL `BOARD-IN` on the boardroom gateway | Rely on VLAN separation alone | Routing would leave the boardroom reachable; the ACL enforces the isolation described in the design |
| 14 | Router-to-ISP links in documentation ranges (203.0.113.0/30, 198.51.100.0/30) | Addresses from the client block | Keeps the client block for the client's own segments |
| 15 | Loopback 8.8.8.8 on both ISP routers | A server behind the ISPs | Simple stand-in for an Internet host that works through either path |
