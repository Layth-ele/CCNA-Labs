# RIPv2 — Dynamic Routing with RIP version 2 (Cisco Packet Tracer)

Two routers running **RIPv2**, a distance-vector routing protocol. Instead of
typing static routes by hand, each router advertises its connected networks and
learns the far LAN automatically. RIP picks paths by **hop count** (fewest
routers to cross), up to a maximum of 15 hops.

## Topology

![RIPv2 lab topology](topology.png)

```
   LAN A                                          LAN B
 192.168.1.0/24                            192.168.2.0/24
     |                                            |
  +-------+  G0/0     WAN 10.0.0.0/30      G0/0  +-------+
  |Switch1|--[ R1 ]==========================[ R2 ]--|Switch2|
  +---+---+       Gig0/1            Gig0/0        +---+---+
    PC1                                            PC2
```

Both routers connect the LAN on Gig0/0 and the WAN on Gig0/1 (R1) / Gig0/0 (R2)
— match the config to your actual cabling.

## Equipment

- 2 × Router (2911)
- 2 × Switch (2960)
- 2 × PC
- Copper straight-through cables

## Addressing table

| Device | Interface   | IP Address     | Mask            | Role          |
|--------|-------------|----------------|-----------------|---------------|
| R1     | Gig0/0      | 192.168.1.1    | 255.255.255.0   | LAN A gateway |
| R1     | Gig0/1      | 10.0.0.1       | 255.255.255.252 | WAN           |
| R2     | Gig0/0      | 10.0.0.2       | 255.255.255.252 | WAN           |
| R2     | Gig0/1      | 192.168.2.1    | 255.255.255.0   | LAN B gateway |
| PC1    | —           | 192.168.1.10   | 255.255.255.0   | GW 192.168.1.1 |
| PC2    | —           | 192.168.2.10   | 255.255.255.0   | GW 192.168.2.1 |

## Key points about RIPv2

- **Classful `network` statements** — RIP takes the classful network address with
  **no wildcard mask** (e.g. `network 192.168.1.0`, not a `/24` or wildcard).
- **`version 2`** — RIPv1 is the default and does not send subnet masks. Always
  set `version 2` so masks are carried (classless / VLSM support).
- **`no auto-summary`** — stops RIP from summarizing to classful boundaries at
  network borders, so the real /30 and /24 subnets are advertised correctly.
- Metric is **hop count**; max **15** hops (16 = unreachable).

## R1 configuration

```
enable
configure terminal

interface gigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

interface gigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
 exit

router rip
 version 2
 no auto-summary
 network 192.168.1.0
 network 10.0.0.0
 exit

end
copy running-config startup-config
```

## R2 configuration

```
enable
configure terminal

interface gigabitEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
 exit

interface gigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
 exit

router rip
 version 2
 no auto-summary
 network 192.168.2.0
 network 10.0.0.0
 exit

end
copy running-config startup-config
```

## What each part does

- `ip address ... ` + `no shutdown` — assign each interface its IP and bring it up.
- `router rip` — start the RIP process.
- `version 2` — use RIPv2 so subnet masks are advertised.
- `no auto-summary` — advertise the actual subnets instead of summarizing to /8, /16, /24.
- `network 192.168.1.0` / `network 10.0.0.0` — tell RIP which connected
  networks to advertise and which interfaces to run on. Use the **classful**
  network, no mask.

## Set the PCs

Each PC → **Desktop → IP Configuration**:

- PC1: IP `192.168.1.10`, mask `255.255.255.0`, gateway `192.168.1.1`
- PC2: IP `192.168.2.10`, mask `255.255.255.0`, gateway `192.168.2.1`

## Testing connectivity

From PC1: `ping 192.168.2.10` (PC2) → succeeds, using a route RIP learned
dynamically rather than a static route you typed in.

## Verify

```
show ip route rip            ! learned routes show with an R
show ip protocols            ! confirms RIP v2, no auto-summary, networks
show ip rip database         ! the RIP routing database
```

Expected: the far LAN in the routing table marked with an **R**, e.g. on R2
`R 192.168.1.0/24 [120/1] via 10.0.0.1`. The `120` is RIP's administrative
distance; the `1` is the hop count.

## Troubleshooting

| Symptom | Likely cause |
|---------|-------------|
| No routes learned | `network` statement missing for the WAN network on one/both routers |
| Route shows wrong/summarized mask | `no auto-summary` not configured |
| Neighbors don't exchange masks | `version 2` not set (defaults to v1) |
| Ping fails but routes present | PC IP / mask / gateway wrong |
| WAN network not advertised | interface still `shutdown`, or wrong `network` classful address |

## Why it matters

RIPv2 is the simplest dynamic routing protocol and a clean first step up from
static routes: it learns routes automatically but scales poorly (15-hop limit,
slow convergence, hop-count-only metric). Understanding its classful `network`
statements and the `version 2` / `no auto-summary` pair is the foundation for
seeing why EIGRP and OSPF improved on it.

## Files

- `ripv2.pkt` — the Packet Tracer project file.
- `topology.png` — topology screenshot.
