# DHCP Server on a Router (Cisco Packet Tracer)

A single LAN where the router hands out IP addresses automatically using DHCP.
Instead of typing a static IP into every PC, each client is set to **DHCP** and
leases an address, subnet mask, gateway, and DNS server from the router.

## Topology

```
        +--------+
        |   R1   |  Gig0/0 = 192.168.10.1/24  (gateway + DHCP server)
        +---+----+
            |
        +---+----+
        | Switch |
        +-+--+-+-+
          |  |  |
        PC1 PC2 PC3   (all set to DHCP)
```

## Equipment

- 1 × Router (2911)
- 1 × Switch (2960)
- 3 × PC
- Copper straight-through cables

## Addressing table

| Device  | Interface | IP Address        | Notes                     |
|---------|-----------|-------------------|---------------------------|
| R1      | Gig0/0    | 192.168.10.1 /24  | Gateway + DHCP server     |
| PC1     | —         | assigned by DHCP  | set to DHCP (not static)  |
| PC2     | —         | assigned by DHCP  | set to DHCP (not static)  |
| PC3     | —         | assigned by DHCP  | set to DHCP (not static)  |

The router keeps a **static** IP (192.168.10.1) — a DHCP server always needs a
fixed address. Only the client PCs receive automatic ones. With the excluded
range below, leases start at 192.168.10.11.

## R1 configuration

```
enable
configure terminal
hostname R1

interface gigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

ip dhcp excluded-address 192.168.10.1 192.168.10.10

ip dhcp pool LAN-POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 exit

end
copy running-config startup-config
```

## What each DHCP line does

- `ip dhcp excluded-address 192.168.10.1 192.168.10.10` — reserve .1–.10 so DHCP
  won't hand them out (protects the router and leaves room for static devices).
  Leasing starts at .11.
- `ip dhcp pool LAN-POOL` — creates a pool named LAN-POOL.
- `network 192.168.10.0 255.255.255.0` — the range of addresses to give out.
- `default-router 192.168.10.1` — the gateway handed to clients.
- `dns-server 8.8.8.8` — the DNS server handed to clients.

## Set the PCs to DHCP

On each PC: **Desktop → IP Configuration → DHCP**. Within a moment it shows
"DHCP request successful" and fills in an address (e.g. 192.168.10.11), mask
255.255.255.0, and gateway 192.168.10.1 — all automatically.

## Verification

```
show ip dhcp binding    ! lists each leased IP and the client MAC
show ip dhcp pool       ! shows how many addresses have been leased
```

## Testing connectivity

1. On each PC run `ipconfig` and confirm it got a 192.168.10.x address
   (.11 or higher) with gateway 192.168.10.1.
2. Ping between PCs, e.g. from PC1: `ping 192.168.10.12`.

## Troubleshooting

| Symptom | Likely cause |
|---------|-------------|
| PC shows 169.254.x.x (APIPA) | No DHCP reply — pool `network` wrong or cable down |
| PC gets IP but no gateway | Missing `default-router` in the pool |
| Wrong subnet handed out | `network` statement doesn't match the LAN |
| Router address leased to a PC | Forgot the `ip dhcp excluded-address` line |

## Files

- `dhcp.pkt` — the Packet Tracer project file.
