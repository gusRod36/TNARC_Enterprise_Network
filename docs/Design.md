# Sites
HQ: Building A, Building B, Server room, 
Branch 1 (new),
Branch 2 (acquired).


## Design principles
- Redundancy above access later: No single distribution or core failure disconnects users.
- Layer 2 stays inside each building, everything between buildings is routed.
- Security baseline applied before any new site goes live.
- Every device gets the security baseline before it joins the network.
- Device management is reachable only from the Management VLANs.
- Shared services (logging, time, DNS, backups) are in place before any new site connects.
- Deny by default at the perimeter, every allowed flow is documented and tested.
- No edge or DMZ device starts connections into the inside network, except narrow management rules.

## Architecture

#### HQ

| Layer | Devices | Model | Role |
|-------|---------|-------|------|
| Core | Core-1, Core-2 | 3650 | Routed backbone joining every block |
| Distribution (Building A) | A-Distribution-1, A-Distribution-2 | 3650 | Gateways for Building A VLANs |
| Distribution (Building B) | B-Distribution-1, B-Distribution-2 | 3650 | Gateways for Building B VLANs |
| Access | A-Access-SW1, A-Access-SW2, B-Access-SW1, B-Access-SW2 | 2960 | User ports |
| Server block | Server-SW1 | 3650 | Connects DHCP server and future servers |
| End devices | 2 PCs per VLAN, DHCP-Server | | Testing and Services |
| Edge | Edge-RTR, ASA-1 | 2911, ASA 5506-X | NAT/PAT, Security |

OSPF areas: 0 = Backbone - 1 = Building A - 2 = Building B - 3 = Server room

#### Branch 1

| Device | Model | Role |
|--------|-------|------|
| BR1-Edge-RTR | 2911 | Connects the Network to ISP1 (ISP2 in a real scenario as failover) |
| BR1-ASA-1 | 5506-X | Filtering and Policy management for the branch |
| BR1-Core-1 | 3650 | Routed ports for traffic from Dist 1 and 2, failover with Core-2 via HSRP |
| BR1-Core-2 | 3650 | Routed ports for traffic from Dist 2 and 1, failover with Core-1 via HSRP |
| BR1-Dist-1 | 3650 | STP: Root primary for VLANs 10,20,30, and 99. |
| BR1-Dist-2 | 3650 | STP: Root secondary for VLANs 10,20,30, and 99. Failover for Dist-1. |
| BR1-Access-SW1 | 2960 | Hosts VLANs 10 and 20, Port-security with maximum 1 MAC, sticky learning and shutdown violation |
| BR1-Access-SW2 | 2960 | Hosts VLANs 30 and 99, Port-security with maximum 1 MAC, sticky learning and shutdown violation |

#### Branch 2 

| Device | Model | Role |
|--------|-------|------|
| BR2-Edge-RTR | 2911 | Connects the Network to ISP2 (ISP1 in a real scenario as failover) |
| BR2-ASA-1 | 5506-X | Filtering and Policy management for the branch |
| BR2-Core-1 | 3650 | Routed ports for traffic from Dist 1 and 2, failover with Core-2 via HSRP |
| BR2-Core-2 | 3650 | Routed ports for traffic from Dist 2 and 1, failover with Core-1 via HSRP |
| BR2-Dist-1 | 3650 | STP: Root primary for VLANs 10,20,30, and 99. |
| BR2-Dist-2 | 3650 | STP: Root secondary for VLANs 10,20,30, and 99. Failover for Dist-1. |
| BR2-Access-SW1 | 2960 | Hosts VLANs 10 and 20, Port-security with maximum 1 MAC, sticky learning and shutdown violation |
| BR2-Access-SW2 | 2960 | Hosts VLANs 30 and 99, Port-security with maximum 1 MAC, sticky learning and shutdown violation |

## Network Addressing

#### HQ

| VLAN | Name | Building | Subnet | Gateway (Virtual IP) |
|------|------|----------|--------|----------------------|
| 10 | Finance-A | A | 10.0.10.0/24 | 10.0.10.1 |
| 20 | Engineering-A | A | 10.0.20.0/24 | 10.0.20.1 |
| 30 | HR-A | A | 10.0.30.0/24 | 10.0.30.1 |
| 99 | Management-A | A | 10.0.99.0/24 | 10.0.99.1 |
| 100 | Management_VLAN | A | 10.0.100.0/26 | 10.0.100.1 |
| 100 | Management_VLAN | B | 10.0.100.64/26 | 10.0.100.65
| 110 | Finance-B | B | 10.0.110.0/24 | 10.0.110.1 |
| 120 | Engineering-B | B | 10.0.120.0/24 | 10.0.120.1 |
| 130 | HR-B | B | 10.0.130.0/24 | 10.0.130.1 |
| 199 | Management-B | B | 10.0.199.0/24 | 10.0.199.1 |
| 210 | DMZ | DMZ | 10.0.210.0/24 | 10.0.210.1 |
|  | DHCP | Server Block | 10.0.200.0/28 | 10.0.200.1 |
|  | AAA | Server Block | 10.0.200.16/28 | 10.0.200.17 |
|  | Management-Server | Server Block | 10.0.200.32/28 | 10.0.200.33 |
|  | PAT | Edge | 203.0.113.0/28 | | 
|  | Outside/ISP | Edge | 198.51.100.0/30, 198.51.100.4/30 | 198.51.100.1, 198.51.100.5 |
|  | ASA-1 to Edge-RTR | Edge | 10.0.250.0/30 | 10.0.250.1 | 
| 777 | Native | A and B | | |
| 999 | Unused | A and B | | |

#### Branches 

| Block | Use | Notes |
|-------|-----|-------|
| 203.0.113.15/32 | HQ-Edge-RTR Loopback1 - Tunnel endpoint | Spare address from TNARC's block |
| 198.51.100.8/30 | ISP1 -> ISP2 | Connection between ISPs for Failover, wishful thinking as it is not currently possible with this router |
| 198.51.100.12/30 | BR1-Edge-RTR -> ISP1 | Public address comes from ISP1 |
| 198.51.100.16/30 | BR2-Edge-RTR -> ISP2 | Public address comes from ISP2 |
| 10.1.253.0/30 | Tunnel1 - HQ-Edge-RTR -> BR1-Edge-RTR | Tunnel address | 
| 172.16.253.0/30 | Tunnel2 - HQ-Edge-RTR -> BR2-Edge-RTR | Tunnel address |
| 10.1.0.0/16 | Branch 1 LANs | Follows the same pattern as HQ for VLANs. |
| 172.16.0.0/16 | Branch 2 LANs (acquired) | Chose this address to spot the EIGRP standout |
| 192.0.2.0/24 | ISP1 LAN | The test network behind ISP1 |
| 192.18.0.0/24 | ISP2 LAN | The test network behind ISP2 |

[Click for more..](./Network_Addresses.md)

## Network-wide decisions
| Decision | Why | Stage |
|----------|-----|-------|
| Routed links between distribution and core | No STP between layers, ECMP uplinks, faster failover | 1 |
| One OSPF area per building | Contains change and failure, room to expand the network | 1 |
| DHCP snooping information option (option 82) disabled on access switches | Server doesn't use it, relay would drop requests | 2 |
| TACACS+ for device administration | Full message encryption, separate authorization and accounting | 2 |
| Local emergency account as AAA fallback | Admins keep access if the AAA server is unreachable | 2 |
| DHCP information option disabled | Server doesn't use it and the distribution relay drops the packet | 2 |
| Centralized NTP server | Keeps all devices synchronized for logging | 3 |
| Syslog trap level debugging | Capture all events, logs and alerts across the network | 3 |
| Events correlated by Syslog server receive time plus sequence numbers | Server gives one timeline | 3 |
| DNS records point to loopbacks | Loopbacks stay reachable when a single link fails | 3 |
| One shared server for Syslog, TFTP, DNS, NTP | Keeps it simple, although it risks all service going down at once | 3 |
| Bootstrap config kept for every device type | A wiped device can't reach TFTP without basic reachability | 3 |
| NAT on Edge-RTR instead of ASA | CPT's ASA only supports PAT to its local interface address, which is a private adress. Edge-RTR handles a PAT pool and static NAT addresses so the ASA only sees private addresses | 4 |
| TNARC owns public address block 203.0.113.0/28, advertised to both ISPs over BGP | Internal Users are translated to TNARC's own addresses, so replies return through either ISP and failover works. | 4 |
| Public block split: PAT pool = 203.0.113.1-10, NAT static 203.0.113.11 DNS-Server, NAT Static 203.0.113.12 Web-Server | The PAT pool can never take a server's public IP address, spare addresses leave room for new services | 4 |
| Edge-RTR reaches AAA, Syslog, NTP and TFTP through host to host only rules | Edge-RTR sits outside the firewall but still needs central login, logging and backups. Each rule allows one source, one destination and one port, nothing else not even ICMP | 4 |
| ASA uses individual local accounts | CPT's ASA doesn't support AAA-server , so no centralized authentication was possible. | 4 |
| DMZ-SW uses individual local accounts, not TACACS+ | A DMZ to AAA-server would let a compromized DMZ device guess credentials or attack the server that controls access to the whole network. Local accounts keep the DMZ fully isolated while logins stay individual | 4 |
| Branches connect over the internet with GRE over IPsec | Cheaper and faster than leased lines; data is always encrypted | 5 |
| Hub and Spoke through HQ-Edge-RTR | One tunnel per branch and one redistribution point, a full mesh grows with every branch. | 5 |
| Router for edge, ASA policy at every site | Consistency and each device serves one purpose. | 5 |
| Direct internet access at branches | Backhauling would use HQ bandwidth without passing the HQ ASA | 5 |
| Fail closed with NO-LEAKS ACL | Company traffic never leaves unencrypted if a tunnel goes down | 5 |
| Branches outside at HQ, One rule per service | Least privilege for every site | 5 |
| Predictable paths through stateful devices | Equal-cost paths broke the ASA and DHCP relay | 5 |
| Layer 2 stops at distribution at every site | Keeps broadcasts out of the core | 5 |

## Security baseline (all access switches)
| Area | Standard |
|------|----------|
| User ports | Port security (sticky, 1 max, shutdown), PortFast + BPDU Guard |
| User ports | DHCP snooping and DAI untrusted |
| Unused ports | Shut down, VLAN 999 |
| Uplinks | DHCP snooping and DAI trusted |

## Management baseline
| Area | Standard |
|------|----------|
| Access | SSH v2 only, VTY ACL VLAN 99/199 hosts only |
| Authentication | TACACS+ via AAA-Server, local emergency fallback account |
| Sessions | exec-timeout 5 minutes, SSH retries 3, SSH timeout 60 seconds |
| Passwords | enable secret, service password-encryption |
| Banner | Restricted access warning on every login |
| Logging | logging host 10.0.200.35, trap level debugging, sequence-numbers, service timestamps log datetime msec | 
| Time | ntp server 10.0.200.35 | 
| Backups | Running-config copied to TFTP after every change |
| DNS | FQDN for queries on hosts, DNS server IP handed out by DHCP to clients |

## WAN and Site-to-site VPN

1. Transport: public internet with GRE over IPsec, instead of a private WAN. GRE carries the routing procotols' multicast and IPsec encrypts the GRE.
2. Hub and Spoke topology, HQ-Edge-RTR is the hub, each branch tunnel uses the HQ link on the branch's own ISP.
3. Crypto is set up as: IKEv1, AES-256, SHA, DH group 5, a unique pre-shared key per branch, transform set esp-aes 256 esp-sha-hmac
4. Tunnels end on the physical ISP interfaces (no local-address in CPT)
5. Fail closed ACL "NO-LEAKS" outbound on every ISP interface denies unecrypted company traffic.

## Security Policy

1. Every site follows one pattern: a router for ISP, NAT and VPN, and an ASA for policy.
2. At HQ, branches are treated as outside with a rule per service. Everything else is denied.
3. Company traffic is never translated; NAT ACLs deny company destinations.
4. ASA rules use named objects, except UDP rules. UDP rules use plain subnets.
5. Full list of policies [here.](./ASA_ACLs.md)

*Built by Gustavo Rodriguez - [LinkedIn](https://www.linkedin.com/in/gustavo-rodriguez-200aab423/)*