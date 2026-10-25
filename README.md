## TNARC Financial: Enterprise Network

The complete network for TNARC Financial (Totally Not A Real Company), built in Cisco Packet Tracer 8.2.2 one lab at a time. Each stage adds a capability, each lab explains that stage in depth.

<img src="./images/TNARC_Enterprise_Network.JPG" alt="TNARC Topology" Width="800">

## Current capabilities
- Redundant three-tier HQ campus: two building and a server room joined by a routed core.
- HSRP gateways and Rapid PVST+, with STP root aligned to HSRP active per VLAN.
- Multi-area OSPF (one area per building) and a central DHCP via relay.
- Access layer security baseline: DHCP snooping, Dynamic ARP inspection, port security and PortFast with BPDU guard on user ports.
- Secured device management: SSH v2 only, central TACACS+ authentication with a local fallback, management access limited to VLAN 99/199, idle timeouts, hashed passwords and login banners.

## Build stages
| Stage | Lab | What it added | Tag |
|-------|-----|---------------|-----|
| 1 | [Lab 01: Three-Tier Campus](https://github.com/gusRod36/TNARC_HQ_Lab_01) | Campus, HRSP, OSPF, Central DHCP | v1-campus |
| 2 | [Lab 02: Access-Layer Security](https://github.com/gusRod36/TNARC_Access_Hardening) | SSH and AAA, DHCP Snooping, DAI, Port-security, BPDU Guard  | v2-accessHardening |
| 3 | [Lab 03: Network Management and DNS](https://github.com/gusRod36/TNARC_Network-Management_Lab03) | In progress... | v3-network_management | 

## Design
- [Design Doc](./docs/Design.md)
- [Network Addresses](./docs/Network_Addresses.md)

## Limitations
1. "login block-for" isn't supported in Cisco Packet Tracer, repeated login attempts are slowed but not locked out.
2. Local emergency account is shared.
3. No central logging or time sync yet, so AAA and security events stay on each device.

*Built by Gustavo Rodriguez - [LinkedIn](https://www.linkedin.com/in/gustavo-rodriguez-200aab423/)*