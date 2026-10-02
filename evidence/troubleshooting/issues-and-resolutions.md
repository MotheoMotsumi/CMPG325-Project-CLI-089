# Issues found during the Milestone 2 build and how they were resolved

| # | Issue | Cause | Resolution | Verified by |
|---|---|---|---|---|
| 1 | IP SLA / route tracking commands were rejected (`show track`, `show ip sla statistics` returned `% Invalid input`), and R3-CORE showed "Gateway of last resort is not set" | The feature is not available in the Packet Tracer version used; only a /32 probe route to 8.8.8.8 existed | Replaced with two static default routes on R3-CORE: primary via 192.168.40.74 (AD 1) and floating backup via 192.168.40.78 (AD 10); removed the /32 probe route | T11 to T14 |
| 2 | Some early ping tests looked successful but proved nothing | Admin-PC1 pinging 192.168.40.34 and File-Server pinging 192.168.40.66 were pinging their own addresses | Re-ran the tests from Sales-PC1 to hosts in other VLANs; TTL 127 shows the traffic crossed R3-CORE | T10 |
| 3 | `no shutdown` on R1-EDGE-PRI returned `% Invalid input` | The command was typed in privileged EXEC mode instead of interface configuration mode | `configure terminal`, `interface gigabitEthernet 0/0`, `no shutdown` | T14, T15 |
| 4 | ISP-B could not ping 192.168.40.77 (0 of 5) | Expected behaviour: the simulated ISP has no route to the private block because R2-EDGE-BAK translates internal addresses with NAT | No fix needed; backup-path connectivity was tested from R2-EDGE-BAK and from the LAN instead | T13 |
| 5 | The first ping to a new destination loses one packet (3 of 4 replies) | ARP resolution on the first packet | No fix needed; documented in the test notes | T10, T11 |
| 6 | The wireless passphrase was visible in configuration screenshots | Screenshots of the AP and tablet configuration screens include the passphrase field | Passphrase changed on the AP and all clients; the screenshot used as evidence has the field masked | T04a |
