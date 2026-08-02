# Static Routing Between Two Routers (Cisco Packet Tracer)

Two LANs, each behind its own router, joined by a point-to-point WAN link.
Because the far LAN is not directly connected, each router is taught the route
manually with the `ip route` command. This is the core idea behind IP routing.

## Topology

```
     LAN A                                              LAN B
  192.168.1.0/24                                    192.168.2.0/24
      │                                                   │
  ┌───────┐   Gig0/0        WAN 10.0.0.0/30        Gig0/0  ┌───────┐
  │Switch1│──[ R1 ]Gig0/1─────────────────────Gig0/0[ R2 ]│Switch2│
  └──┬─┬──┘   .1          10.0.0.1   10.0.0.2          .1  └──┬─┬──┘
   PC1 PC2                                                  PC3 PC4
```

Note on cabling: in this build R1 uses **Gig0/1** for the WAN and R2 uses
**Gig0/0** for the WAN (the ports don't have to match on both ends — the config
just has to match whichever port each cable is plugged into).

## Addressing table

| Device | Interface        | IP Address   | Subnet Mask       | Role        |
|--------|------------------|--------------|-------------------|-------------|
| R1     | Gig0/0           | 192.168.1.1  | 255.255.255.0     | LAN A gwy   |
| R1     | Gig0/1           | 10.0.0.1     | 255.255.255.252   | WAN         |
| R2     | Gig0/0           | 10.0.0.2     | 255.255.255.252   | WAN         |
| R2     | Gig0/1           | 192.168.2.1  | 255.255.255.0     | LAN B gwy   |
| PC1    | —                | 192.168.1.2  | 255.255.255.0     | GW 192.168.1.1 |
| PC2    | —                | 192.168.1.3  | 255.255.255.0     | GW 192.168.1.1 |
| PC3    | —                | 192.168.2.2  | 255.255.255.0     | GW 192.168.2.1 |
| PC4    | —                | 192.168.2.3  | 255.255.255.0     | GW 192.168.2.1 |

The `/30` (255.255.255.252) on the WAN link gives exactly two usable host
addresses — all a point-to-point link needs.

## Equipment

- 2 × Router (2911)
- 2 × Switch (2960)
- 4 × PC
- Copper straight-through cables

## R1 configuration

```cisco
enable
configure terminal
hostname R1

interface gigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

interface gigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
 exit

ip route 192.168.2.0 255.255.255.0 10.0.0.2

end
copy running-config startup-config
```

## R2 configuration

```cisco
enable
configure terminal
hostname R2

interface gigabitEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
 exit

interface gigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
 exit

ip route 192.168.1.0 255.255.255.0 10.0.0.1

end
copy running-config startup-config
```

The `ip route` line is the heart of the lab. On R1 it reads: *"to reach
192.168.2.0/24, send packets to 10.0.0.2."* Without it, a router only knows its
own directly-connected networks and drops everything else. **Both** routers need
their route — otherwise packets cross but replies can't return.

## Testing connectivity

1. PC1 → `ping 192.168.1.1` — its own gateway (should always work).
2. R1 → `ping 10.0.0.2` — the WAN link between routers.
3. PC1 → `ping 192.168.2.2` — the end-to-end test across both routers.

A working cross-network ping shows **TTL=126** (down 2 from 128) — proof the
packet passed through two routers. The first packet often times out while ARP
resolves, then replies follow.

## Verify the routes

On each router, `show ip route` should list the static route with an **S**:

```
S    192.168.2.0/24 [1/0] via 10.0.0.2      (on R1)
S    192.168.1.0/24 [1/0] via 10.0.0.1      (on R2)
```

## Troubleshooting

| Symptom | Likely cause |
|---------|-------------|
| Cross-network ping fails, gateway ping works | Static route missing on one router |
| Routers can't ping each other over WAN | WAN cable in the wrong port — IPs and cabling don't match. Check with `show cdp neighbors` |
| `% overlaps with GigabitEthernetX` when swapping IPs | Remove old IPs first with `no ip address`, then reassign |
| Interface shows up/down | Other end of the link is shut down or unconfigured |

`show cdp neighbors` is the key tool: it reveals which local port connects to
which remote port, so you can confirm the cabling matches your intended config.

## Files

 `static-routing.pkt` — the Packet Tracer project file.
