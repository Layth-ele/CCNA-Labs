# Command Reference — CCNA Labs (01–06)

Every command used across the labs so far, grouped by purpose. Cisco IOS
commands run on routers/switches; PC commands run in the Desktop → Command
Prompt; shell/git commands run in the Mac Terminal.

---

## 1. Moving between IOS modes

| Command | What it does |
|---------|-------------|
| `enable` (`en`) | Enter privileged EXEC mode (`R1#`) |
| `configure terminal` (`conf t`) | Enter global config mode (`R1(config)#`) |
| `interface gigabitEthernet0/0` | Enter interface config (`R1(config-if)#`) |
| `interface range fastEthernet0/1 - 2` | Configure several ports at once |
| `interface gigabitEthernet0/0.10` | Create/enter a subinterface |
| `interface loopback0` | Create/enter a virtual loopback |
| `router ospf 1` | Enter OSPF router config (`R1(config-router)#`) |
| `ip dhcp pool LAN-POOL` | Enter DHCP pool config |
| `exit` | Go back one mode level |
| `end` | Jump straight back to privileged EXEC |

## 2. Basic device setup

| Command | What it does |
|---------|-------------|
| `hostname R1` | Name the device |
| `copy running-config startup-config` (`wr`) | Save config so it survives reload |

## 3. Interface configuration

| Command | What it does |
|---------|-------------|
| `ip address 192.168.1.1 255.255.255.0` | Assign an IP + mask |
| `no ip address` | Remove the IP from an interface |
| `no shutdown` | Turn the interface on |
| `shutdown` | Turn the interface off |
| `description LINK-TO-LAN` | Add a label (cosmetic) |
| `ip nat inside` / `ip nat outside` | Mark the private / public NAT side |
| `encapsulation dot1Q 10` | Tag a subinterface for a VLAN |

## 4. Switching (VLANs & trunks)

| Command | What it does |
|---------|-------------|
| `vlan 10` | Create VLAN 10 |
| `name IT` | Name the VLAN |
| `switchport mode access` | Set port as an access (end-device) port |
| `switchport access vlan 10` | Put the port in VLAN 10 |
| `switchport mode trunk` | Set port as a trunk (carries all VLANs) |
| `switchport nonegotiate` | Disable DTP auto-negotiation |

## 5. Routing

| Command | What it does |
|---------|-------------|
| `ip route 192.168.2.0 255.255.255.0 10.0.0.2` | Static route to a network |
| `ip route 0.0.0.0 0.0.0.0 200.0.0.2` | Default route (gateway of last resort) |
| `router ospf 1` | Start OSPF process 1 |
| `network 192.168.1.0 0.0.0.255 area 0` | Advertise a network in OSPF area 0 |

## 6. DHCP (router as server)

| Command | What it does |
|---------|-------------|
| `ip dhcp excluded-address 192.168.10.1 192.168.10.10` | Reserve IPs DHCP won't hand out |
| `ip dhcp pool LAN-POOL` | Create an address pool |
| `network 192.168.10.0 255.255.255.0` | Range of addresses to lease |
| `default-router 192.168.10.1` | Gateway given to clients |
| `dns-server 8.8.8.8` | DNS server given to clients |

## 7. NAT / PAT

| Command | What it does |
|---------|-------------|
| `access-list 1 permit 192.168.1.0 0.0.0.255` | Define which addresses may be translated |
| `ip nat inside source list 1 interface gigabitEthernet0/1 overload` | Translate (PAT) matched traffic to the outside interface IP |

## 8. Verification / show commands

| Command | Shows |
|---------|-------|
| `show ip interface brief` | Every interface: IP + up/down status |
| `show ip route` | The routing table (C, S, O, L codes) |
| `show ip protocols` | Which routing protocols are running |
| `show ip ospf neighbor` | OSPF neighbors and their state (FULL) |
| `show vlan brief` | VLANs and which ports are in each |
| `show interfaces status` | Per-port status, VLAN, connected/notconnect |
| `show interfaces trunk` | Which ports are trunks and allowed VLANs |
| `show interfaces gig0/1 switchport` | Detailed switchport mode info |
| `show cdp neighbors` | Directly connected Cisco devices + ports |
| `show ip nat translations` | Active NAT translations |
| `show ip nat statistics` | NAT inside/outside interfaces + hit counts |
| `show access-lists` | Configured ACLs and their entries |
| `show running-config` | The full active config |
| `show running-config \| include nat` | Filter the config for matching lines |

## 9. Removing / undoing config

Put `no` in front of most commands to reverse them:

| Command | What it does |
|---------|-------------|
| `no shutdown` | Enable an interface (undo shutdown) |
| `no ip address` | Remove an IP |
| `no ip route 192.168.2.0 255.255.255.0 10.0.0.2` | Delete a static route |
| `no network 10.0.0.2 0.0.0.3 area 0` | Remove an OSPF network line |

## 10. PC commands (Desktop → Command Prompt)

| Command | What it does |
|---------|-------------|
| `ipconfig` | Show the PC's IP, mask, gateway |
| `ping 192.168.2.2` | Send 4 test packets to a host |
| `ping -t 8.8.8.8` | Ping continuously (Ctrl+C to stop) |
| `ping -n 10 <ip>` | Send a specific number of packets |

## 11. Mac Terminal (shell) commands

| Command | What it does |
|---------|-------------|
| `cd <folder>` | Change directory |
| `ls` / `ls -a` | List files (`-a` includes hidden ones) |
| `pwd` | Print the current folder path |
| `mkdir <name>` | Make a folder |
| `touch <file>` | Create an empty file |
| `mv <src> <dest>` | Move / rename a file |
| `cp <src> <dest>` | Copy a file |
| `cat <file>` | Print a file's contents |
| `head -3 <file>` | First 3 lines of a file |
| `grep "text" <file>` | Search a file for text (`-A5` = 5 lines after) |
| `find ~ -name "file" 2>/dev/null` | Search for a file by name |
| `nano <file>` / `open -e <file>` | Edit a file (nano / TextEdit) |

## 12. Git commands

| Command | What it does |
|---------|-------------|
| `git init` | Start tracking a folder |
| `git add .` | Stage all changes |
| `git commit -m "message"` | Save a snapshot with a message |
| `git push` | Upload commits to GitHub |
| `git pull --no-edit` | Download + merge remote changes |
| `git config pull.rebase false` | Set merge as the pull strategy |
| `git remote add origin <url>` | Link the repo to GitHub |
| `git remote -v` | Show the linked remote |
| `git remote set-url origin <url>` | Change the remote URL |
| `git status` | Show what's changed / staged |
| `git log --oneline` | Compact commit history |
| `git ls-files` | List tracked files |
| `git rm --cached <file>` | Stop tracking a file (keep it on disk) |
| `git branch -M main` | Rename the current branch to main |
| `git mv <old> <new>` | Rename/move a tracked file |
