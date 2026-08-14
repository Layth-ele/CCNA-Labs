# IPv6 Addressing & Static Routing (Cisco Packet Tracer)

Two routers, each with an IPv6 LAN, joined by an IPv6 WAN link. Configure IPv6
addresses, enable IPv6 routing, add static routes, and ping across using IPv6 —
the IPv6 version of the static routing lab.

## Topology

![IPv6 lab topology](topology.png)

```
   LAN A                                          LAN B
2001:DB8:A::/64                            2001:DB8:B::/64
     |                                            |
  +-------+   G0/0    WAN 2001:DB8:C::/64   G0/0  +-------+
  |Switch1|--[ R1 ]==========================[ R2 ]--|Switch2|
  +---+---+  ::1        ::1        ::2       ::1  +---+---+
    PC1                                            PC2
```

## Equipment

- 2 × Router (2911)
- 2 × Switch (2960)
- 2 × PC
- Copper straight-through cables

## Addressing table (IPv6)

| Device | Interface   | IPv6 Address       | Role            |
|--------|-------------|--------------------|-----------------|
| R1     | Gig0/0      | 2001:DB8:A::1/64   | LAN A gateway   |
| R1     | Gig0/1      | 2001:DB8:C::1/64   | WAN             |
| R2     | Gig0/0      | 2001:DB8:C::2/64   | WAN             |
| R2     | Gig0/1      | 2001:DB8:B::1/64   | LAN B gateway   |
| PC1    | —           | 2001:DB8:A::10/64  | GW 2001:DB8:A::1 |
| PC2    | —           | 2001:DB8:B::10/64  | GW 2001:DB8:B::1 |

## IPv6 primer

- IPv6 addresses are **128-bit**, written as 8 groups of 4 hex digits separated
  by colons.
- **`::`** replaces one run of consecutive all-zero groups. So `2001:DB8:A::1`
  expands to `2001:0DB8:000A:0000:0000:0000:0000:0001`.
- **`/64`** is the standard LAN prefix length (the IPv6 equivalent of `/24`).
- **`2001:DB8::/32`** is the reserved documentation/example range — used here
  because it is meant for labs and docs.

## R1 configuration

```
enable
configure terminal
hostname R1

! enables IPv6 routing — without it the router won't route IPv6 at all
ipv6 unicast-routing

interface gigabitEthernet0/0
 ipv6 address 2001:DB8:A::1/64
 no shutdown
 exit

interface gigabitEthernet0/1
 ipv6 address 2001:DB8:C::1/64
 no shutdown
 exit

! static route to the far LAN
ipv6 route 2001:DB8:B::/64 2001:DB8:C::2

end
copy running-config startup-config
```

## R2 configuration

```
enable
configure terminal
hostname R2

ipv6 unicast-routing

interface gigabitEthernet0/0
 ipv6 address 2001:DB8:C::2/64
 no shutdown
 exit

interface gigabitEthernet0/1
 ipv6 address 2001:DB8:B::1/64
 no shutdown
 exit

ipv6 route 2001:DB8:A::/64 2001:DB8:C::1

end
copy running-config startup-config
```

## Key commands

- `ipv6 unicast-routing` — the single most important IPv6 command. IPv6 routing
  is **off by default**; this turns it on. Forget it and nothing routes. (IPv4
  routing is on by default, so there's no IPv4 equivalent.)
- `ipv6 address 2001:DB8:A::1/64` — assigns the IPv6 address + prefix length.
- `ipv6 route <dest-prefix> <next-hop>` — the IPv6 version of `ip route`
  (static route to a non-directly-connected network).

## Set the PCs

Each PC → **Desktop → IP Configuration → IPv6 section**:

- PC1: IPv6 `2001:DB8:A::10`, prefix `64`, gateway `2001:DB8:A::1`
- PC2: IPv6 `2001:DB8:B::10`, prefix `64`, gateway `2001:DB8:B::1`

## Testing connectivity

From PC1: `ping 2001:DB8:B::10` (PC2) → succeeds. IPv6 traffic routed across
both routers via the static routes.

## Verify

```
show ipv6 interface brief    ! IPv6 addresses per interface (note the FE80:: link-local too)
show ipv6 route              ! IPv6 routing table (C, L, S codes, like IPv4)
```

`show ipv6 route` should list the static route with an **S** and connected
routes with **C**. Every interface also has an automatic **link-local** address
starting with `FE80::` — IPv6 generates one on every interface for local
communication.

## Troubleshooting

| Symptom | Likely cause |
|---------|-------------|
| Ping across networks fails | Forgot `ipv6 unicast-routing` on a router |
| Cross-network ping fails, gateway works | Static route missing on one router |
| Interface has only FE80:: address | `ipv6 address` not configured (only link-local auto-generated) |
| PC can't reach gateway | PC IPv6/prefix/gateway wrong |
| Address rejected | Bad `::` compression (only one `::` allowed per address) |

## Why it matters

IPv4 has ~4.3 billion addresses and the world ran out; IPv6 has 340 undecillion.
Modern networks run **dual-stack** (IPv4 + IPv6 together). The essentials —
address format, `::` compression, `ipv6 unicast-routing`, and `ipv6 route` —
are a solid chunk of the CCNA.

## Files

- `ipv6.pkt` — the Packet Tracer project file.
- `topology.png` — topology screenshot.
