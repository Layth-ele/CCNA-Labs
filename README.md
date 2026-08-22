# CCNA Labs — Cisco Packet Tracer

Hands-on networking labs built and tested in Cisco Packet Tracer while studying
for the CCNA (200-301). Each folder has the lab's `.pkt` file and a README with
the topology, addressing, configuration, and verification steps.

## Labs

| # | Lab | Concepts |
|---|-----|----------|
| 01 | [Basic Routing](01-basic-routing/) | Directly connected networks, router interfaces, default gateways |
| 02 | [Inter-VLAN Routing](02-inter-vlan-routing/) | VLANs, access & trunk ports, router-on-a-stick, 802.1Q subinterfaces |
| 03 | [Static Routing](03-static-routing/) | Two routers, WAN /30 link, `ip route`, CDP troubleshooting |
| 04 | [OSPF (single-area)](04-ospf/) | Dynamic routing, OSPF neighbors, wildcard masks |
| 05 | [DHCP](05-dhcp/) | Router as DHCP server, address pools, excluded addresses, leases |
| 06 | [NAT / PAT](06-nat/) | NAT overload, inside/outside interfaces, ACL match, default route |
| 07 | [ACLs](07-acl/) | Standard & extended ACLs, permit/deny, filtering traffic on interfaces |
| 08 | [VLAN Trunking](08-vlan-trunking/) | 802.1Q trunks between switches, allowed VLANs, DTP / `nonegotiate` |
| 09 | [Spanning Tree](09-spanning-tree/) | STP, root bridge election, port roles/states, PortFast |
| 10 | [EtherChannel](10-etherchannel/) | Link aggregation, LACP/PAgP, port-channel bundles |
| 11 | [Port Security](11-port-security/) | MAC address limiting, sticky MACs, violation modes |
| 12 | [SSH / Device Hardening](12-ssh-hardening/) | SSH access, VTY lines, `enable secret`, banners, service hardening |
| 13 | [IPv6 Addressing & Static Routing](13-ipv6/) | IPv6 unicast-routing, `/64` addressing, IPv6 static routes |
| 14 | [OSPFv3](14-ospfv3/) | OSPF for IPv6, per-interface enablement, IPv4-format router-id |

## Roadmap

Planned: wireless (WLC/AP basics), and additional multi-area / redistribution
routing scenarios.

## Notes

`.pkt` files are binary, so this repo is a portfolio/archive — GitHub stores the
files but won't show line-by-line diffs. Open any `.pkt` in Cisco Packet Tracer
to explore or re-test the lab.
