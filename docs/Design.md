# Sites
HQ: Building A, Building B, Server room. Planned: Branch 1 (new), Branch 2 (acquired).


## Design principles
- Redundancy above access later: No single distribution or core failure disconnects users.
- Layer 2 stays inside each building, everything between buildings is routed.
- Security baseline applied before any new site goes live.
- Every device gets the security baseline before it joins the network.
- Device management is reachable only from the Management VLANs.

## Architecture

| Layer | Devices | Model | Role |
|-------|---------|-------|------|
| Core | Core-1, Core-2 | 3650 | Routed backbone joining every block |
| Distribution (Building A) | A-Distribution-1, A-Distribution-2 | 3650 | Gateways for Building A VLANs |
| Distribution (Building B) | B-Distribution-1, B-Distribution-2 | 3650 | Gateways for Building B VLANs |
| Access | A-Access-SW1, A-Access-SW2, B-Access-SW1, B-Access-SW2 | 2960 | User ports |
| Server block | Server-SW1 | 3650 | Connects DHCP server and future servers |
| End devices | 2 PCs per VLAN, DHCP-Server | | Testing and Services |

OSPF areas: 0 = Backbone - 1 = Building A - 2 = Building B - 3 = Server room

## Network Addressing
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
|  | DHCP | Server Block | 10.0.200.0/24 | 10.0.200.1 |
|  | AAA | Server Block | 10.0.201.0/24 | 10.0.201.1 |
| 777 | Native | A and B | | |
| 999 | Unused | A and B | | |

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


*Built by Gustavo Rodriguez - [LinkedIn](https://www.linkedin.com/in/gustavo-rodriguez-200aab423/)*