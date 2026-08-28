# 5. Design Decisions and Rationale

**Project:** CMPG325-2026-089 · Kamogelo Insurance Brokers (Vryburg)

Each decision below records the option chosen, the alternatives considered, and why the choice suits this
client. These are the points I expect to be questioned on at the design review and in the video demonstration.

---

### DD-01 — Router-on-a-stick instead of a Layer 3 switch

**Chosen:** 802.1Q sub-interfaces on `R1-EDGE Gi0/0` for all seven VLANs.
**Alternatives:** SVIs with `ip routing` on the 3560 core; separate physical router interfaces per VLAN.
**Why:** Traffic volumes for ~65 devices are small, the router is already required for dual-ISP NAT, and
keeping routing and policy on one device means one place to configure ACLs, DHCP relay and NAT — important for
a client with no on-site network administrator. A 2911 has only three GigE ports, two of which are consumed by
the two Internet links (CR15), so one trunked LAN interface is also the only physically viable option.
**Trade-off:** All inter-VLAN traffic crosses one link ("router-on-a-stick" bottleneck) and the router is a
single point of failure for routing. Accepted for the size and budget; noted as the first thing to upgrade
(move routing to the 3560) if KIB grows.

### DD-02 — Seven VLANs, aligned to business function

**Chosen:** VLANs 10/20/30/40/50/60/99 plus a parking VLAN 999.
**Alternatives:** One flat network; two VLANs (staff/guest).
**Why:** A flat /24 would put client-data traffic, printers and walk-in visitors in one broadcast domain, which
fails the confidentiality driver and the acceptance criterion AC-02. Function-based VLANs give a clean place to
apply policy (guest denied to internal, boardroom limited to what a presenter needs) and make the addressing
plan self-documenting.
**Trade-off:** More configuration and a slightly more complex fault domain; mitigated by consistent naming.

### DD-03 — WLC-managed lightweight APs instead of standalone APs

**Chosen:** WLC 2504 with three lightweight APs.
**Alternatives:** Three autonomous APs configured individually.
**Why:** The assigned challenge is *AP integration & coverage*. A controller lets three SSIDs be mapped to
three different VLANs from a single configuration point, gives consistent security settings, supports client
roaming between AP1 and AP2, and lets channel/power be managed centrally. Standalone Packet Tracer APs support
effectively one SSID each, which cannot deliver staff + guest + boardroom separation.
**Trade-off:** The WLC is a single point of wireless management failure; APs continue serving associated
clients but new WLAN changes need the controller.

### DD-04 — Dedicated boardroom VLAN with both wireless and wired paths

**Chosen:** VLAN 40 carries both the `KIB-BOARD` SSID and five wired presentation ports.
**Alternatives:** Put the boardroom on the staff VLAN; wireless-only presentation.
**Why:** This directly implements the client's design constraint. One VLAN for both media means a presenter
gets the same addressing and the same permissions whether they plug in or associate wirelessly, and the wired
ports are a guaranteed fallback when a visitor's laptop will not join Wi-Fi — which is exactly the failure the
constraint exists to prevent.
**Trade-off:** A visitor on the boardroom SSID is more trusted than a guest; mitigated by limiting VLAN 40 to
HTTP/file access to the server VLAN only.

### DD-05 — Guest wireless isolated at Layer 3, not just at Layer 2

**Chosen:** VLAN 60 with an inbound extended ACL on `Gi0/0.60` denying 192.168.40.0/24 internal destinations
(except DHCP/DNS to the server) and permitting the rest.
**Alternatives:** Trust VLAN separation alone; use a separate guest Internet link.
**Why:** VLAN separation prevents Layer 2 snooping but not routed access — without an ACL, a guest can still
reach the file server by IP. Blocking at the router satisfies AC-02 and is verifiable in the demonstration
(ping to server fails, ping to Internet succeeds).

### DD-06 — Floating static default routes for CR15 rather than a routing protocol

**Chosen:** `ip route 0.0.0.0 0.0.0.0 <ISP-A> ` and `ip route 0.0.0.0 0.0.0.0 <ISP-B> 5`.
**Alternatives:** eBGP with both providers; per-flow load balancing across both links.
**Why:** KIB has no public AS or provider-independent address space, so BGP is not available or affordable.
Floating statics give automatic failover with two commands and are fully demonstrable in Packet Tracer by
shutting the primary interface. Load balancing was rejected because the secondary link is an LTE service with
different performance, and session-breaking asymmetry would hurt insurer portal use.
**Trade-off:** Failover reacts to interface/line-protocol failure, not to "link up but ISP broken"; noted in
the change-request document with IP SLA tracking as the real-world improvement.

### DD-07 — Central DHCP server with helper addresses instead of router pools

**Chosen:** Scopes on `SRV-DHCP-DNS` (192.168.40.66), `ip helper-address` on each user sub-interface.
**Alternatives:** `ip dhcp pool` on `R1-EDGE`.
**Why:** Keeps address management in one place alongside DNS, gives the client a familiar single console for
reservations, and keeps router configuration lean. It also demonstrates DHCP relay, which router-based pools
would not.
**Trade-off:** Loss of the server stops new leases; mitigated by long leases on wired VLANs and static
addressing for all infrastructure.

### DD-08 — Static addressing for infrastructure, printers and servers

**Chosen:** Statics for router, switches, WLC, APs, servers and MFPs; DHCP for everything else.
**Why:** Management and printing must not depend on lease behaviour, and documented statics make
troubleshooting deterministic during the review and video demonstration.

### DD-09 — VLSM with a deliberate reserve

**Chosen:** Guest /26, three user /27s, three /28s, 48 addresses reserved.
**Alternatives:** Eight equal /27s (fits exactly 8 subnets, no reserve); flat /24.
**Why:** Equal /27s would waste 30 addresses each on the server and management segments while leaving the guest
segment short at busy times. Sizing to demand plus a reserve satisfies TR-01 and the client's growth driver
without renumbering later.

### DD-10 — Security hardening chosen for a small, unattended site

**Chosen:** Native VLAN moved off VLAN 1, unused ports shut into VLAN 999, PortFast + BPDU Guard on access
ports, WPA2 on staff/boardroom WLANs, SSH-only management from VLAN 99, port security on finance and boardroom
ports.
**Why:** These are configuration-only controls with no extra cost, and each maps to a realistic risk at a
public-facing office where visitors sit metres from live network outlets.
