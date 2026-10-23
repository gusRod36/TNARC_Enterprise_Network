## TNARC Financial: Enterprise Network

The complete network for TNARC Financial (Totally Not A Real Company), built in Cisco Packet Tracer 8.2.2 one lab at a time. Each stage adds a capability, each lab explains that stage in depth.

<img src="./images/TNARC_HQ_Topology_final.JPG" alt="TNARC Topology" Width="800">

## Current capabilities
- Redundant three-tier HQ campus: two building and a server room joined by a routed core.
- HSRP gateways and Rapid PVST+, with STP root aligned to HSRP active per VLAN.
- Multi-area OSPF (one area per building) and a central DHCP via relay.

## Build stages
| Stage | Lab | What it added | Tag |
|-------|-----|---------------|-----|
| 1 | [Lab 01: Three-Tier Campus](https://github.com/gusRod36/TNARC_HQ_Lab_01) | Campus, HRSP, OSPF, Central DHCP | v1-campus |
| 2 | [Lab 02: Access-Layer Security](https://github.com/gusRod36/TNARC_Access_Hardening) | In progress | |

## Design
- 