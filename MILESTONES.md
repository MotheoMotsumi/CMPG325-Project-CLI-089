# Milestone progress log - CMPG325-2026-089 (CLI-089)

| Milestone | Date | Status | Summary |
|---|---|---|---|
| Commencement | 14 Aug 2026 | Complete | Brief allocated, client analysed |
| Milestone 1 - Client design review | 28 Aug 2026 | Submitted | Client requirements, physical topology, logical topology, IP addressing plan, initial GitHub repository |
| Milestone 2 - Client implementation review | 2 Oct 2026 | Complete | Packet Tracer build, WLAN challenge configured and verified, CR15 integrated, testing evidence documented |
| Final submission | 16 Oct 2026 | Not started | .pkt file, full portfolio, 15-20 min video demonstration |

## Milestone 2 - what was delivered

- **Packet Tracer file:** `packet-tracer/41961366_CMPG325_M2.pkt` - three-tier network with VLANs 10, 20, 30 and 99, router-on-a-stick inter-VLAN routing, DHCP, dual-ISP NAT.
- **Assigned feature (Wireless LAN):** AP1-BOARD in the boardroom VLAN, WPA2-PSK, three wireless clients, wrong-passphrase rejection tested.
- **CR15:** second edge router and ISP with floating static default routes; failover and restore tested.
- **Boardroom isolation:** ACL `BOARD-IN` on R3-CORE Gi0/0.30.
- **Testing evidence:** tests T01 to T16, screenshots in `evidence/screenshots/`, explained in `docs/Milestone2_Testing_Evidence_41961366.docx`.
- **Written review:** `docs/Milestone2_Client_Implementation_Review_41961366.docx`, including the differences from the Milestone 1 diagrams.
- **Issues and resolutions:** `evidence/troubleshooting/issues-and-resolutions.md`.

## Next - final submission (16 Oct 2026)

- Final `.pkt` file and full portfolio.
- 15-20 minute video demonstration.
