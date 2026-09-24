# Junos OS — Full Commands Cheat Sheet

> Complete ordered cheat sheet built from `1.md` → `37.md`.
> Sections follow course order. Subsections group related commands.

---

## 1. Access & CLI Basics

### 1.1 SSH Remote Access
```bash
ssh username@IP_ADDRESS
ssh employee@172.16.10.1
ssh r.chen@10.1.2.2
ssh user@IP
```
- SSH = TCP/22, encrypted. Telnet = TCP/23, clear-text, avoid.

### 1.2 CLI Modes
```bash
configure          # Operational (>) -> Configuration (#) [edit]
exit               # Configuration -> Operational
```
- `>` = Operational Mode, `#` = Configuration Mode `[edit]`

### 1.3 CLI Help, Completion, History, Paging
```bash
?                          # show available options
show system ?              # example: list completions
show system a              # may be ambiguous (alarms vs audit)
show system al             # disambiguate
show interfaces terse | ?  # discover pipe filters
```
- `Tab` / `Space` = auto-complete
- `↑` = command history
- Paging: `Space` = next screen, `Enter` = next line, `q` = quit, `h` = help at `---(more)---`

### 1.4 Operational Mode Command Families
```bash
show interfaces
monitor interface traffic
clear interfaces statistics all
request system reboot
```
| Command | Purpose |
|---|---|
| `show` | Display snapshot |
| `monitor` | Continuously updated / real-time |
| `clear` | Clear / reset statistics or state |
| `request` | Administrative actions |

### 1.5 Run Operational Commands from Configuration Mode
```bash
run show interfaces terse
run show interfaces terse xe-0/1/1
# syntax:
run <operational-mode-command>
```

### 1.6 Unix Shell / FreeBSD Shell
```bash
cli            # Unix shell (root@device%) -> Junos CLI (root@device>)
start shell    # Junos CLI -> FreeBSD shell
pwd
cd /var/log
ls -la
```

---

## 2. System Verification — `show system`

### 2.1 System Status
```bash
show system uptime          # uptime, boot time, current time, protocol start
show system information     # model, Junos version, hostname. e.g. MX204, 22.4R1.10
show version                # detailed versions for all Junos modules (pre/post upgrade)
show chassis hardware       # chassis hardware inventory
show chassis routing-engine # CPU, memory, temperature
show system users           # currently logged-in users (CLI, J-Web, NETCONF)
request system logout user <username>  # terminate session
show system commit          # commit history: number, timestamp, user, method, comment
```

### 2.2 LLDP — Neighbor Discovery
```bash
show lldp neighbors
show route
show ospf
show ospf neighbor
show spanning-tree
show interfaces ge-0/0/1
```

---

## 3. Interfaces — Naming, Status, Descriptions

### 3.1 Naming Convention
```text
<type>-<FPC>/<PIC>/<port>[.<unit>]
ge-0/0/0, xe-1/1/2, et-2/0/1, xe-0/1/1.0, ge-0/0/6
fe- = 100 Mbps, ge- = 1 Gbps, xe- = 10 Gbps, et- = 40 Gbps+
FPC = Flexible PIC Concentrator, PIC = Physical Interface Card
inet = IPv4, inet6 = IPv6
```

### 3.2 Quick Interface Status
```bash
show interfaces terse
show interfaces terse xe-0/1/1
show interfaces terse xe-0/1/0
show interfaces terse xe-0/1/0.20
show interfaces terse lo0
show interfaces terse irb
show interfaces terse | match ae0
show interfaces terse xe-0/1/*
show interfaces terse routing-instance TEST_INSTANCE
show interfaces <interface> | match snmp   # find SNMP index
```

### 3.3 Detail Levels
```bash
show interfaces
show interfaces xe-0/1/2
show interfaces brief
show interfaces detail
show interfaces extensive
show interfaces xe-0/1/5 extensive
show interfaces extensive xe-0/1/5
show interfaces extensive xe-0/1/5 | match error
```
- Order: `terse -> brief -> normal -> detail -> extensive`

### 3.4 Descriptions
```bash
show interfaces descriptions
```

### 3.5 Live Interface Monitoring
```bash
monitor interface traffic       # all interfaces, live
monitor interface xe-0/1/5      # single interface, live: bytes/packets/errors/drops
# monitor controls: q/ESC=quit, Space=more, c=clear, r=refresh
```

### 3.6 Control-Plane Traffic Capture
```bash
monitor traffic interface xe-0/1/5
monitor traffic interface xe-0/1/5 no-resolve
monitor traffic interface xe-0/1/5 detail no-resolve
monitor traffic interface xe-0/1/5 extensive no-resolve
monitor traffic interface xe-0/1/5 write-file <filename>
```

### 3.7 Clear Counters
```bash
clear interfaces statistics all
clear interfaces statistics xe-0/1/5
```

---

## 4. CLI Pipe Filters — `|`

### 4.1 Core Filters
```bash
show interfaces terse xe-0/1/* | match down
show interfaces xe-0/1/* | match physical
show interfaces xe-0/1/* | match "physical|flapped"   # OR inside quotes
show interfaces terse xe-0/1/1 | except fe80           # exclude link-local
show interfaces xe-0/1/* | match physical | except down  # AND via chaining
show interfaces xe-0/1/* | match physical | count
show interfaces xe-0/1/2 | find logical                # start at first match
show interfaces xe-0/1/5.0 | find inet | except cache
show log messages | match error
show log messages | match warning
show log messages | match ge-
show log messages | match rpd
show log messages | match mgd
show log messages | last 20
show log interactive-commands | match user | last 6
show route 172.16.40.0/24 exact | refresh 10
show ospf neighbor | refresh 10
show <command> | save <filename>
show <command> | append <filename>
<command> | last <X>
<command> | no-more
show configuration | no-more
```

| Filter | Function |
|---|---|
| `match` | Show lines containing text |
| `except` | Remove lines containing text |
| `find` | Start output at first occurrence |
| `count` | Count output lines |
| `last` | Display only end of output |
| `save` | Save output to file |
| `append` | Append output to file |
| `tee` | Write to stdout + file |
| `no-more` | Disable `--more--` pagination |
| `refresh N` | Repeat every N seconds |

---

## 5. Configuration Views — Hierarchy vs Set

### 5.1 View Active Configuration
```bash
show configuration
show configuration | display set
show configuration | display omit          # reveal apply-flags omit
show configuration | no-more
show configuration system
show configuration interfaces
show configuration protocols
show configuration system | display set
show configuration interfaces | display set
show configuration protocols | display set
show configuration interfaces xe-0/1/2
show configuration interfaces xe-0/1/2 | display set
show configuration interfaces ge-0/0/1
show configuration interfaces ge-0/0/1 | display set
show configuration vlans
show configuration vlans | display set
show configuration firewall
show configuration firewall family inet filter LAN_TO_WAN
show configuration firewall family inet filter LAN_TO_WAN | display set
show configuration firewall family inet filter LAN_TO_WAN | display set relative
show configuration routing-options | display set
show configuration routing-options static
show configuration routing-options rib inet6.0 static
show configuration protocols ospf
show configuration protocols ospf | display set
show configuration protocols ospf3
show configuration protocols ospf3 | display set
show configuration protocols lldp | display set
show configuration policy-options
show configuration policy-options policy-statement <POLICY>
show configuration routing-instances
show configuration system login user employee | display set
show configuration interfaces xe-0/1/5 | display inheritance
show configuration interfaces xe-0/1/6 | display inheritance no-comments
show configuration interfaces xe-0/1/5 | display set | display inheritance
show configuration interfaces xe-0/1/5 | display set
```

### 5.2 Search Configuration
```bash
show configuration interfaces | display set | match mtu
show configuration | display set | match mtu
show configuration | display set | match xe-0/1/1
show configuration | display set | match ge-0/0/1
show configuration | display set | match xe-0/1/[1-3]
show configuration | compare rollback 1
show configuration | compare SATURDAY_CHANGES.txt
```

### 5.3 XML View (Automation)
```bash
show system information
show system information | display xml
show <command> | display xml
```

---

## 6. Candidate Configuration, Commit, Rollback

### 6.1 Basic Workflow
```bash
configure
show | compare          # + added, - removed. Most important before commit
commit                  # activate, stay in [edit]
commit and-quit         # activate + exit to operational
commit check            # validate only, no activation
rollback                # discard uncommitted candidate changes (= candidate = active)
exit
```

### 6.2 History & Rescue
```bash
show system commit
rollback 0              # load current active into candidate
rollback 1              # load previous commit into candidate + needs commit
rollback 2
rollback 49             # oldest retained (up to 49)
rollback rescue         # restore rescue snapshot (needs commit)
show configuration | compare rollback 1
```

### 6.3 Safe Remote Commits
```bash
commit confirmed 5
commit confirmed 2
commit confirmed 10
commit confirmed        # default 10 min auto-rollback if not confirmed
commit                  # confirm (also commit check cancels pending rollback per module)
commit comment "Changed WAN interface"
commit comment "Updated server LAN IPv4 address"
commit confirmed 2 comment "Deleted the Internet" and-quit
commit at "18:00:00"
commit at "2023-10-06 14:00:00"
clear system commit     # cancel scheduled commit
```

### 6.4 Factory Default & Rescue (New Device)
```bash
load factory-default
delete system commit factory-settings   # remove factory-settings marker before commit
request system configuration rescue save
request system configuration rescue delete
# rollback rescue (see 6.2)
```

---

## 7. Configuration Editing — `set / delete / edit`

### 7.1 Add / Modify
```bash
set <configuration>
set interfaces xe-0/1/2 unit 0 family inet address 172.16.20.1/24
set interfaces xe-0/1/1 unit 0 family inet address 192.168.1.0/24
```

### 7.2 Delete (Hierarchy-Aware)
```bash
delete interfaces xe-0/1/2 unit 0 family inet address 172.16.200.1/24
delete interfaces xe-0/1/5 unit 0 family inet6 address 2001:db8:1:2::1/64  # only address
delete interfaces xe-0/1/5 unit 0 family inet6                            # entire inet6
delete interfaces xe-0/1/5 unit 0                                         # entire unit
delete interfaces xe-0/1/5                                                # entire interface
delete interfaces                                                         # all interfaces
delete                                                                    # entire candidate (asks confirm; rollback to undo)
delete interfaces xe-0/1/1 mtu 1800
delete interfaces xe-0/1/3 unit 0 family inet mtu 900
delete protocols lldp                                                     # entire LLDP hierarchy
wildcard delete interfaces ge-0/1/*                                       # bulk delete with confirm
```

### 7.3 Navigation
```bash
edit system
edit interfaces xe-0/1/3 unit 0 family inet
edit interfaces interface-range INTERFACES_VLAN_50
up
up 2
top
top
commit   # must commit private config from top
```

### 7.4 Exclusive / Private Candidate
```bash
configure exclusive   # lock shared candidate for one user
configure private     # temporary private candidate; discarded on exit if uncommitted
```

### 7.5 Rename / Replace / Insert / Copy / Move
```bash
rename address 172.16.33.1/24 to address 172.16.30.1/24
rename interfaces xe-0/1/2 to xe-0/1/4
rename address 10.8.9.1/24 to address 10.8.9.8/24
replace pattern xe-0/1/2 with xe-0/1/4
replace pattern xe-0/1/1 with xe-0/1/0.10
replace pattern xe-0/1/2 with xe-0/1/0.20
replace pattern xe-0/1/3 with xe-0/1/0.30
insert firewall family inet filter LAN_TO_WAN term BLOCK_20_123 before term ACCEPT_ALL_ELSE
insert firewall family inet filter LAN_TO_WAN term BLOCK_20_123 after term BLOCK_GUEST_PING
# also: copy, move, insert, annotate, save, load merge, load override (see §14)
```

### 7.6 Disable / Deactivate / Activate / Protect / Annotate
```bash
set interfaces xe-0/1/3 disable
delete interfaces xe-0/1/3 disable
set interfaces xe-0/1/3 unit 0 disable
set protocols lldp interface xe-0/1/1 disable
deactivate interfaces xe-0/1/3
activate interfaces xe-0/1/3
protect protocols lldp
unprotect protocols lldp
annotate system "Do not make changes to this hierarchy"
annotate host-name "This is a good host name"
set firewall family inet filter LAN_TO_WAN apply-flags omit
set routing-options apply-flags omit
set protocols ospf apply-flags omit
```

### 7.7 CLI Emacs Shortcuts
```text
Ctrl-w = delete previous word
Ctrl-a = beginning of line
Ctrl-e = end of line
Esc-b  = back one word
Esc-f  = forward one word
Ctrl-k = delete from cursor to end
```

---

## 8. System — Hostname, Time, NTP, DNS, Management

### 8.1 Hostname & Management Interface
```bash
set system host-name R1
set system host-name TEST_DEVICE
set interfaces fxp0 unit 0 family inet address 172.25.11.1/24
# platform names: fxp0, me0, em0, re0:mgmt-0, re0:mgmt-1, re1:mgmt-0, re1:mgmt-1
```

### 8.2 Time / NTP
```bash
set system time-zone US/Eastern
set date YYYYMMDDHHMM.SS          # operational mode, e.g. set date 202309051530.00
set system ntp server 203.0.113.47
show system uptime                # Current time, Time Source (NTP CLOCK when synced)
show ntp associations             # * = synchronized; st = stratum
```

### 8.3 DNS
```bash
set system name-server 203.0.113.5
ping juniper.net                  # resolves then pings
```

### 8.4 Login Banner
```bash
set system login message "Beware all ye who enter here!"       # before login
set system login message "Authorized access only"
set system login announcement "This is line 1\nand this is line 2"  # after auth (\n=newline)
set system login announcement "Welcome to R1\nAuthorized users only"
set system login ssh root-login allow
```

---

## 9. Users, Login Classes, Authentication

### 9.1 Local Users
```bash
set system login user employee uid 2000
set system login user employee class super-user
set system login user employee authentication encrypted-password "..."
set system login user r.chen class super-user authentication plain-text-password
set system login user employee authentication plain-text-password
set system login user employee authentication encrypted-password "$6$..."
set system login user admin class super-user authentication plain-text-password
set system root-authentication encrypted-password "..."
set system root-authentication plain-text-password
show configuration system login user <username> | display set
```
- `$6$` = SHA-512 hash. `plain-text-password` prompts then stores hashed. `encrypted-password` takes existing hash.

### 9.2 Remote Authentication (RADIUS / TACACS+)
```bash
set system radius-server <SERVER-IP> secret <SECRET>
set system tacplus-server <SERVER-IP> secret <SECRET>
set system authentication-order [ radius tacplus password ]
# password = local. Reject = follow order. Unreachable = local fallback.
```

### 9.3 Login Classes & Permissions
```bash
set system login class FIRST_LINE permissions [ clear network view ]
set system login class FIRST_LINE allow-commands "configure private"
set system login class FIRST_LINE deny-commands "(file).*"
set system login class FIRST_LINE allow-configuration "(interfaces)|(firewall)"
set system login class FIRST_LINE deny-configuration "(groups)"
set system login class FIRST_LINE idle-timeout 5
```
- Defaults: `super-user (all)`, `operator (view,clear,reset,trace,network)`, `read-only (view)`, `unauthorized (none)`
- Flags: `all, clear, configure, network (ping,ssh,telnet,traceroute), view`
- `allow-*` overrides `deny-*`. Defaults cannot be modified.

### 9.4 J-Web (Web Management) + System Services
```bash
set system services ssh
set system services telnet
set system services web-management http
set system services web-management https system-generated-certificate
# access: https://<device-IP>  e.g. https://172.25.11.1
```

---

## 10. Physical vs Logical Interface Configuration

### 10.1 MTU & Addresses
```bash
set interfaces xe-0/1/1 mtu 1800                                          # L2 MTU (physical)
set interfaces xe-0/1/1 unit 0 family inet mtu 900                        # L3 MTU (logical)
set interfaces xe-0/1/1 unit 0 family inet address 172.16.10.1/24
set interfaces xe-0/1/1 unit 0 family inet6 address 2001:db8:0:10::1/64
set interfaces xe-0/1/3 unit 0 family inet6                               # enable IPv6 link-local only
set interfaces xe-0/1/1 description "Corporate LAN"
```

### 10.2 Primary / Preferred Source Selection
```bash
set interfaces xe-0/1/5 unit 0 family inet address 10.1.2.30/24 preferred   # same-subnet source
set interfaces xe-0/1/5 unit 0 family inet address 10.10.20.30/24 primary   # different-subnet source
show interfaces xe-0/1/5.0 | find inet | except cache   # check Is-Preferred / Is-Primary
```

### 10.3 Loopback
```bash
set interfaces lo0 unit 0 family inet address 192.168.1.1/32
set interfaces lo0 unit 0 family inet6 address 2001:db8:1::1/128
set protocols ospf area 0 interface lo0.0
set protocols ospf3 area 0 interface lo0.0
show route 192.168.1.1/32
show interfaces terse lo0
ping 192.168.1.1
```

### 10.4 Aggregated Ethernet (LAG)
```bash
set chassis aggregated-devices ethernet device-count 1   # creates ae0; 2=ae0+ae1; 3=ae0+ae1+ae2
set interfaces ae0 unit 0 family inet address 10.1.2.1/24
set interfaces ae0 aggregated-ether-options lacp active
set interfaces ge-0/0/0 ether-options 802.3ad ae0
set interfaces ge-0/0/1 ether-options 802.3ad ae0
show interfaces terse | match ae0
```

---

## 11. Routing Tables & Route Selection

### 11.1 View Routes
```bash
show route
show route table inet.0
show route table inet6.0
show route 172.16.10.0/24
show route table inet.0 172.16.10.0/24
show route 172.16.10.0/24 table inet.0
show route 2001:db8:10::/64
show route 2001:db8:0:40::/64
show route 0/0 exact
show route ::/0 exact
show route 0.0.0.0/0 exact
show route table TEST_INSTANCE.inet.0
show route protocol direct
show route protocol ospf
show route protocol ospf3
show route table inet.0 protocol direct
show route protocol static
show route table inet6.0
show route ::/0
show route 0.0.0.0/0
show route forwarding-table
show route forwarding-table destination <prefix>
show route forwarding-table table default
show route forwarding-table extensive
```
- `*` = active route, `>` = best path within protocol. `inet.0`=IPv4, `inet6.0`=IPv6.
- Preference: Direct 0 < Static 5 < OSPF 10. LPM first, then preference.

---

## 12. Static Routes

### 12.1 IPv4 / IPv6 Static
```bash
ping 10.1.2.2
ping 10.1.2.2 count 2
ping 2001:db8:1:2::2 count 2
ping 10.1.2.2 source 172.16.10.1
show route 172.16.40.0/24
show route 2001:db8:0:40::/64
set routing-options static route 172.16.40.0/24 next-hop 10.1.2.2
show | compare
commit
show route 172.16.40.0/24
ping 172.16.40.1 count 2
set routing-options rib inet6.0 static route 2001:db8:0:40::/64 next-hop 2001:db8:1:2::2
show route 2001:db8:0:40::/64
ping 2001:db8:0:40::1 count 1
```

### 12.2 Default Routes
```bash
set routing-options static route 0.0.0.0/0 next-hop 10.1.254.254
set routing-options static route 0/0 next-hop 10.1.254.254
set routing-options rib inet6.0 static route ::/0 next-hop 2001:db8:1:254::254
show route 0/0 exact
show route ::/0 exact
ping 203.0.113.5 count 2
```

### 12.3 Advanced Static — Backup, Resolve, No-Readvertise
```bash
set routing-options static route 0/0 next-hop 10.1.254.254
set routing-options static route 0/0 qualified-next-hop 10.10.254.254 preference 7
set routing-options rib inet6.0 static route ::/0 next-hop 2001:db8:1:254::254
set routing-options rib inet6.0 static route ::/0 qualified-next-hop 2001:db8:10:254::254 preference 7
show configuration routing-options static
show configuration routing-options rib inet6.0 static
show route 0/0 exact
show route ::/0 exact
set routing-options static route 172.16.254.0/24 next-hop 172.16.40.99 resolve
set routing-options static route 192.168.200.0/24 next-hop 172.16.10.5 no-readvertise
```

---

## 13. OSPF (OSPFv2 for IPv4, OSPFv3 for IPv6)

### 13.1 Enable OSPF
```bash
set protocols ospf area 0 interface xe-0/1/5.0
set protocols ospf3 area 0 interface xe-0/1/5.0
# passive (advertise LAN, no Hellos/neighbors):
set protocols ospf area 0 interface xe-0/1/1.0 passive
set protocols ospf area 0 interface xe-0/1/2.0 passive
set protocols ospf area 0 interface xe-0/1/3.0 passive
set protocols ospf3 area 0 interface xe-0/1/1.0 passive
# after migration to sub-interfaces:
set protocols ospf area 0 interface xe-0/1/0.10 passive
set protocols ospf area 0 interface xe-0/1/0.20 passive
set protocols ospf area 0 interface xe-0/1/0.30 passive
```

### 13.2 Verify OSPF
```bash
show configuration protocols ospf
show configuration protocols ospf | display set
show configuration protocols ospf3
show configuration protocols ospf3 | display set
show ospf neighbor
show ospf3 neighbor
show route protocol ospf
show route protocol ospf3
show route 172.16.40.0/24
show route 0.0.0.0/0 exact
commit
```

### 13.3 OSPF Traceoptions (Deep Troubleshooting)
```bash
set protocols ospf traceoptions file OSPF_TRACE.txt
set protocols ospf traceoptions flag error
set protocols ospf traceoptions flag event
set protocols ospf traceoptions flag general
set protocols ospf traceoptions flag packets
set protocols ospf traceoptions flag event detail
set protocols ospf traceoptions flag error detail
show log OSPF_TRACE.txt
```

---

## 14. Switching — VLANs, Access, Trunk, MAC, IRB

### 14.1 Create VLANs
```bash
set vlans Corporate vlan-id 10
set vlans Server vlan-id 20
set vlans Guest vlan-id 30
show configuration vlans
show configuration vlans | display set
show vlans
show vlans Guest
show vlans Corporate
show vlans Corporate detail
show vlans FINANCE
configure
show | compare
commit
```

### 14.2 Access Ports
```bash
set interfaces ge-0/0/1 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/1 unit 0 family ethernet-switching vlan members Corporate
set interfaces ge-0/0/4 unit 0 family ethernet-switching vlan members Guest
set interfaces ge-0/0/5 unit 0 family ethernet-switching vlan members Guest
set interfaces xe-0/2/3 unit 0 family ethernet-switching vlan members Guest
show configuration interfaces ge-0/0/1
show configuration interfaces ge-0/0/4
```

### 14.3 Trunk Ports
```bash
set interfaces xe-0/2/0 unit 0 family ethernet-switching interface-mode trunk
set interfaces xe-0/2/0 unit 0 family ethernet-switching vlan members Corporate
set interfaces xe-0/2/0 unit 0 family ethernet-switching vlan members Server
set interfaces xe-0/2/0 unit 0 family ethernet-switching vlan members Guest
set interfaces xe-0/2/0 unit 0 family ethernet-switching vlan members { Corporate Server Guest }
set interfaces xe-0/2/0 unit 0 family ethernet-switching vlan members all
show configuration interfaces xe-0/2/0
show vlans
```

### 14.4 MAC Table
```bash
show ethernet-switching table
show ethernet-switching table vlan Corporate
show ethernet-switching table vlan Guest
```

### 14.5 Router-on-a-Stick (VLAN Tagging + Logical Units)
```bash
set interfaces xe-0/1/0 vlan-tagging
set interfaces xe-0/1/0 unit 10 vlan-id 10
set interfaces xe-0/1/0 unit 10 description "Corporate LAN"
set interfaces xe-0/1/0 unit 10 family inet address 172.16.10.1/24
set interfaces xe-0/1/0 unit 10 family inet6 address 2001:db8:0:10::1/64
set interfaces xe-0/1/0 unit 20 vlan-id 20
set interfaces xe-0/1/0 unit 20 description "Server LAN"
set interfaces xe-0/1/0 unit 20 family inet address 172.16.20.1/24
set interfaces xe-0/1/0 unit 20 family inet6 address 2001:db8:0:20::1/64
set interfaces xe-0/1/0 unit 30 vlan-id 30
set interfaces xe-0/1/0 unit 30 description "Guest LAN"
set interfaces xe-0/1/0 unit 30 family inet address 172.16.30.1/24
set interfaces xe-0/1/0 unit 30 family inet6 address 2001:db8:0:30::1/64
show configuration interfaces xe-0/1/0
show interfaces terse xe-0/1/0
show interfaces terse xe-0/1/0.20
ping 172.16.20.21
```

### 14.6 IRB (VLAN Layer-3 Gateway)
```bash
set interfaces irb unit 10 family inet address 172.16.10.1/24
set interfaces irb unit 20 family inet address 172.16.20.1/24
set interfaces irb unit 30 family inet address 172.16.30.1/24
set vlans Corporate l3-interface irb.10
set vlans Server l3-interface irb.20
set vlans Guest l3-interface irb.30
show interfaces terse irb
show vlans Corporate detail
```

### 14.7 Interface Ranges (Bulk Switch Ports)
```bash
edit interfaces interface-range INTERFACES_VLAN_50
set member-range ge-0/0/10 to ge-0/0/12
set member-range ge-0/0/16 to ge-0/0/19
set unit 0 family ethernet-switching vlan members FINANCE
show vlans FINANCE
# member also supports wildcards/regex e.g. ge-2/1/*
```

### 14.8 LLDP for Trunks / Migration
```bash
set protocols lldp interface all
set protocols lldp interface xe-0/1/0
set protocols lldp interface xe-0/1/1
set protocols lldp interface xe-0/1/2
delete protocols lldp
show configuration protocols lldp | display set
# generic: set protocols <protocol> interface <interface>
# e.g. set protocols rstp interface xe-0/1/2
# e.g. set protocols isis interface xe-0/1/2
# e.g. set protocols mpls interface xe-0/1/3
```

---

## 15. Routing Instances & Virtualization

```bash
set routing-instances TEST_INSTANCE instance-type virtual-router
set routing-instances TEST_INSTANCE interface ge-0/0/2.0
set routing-instances TEST_INSTANCE interface ge-0/0/3.30
set routing-instances TEST_INSTANCE routing-options static route 0/0 next-hop 172.16.20.5
set routing-instances TEST_INSTANCE protocols ospf area 0 interface ge-0/0/2.0
set routing-instances TEST_INSTANCE protocols ospf area 0 interface ge-0/0/3.30
show configuration routing-instances
show route table TEST_INSTANCE.inet.0
show interfaces terse routing-instance TEST_INSTANCE
ping 172.16.20.5 routing-instance TEST_INSTANCE
```

---

## 16. Firewall Filters (Stateless) — Family `inet`

### 16.1 Create Terms (`from` = match, `then` = action)
```bash
set firewall family inet filter LAN_TO_WAN term BLOCK_GAMING from source-address 172.16.10.22/32
set firewall family inet filter LAN_TO_WAN term BLOCK_GAMING from source-address 172.16.10.44/32
set firewall family inet filter LAN_TO_WAN term BLOCK_GAMING from destination-address 198.51.100.48/32
set firewall family inet filter LAN_TO_WAN term BLOCK_GAMING from destination-port 666
set firewall family inet filter LAN_TO_WAN term BLOCK_GAMING then log
set firewall family inet filter LAN_TO_WAN term BLOCK_GAMING then discard
set firewall family inet filter LAN_TO_WAN term GUEST_COUNT_WEB then count COUNTER_GUEST_WEB
set firewall family inet filter LAN_TO_WAN term BLOCK_20_123 from source-address 172.16.20.123/32
set firewall family inet filter LAN_TO_WAN term BLOCK_20_123 then reject
set firewall family inet filter LAN_TO_WAN term ACCEPT_ALL_ELSE then accept
# port shortcuts: from destination-port http (=80), https (=443), ssh, telnet, ftp, ftp-data, domain, dhcp, ntp, bgp
set firewall family inet filter LAN_TO_WAN term TERM from source-port <port>
set firewall family inet filter LAN_TO_WAN term TERM from destination-port <port>
set firewall family inet filter LAN_TO_WAN term TERM from port <port>
set firewall family inet filter LAN_TO_WAN term TERM from protocol tcp
set firewall family inet filter LAN_TO_WAN term TERM from protocol udp
```

### 16.2 View & Test
```bash
show configuration firewall family inet filter LAN_TO_WAN
show configuration firewall family inet filter LAN_TO_WAN | display set
show configuration firewall family inet filter LAN_TO_WAN | display set relative
show configuration firewall
show firewall
show firewall filter <filter-name>
show firewall counter COUNTER_GUEST_WEB filter LAN_TO_WAN
show firewall log
show firewall log detail
show | compare
commit
commit and-quit
commit confirmed 2
ping 203.0.113.47 source 172.16.20.1
ping 203.0.113.47 source 172.16.30.1
ping <destination> source <source-IP>
```

### 16.3 Apply to Interface (Input vs Output)
```bash
set interfaces xe-0/1/6 unit 0 family inet filter output LAN_TO_WAN
# syntax: set interfaces <interface> unit <unit> family inet filter <input|output> <filter-name>
# input = entering interface, output = leaving interface
commit confirmed 2
```

### 16.4 Term Order
```bash
insert firewall family inet filter LAN_TO_WAN term BLOCK_20_123 before term ACCEPT_ALL_ELSE
insert firewall family inet filter LAN_TO_WAN term BLOCK_20_123 after term BLOCK_GUEST_PING
# top -> bottom, first terminating match stops. Specific before broad.
```

### 16.5 Reject Variants
```bash
# then reject sends ICMP unreachable; then discard drops silently; then accept allows
# TCP variant (platform-dependent):
# then reject tcp-reset
```

### 16.6 Control-Plane Protection (via lo0)
```bash
# apply input filter to lo0.0 (no IP required) to protect RE:
# set interfaces lo0 unit 0 family inet filter input <CONTROL-FILTER>
# on most EX/QFX, me0 needs separate filter (lo0.0 does not protect me0)
# on routers, lo0.0 filter can also protect fxp0
```

### 16.7 CoS / Policers Referenced by Filters
```bash
show class-of-service forwarding-class
# filter actions: then policer <name>, then forwarding-class <class>, then dscp <v>, then loss-priority <v>, then next-interface <if>, then next term, then count, then log, then syslog
```

---

## 17. Advanced Firewall — Prefix-Lists, Match Options, Policers

### 17.1 Prefix-Lists & Apply-Path (see also Routing Policy §19)
```bash
show configuration policy-options
show configuration firewall
show configuration interfaces
# concept: policy-options { prefix-list TRUSTED-HOSTS { 172.16.10.11/32; 172.16.10.12/32; } }
# concept: apply-path auto-populates prefix-list from config (e.g. BGP neighbors, interface subnets)
```

### 17.2 Advanced Match Concepts (no single CLI — use in `from`)
```text
tcp-flags "syn" / "syn no-ack"
ip-options / ip-options-except
is-fragment
protocol-except udp, destination-port-except, icmp-code-except
172.16.10.0/24 except 172.16.10.50/32
family inet / family inet6 / family ethernet-switching
```

---

## 18. Routing Policy (`policy-options`)

### 18.1 Create & Apply
```bash
set policy-options policy-statement EXPORT-STATIC term STATIC-DEFAULT from protocol static
set policy-options policy-statement EXPORT-STATIC term STATIC-DEFAULT from route-filter 0.0.0.0/0 exact
set policy-options policy-statement EXPORT-STATIC term STATIC-DEFAULT then accept
set protocols ospf export EXPORT-STATIC
set policy-options policy-statement EXPORT-STATIC-V6 term STATIC-DEFAULT from protocol static
set policy-options policy-statement EXPORT-STATIC-V6 term STATIC-DEFAULT from route-filter ::/0 exact
set policy-options policy-statement EXPORT-STATIC-V6 term STATIC-DEFAULT then accept
set protocols ospf3 export EXPORT-STATIC-V6
set policy-options policy-statement EXPORT-DEFAULT term DEFAULT from protocol static
set policy-options policy-statement EXPORT-DEFAULT term DEFAULT from route-filter 0.0.0.0/0 exact
set policy-options policy-statement EXPORT-DEFAULT term DEFAULT then accept
set protocols ospf export EXPORT-DEFAULT
set protocols ospf export [ POLICY-A POLICY-B POLICY-C ]
set policy-options defaults route-filter walkup
```

### 18.2 Match / Filter Options
```bash
# from: protocol, neighbor, tag, community, rib, interface, route-filter, prefix-list, prefix-list-filter
# from route-filter variants:
# from route-filter 172.16.0.0/16 exact
# from route-filter 172.16.0.0/16 orlonger
# from route-filter 172.16.0.0/16 longer
# from route-filter 172.16.0.0/16 upto /24
# from route-filter 172.16.0.0/16 prefix-length-range /20-/24
set policy-options prefix-list MY-PREFIXES 10.0.0.0/8
set policy-options prefix-list MY-PREFIXES 172.16.0.0/12
set policy-options prefix-list MY-PREFIXES 192.168.0.0/16
# from prefix-list MY-PREFIXES (exact)
# from prefix-list-filter MY-PREFIXES orlonger (allows exact/longer/orlonger)
```

### 18.3 Actions
```bash
# then metric 100
# then preference 50
# then tag 100
# then community <name>
# then accept / then reject / then next term / then next policy
```

### 18.4 Verify
```bash
show configuration policy-options
show configuration policy-options policy-statement <POLICY>
show | compare
show route
show route protocol ospf
show route 0.0.0.0/0
show route table inet6.0
show route ::/0
```

---

## 19. Syslog, Logging, SNMP, Traceoptions

### 19.1 Syslog Files
```bash
show log messages
show log interactive-commands
show log <filename>
show log ?
show log messages.1.gz
show log messages | match <pattern>
show log interactive-commands | match user | last 6
show configuration system
```

### 19.2 Configure Syslog
```bash
set system syslog file interactive-commands interactive-commands any
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file DAEMON_EVENTS.txt daemon info
set system syslog file DAEMON_EVENTS.txt archive files 3 size 1M world-readable
set system syslog host 172.16.20.5 any notice
set system syslog user * any emergency
show | compare
commit and-quit
```

### 19.3 Real-Time Logs & Decode
```bash
monitor start messages
monitor stop
help syslog RPD_OSPF_NBRDOWN
help syslog <message-name>
help apropos <keyword>
help apropos host-name
help topic <topic>
help topic ospf dead-interval
help reference <hierarchy>
help reference ospf area
```

### 19.4 SNMP
```bash
set snmp community MY_COMMUNITY authorization read-only
set snmp community MY_COMMUNITY clients 172.16.20.0/24
# trap-group: version v2, categories (authentication,chassis,configuration,link,routing,services), targets <NMS-IP>
show | compare
show snmp mib get 1.3.6.1.2.1.2.2.1.7
show snmp mib get <OID>
show snmp mib walk <OID>
show interfaces <interface> | match snmp
```

### 19.5 Traceoptions (see also §13.3)
```bash
set protocols ospf traceoptions file OSPF_TRACE.txt
set protocols ospf traceoptions flag error
set protocols ospf traceoptions flag event
show log OSPF_TRACE.txt
```

---

## 20. Connectivity Troubleshooting Toolkit

```bash
show interfaces terse
show route
show ospf neighbors
show ospf neighbor
show arp
show arp interface xe-0/1/6.0
show ipv6 neighbors
ping <IP>
ping 10.1.2.2 rapid
ping 10.1.2.2 rapid count 100
ping 10.1.2.2 size 1472 do-not-fragment count 1
ping 10.1.2.2 size 1473 do-not-fragment count 1
traceroute <IP>
ssh <user>@<IP>
monitor interface traffic
monitor interface <interface>
show interfaces extensive
clear interfaces statistics <int>
show route forwarding-table
show chassis routing-engine
```

---

## 21. Configuration Files, Load, Groups, Archival

### 21.1 Files
```bash
file list
file list /var/tmp/
file show FILENAME
file delete FILENAME
file compare files OSPF_START_CONFIG.txt SATURDAY_CHANGES.txt
file copy scp://user@172.16.40.10/lab/configs/BASE_CONFIG_AT_START.txt /var/tmp/
file copy /var/tmp/NEW_CONFIG.txt scp://lab@172.16.40.10/lab/configs/
file copy <SOURCE> <DESTINATION>
file copy <download-URL> /var/tmp/
show interfaces terse | save CURRENT_INTERFACE_STATUS.txt
show configuration | compare SATURDAY_CHANGES.txt
<command> | append <filename>
scp <image> <user>@<device>:/var/tmp/
```

### 21.2 Load (Bulk Config)
```bash
load factory-default
load override FILENAME
load override MY_START_CONFIG.txt
load override terminal
load merge ADDITIONAL_HIERARCHY_CONFIG.txt
load merge terminal
load merge terminal relative
load set terminal
# after load: show | compare -> commit check -> commit confirmed -> commit
show | compare
commit
commit check
commit confirmed
```

### 21.3 Configuration Groups & Inheritance
```bash
set groups MTU_9192_GROUP interfaces <x> mtu 9192
set interfaces apply-groups MTU_9192_GROUP
set interfaces xe-0/1/5 apply-groups-except MTU_9192_GROUP
show configuration interfaces xe-0/1/5 | display inheritance
show configuration interfaces xe-0/1/6 | display inheritance no-comments
show configuration interfaces xe-0/1/5 | display set | display inheritance
commit
```

### 21.4 Automatic Archival
```bash
set system archival configuration transfer-on-commit
set system archival configuration transfer-interval <interval>
set system archival configuration archive-sites "scp://user@server/path" password <password>
```

---

## 22. Upgrade Junos OS

```bash
show system storage
request system storage cleanup dry-run
request system storage cleanup
file copy <SOURCE> <DESTINATION>
file copy <download-URL> /var/tmp/
file list /var/tmp/
show version
show system information
request system software add /var/tmp/jinstall-ex-4300-21.4R3-S4.18-signed.tgz
request system software add <package> no-copy
request system software add <package> no-validate
request system software add <package> reboot
request system software add <package> no-copy no-validate reboot
request system software add ?
request vmhost software add /var/tmp/<package>
request system software rollback
request system reboot
# J-Web equivalents: Reboot If Required=reboot, Do not save backup=no-copy, no-validate=CLI only
```

---

## 23. Reboot / Power

```bash
request system reboot
# confirm: yes
```

---

## 24. Quick Memory — Must-Know One-Liners

```text
> = Operational, # = Config [edit]
configure / exit / commit / show | compare / rollback / rollback 1
commit check = validate only, commit confirmed = auto-rollback safety
Direct 0 < Static 5 < OSPF 10; LPM beats preference; * = active, > = best path
inet.0 = IPv4, inet6.0 = IPv6
inet = IPv4, inet6 = IPv6
0.0.0.0/0 = IPv4 default, ::/0 = IPv6 default
OSPFv2: protocols ospf / show ospf neighbor / show route protocol ospf
OSPFv3: protocols ospf3 / show ospf3 neighbor / show route protocol ospf3
Area 0 = backbone, passive = advertise without Hellos
VLAN: set vlans NAME vlan-id ID; access = 1 VLAN; trunk = many VLANs
MAC: show ethernet-switching table [vlan NAME]; D=dynamic, S=static, L=locally learned
IRB: set interfaces irb unit X + set vlans NAME l3-interface irb.X
lo0 = stable / management / Router ID; aeX = LAG (device-count, lacp active, 802.3ad)
Firewall filter = stateless (firewall), Security policy = stateful (security)
from = match, then = accept/discard/reject (terminating) + count/log/syslog/policer/forwarding-class/next-interface/next term (non-terminating)
Top->bottom, no match = implicit discard; insert ... before/after term ...
input = entering, output = leaving; lo0.0 input = control-plane protection
Routing policy: policy-options policy-statement; import = Protocol->RIB, export = RIB->Protocol
exact/orlonger/longer/upto/prefix-length-range; accept/reject/next term/next policy
SNMP: GET/SET/TRAP; v2c=community plaintext, v3=secure; MIB/OID/index
monitor interface traffic = live stats, monitor traffic = control-plane capture, show log = snapshot, monitor start = live log
help apropos = find command, help topic = learn concept, help reference = syntax
file copy / file list / load override (=replace) / load merge (=add) / load set terminal
groups: apply-groups / apply-groups-except + display inheritance
interface-range + member-range; wildcard delete; transfer-on-commit/transfer-interval
RADIUS=radius-server, TACACS+=tacplus-server, order=[radius tacplus password]
allow-* overrides deny-*; super-user > operator > read-only > unauthorized
OOB = fxp0/me0/em0; NTP: set system ntp server + show ntp associations (*=synced)
DNS: set system name-server; J-Web: http vs https system-generated-certificate
New device: root/Amnesiac -> cli -> set system root-authentication plain-text-password -> hostname/user/NTP/DNS/SSH -> rescue save
Upgrade: show system storage -> file copy /var/tmp/ -> show version -> request system software add -> reboot -> show version
XML: show ... | display xml; CLI->XML API->mgd->Junos; RPC/rpc-reply; NETCONF=RPC+XML; PyEZ=Python lib; REST=HTTP
```

