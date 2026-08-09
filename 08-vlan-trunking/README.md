# VLAN Trunking Between Two Switches (Cisco Packet Tracer)

Two switches connected by a **trunk** link, with VLAN 10 and VLAN 20 on both.
PCs in the same VLAN can communicate across the trunk even though they sit on
different switches; different VLANs stay isolated. This is a pure Layer 2 lab —
no router.

## Topology

![VLAN trunking lab topology](topology.png)

```
     VLAN 10        VLAN 20              VLAN 10        VLAN 20
      PC1            PC2                  PC3            PC4
       |              |                    |              |
     Gig0/1        Gig0/2               Gig0/1        Gig0/2
       +----[ SW1 ]----+     trunk     +----[ SW2 ]----+
                 Fa0/1 =============== Fa0/3
```

Two switches, four PCs, one trunk link. In this build the PCs connect to the
switches' **Gig0/1 / Gig0/2** ports, and the inter-switch trunk runs from
**SW1 Fa0/1** to **SW2 Fa0/3** (the trunk ports don't have to match on both
ends — each side just has to be a trunk).

## Equipment

- 2 × Switch (2960)
- 4 × PC
- Copper cables

## Addressing table

| Device    | VLAN | Switch port | IP Address    | Subnet Mask   |
|-----------|------|-------------|---------------|---------------|
| PC1 (SW1) | 10   | Gig0/1      | 192.168.10.10 | 255.255.255.0 |
| PC2 (SW1) | 20   | Gig0/2      | 192.168.20.10 | 255.255.255.0 |
| PC3 (SW2) | 10   | Gig0/1      | 192.168.10.20 | 255.255.255.0 |
| PC4 (SW2) | 20   | Gig0/2      | 192.168.20.20 | 255.255.255.0 |

No gateways needed — this lab only tests same-VLAN (Layer 2) communication, and
there is no router to route between VLANs.

## SW1 configuration

```
enable
configure terminal
hostname SW1

vlan 10
 name Sales
 exit
vlan 20
 name Engineering
 exit

interface gigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
 exit
interface gigabitEthernet0/2
 switchport mode access
 switchport access vlan 20
 exit

interface fastEthernet0/1
 switchport mode trunk
 exit

end
copy running-config startup-config
```

## SW2 configuration

Identical, just a different hostname and trunk port (Fa0/3):

```
enable
configure terminal
hostname SW2

vlan 10
 name Sales
 exit
vlan 20
 name Engineering
 exit

interface gigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
 exit
interface gigabitEthernet0/2
 switchport mode access
 switchport access vlan 20
 exit

interface fastEthernet0/3
 switchport mode trunk
 exit

end
copy running-config startup-config
```

## Access ports vs the trunk

- **Access ports** (Gig0/1, Gig0/2 here) carry **one** VLAN and connect to end
  devices (PCs). Each PC's port belongs to a single VLAN.
- **The trunk** (SW1 Fa0/1 ↔ SW2 Fa0/3) carries **all** VLANs between the
  switches. It adds an 802.1Q tag to every frame so the far switch knows which
  VLAN it belongs to. Without the trunk, VLAN 10 on SW1 and VLAN 10 on SW2 would
  be isolated islands.

Always hard-set trunk ports with `switchport mode trunk` rather than leaving
them in the default "dynamic auto" mode — otherwise a port may auto-negotiate a
trunk you didn't intend (and it's a security best practice).

## Testing connectivity

| From | Ping | Expected | Why |
|------|------|----------|-----|
| PC1 (VLAN 10, SW1) | `ping 192.168.10.20` (PC3) | Success | Same VLAN, carried across the trunk |
| PC2 (VLAN 20, SW1) | `ping 192.168.20.20` (PC4) | Success | Same VLAN across the trunk |
| PC1 (VLAN 10) | `ping 192.168.20.20` (PC4) | Fail | Different VLAN, no router to route between them |

The first two prove the trunk carries both VLANs between switches. The third
proves VLANs are still isolated (routing between them needs a router or Layer 3
switch — see the inter-VLAN routing lab).

## Verify

```
show vlan brief          ! VLAN-to-port assignments (PC ports show under their VLAN)
show interfaces trunk    ! confirms the trunk port + which VLANs it carries
show interfaces status   ! per-port status, VLAN, connected/notconnect
show cdp neighbors       ! confirms which local port connects to the other switch
```

`show interfaces trunk` should list the trunk port (Fa0/1 on SW1, Fa0/3 on SW2)
carrying VLANs 10 and 20. `show cdp neighbors` confirms SW1 Fa0/1 ↔ SW2 Fa0/3.

## Troubleshooting

| Symptom | Likely cause |
|---------|-------------|
| Same-VLAN PCs across switches can't ping | Trunk not up — both ends must be trunking |
| VLAN assigned but PC still in VLAN 1 | Access-vlan command applied to the wrong port |
| Trunk formed on an unexpected port | Port left in "dynamic auto" mode auto-negotiated a trunk |
| Different-VLAN ping works (shouldn't) | Ports accidentally in the same VLAN |

Use `show interfaces status` and `show cdp neighbors` to confirm which ports
actually have PCs and which is the trunk — assumptions about port numbers are
the #1 source of confusion.

## Files

- `vlan-trunking.pkt` — the Packet Tracer project file.
- `topology.png` — topology screenshot.
