# CCNA Networking Lab

A multi-device enterprise networking lab built in the **PingMyNetwork.com sandbox** to practise configuring and troubleshooting OSPF, DHCP, DNS, Syslog, inter-VLAN routing, and EtherChannel together.

## Table of Contents

1. [About the Lab](#about-the-lab)
2. [Topology](#topology)
3. [Address Planning](#address-planning)
4. [Server Segment Configuration](#server-segment-configuration)
5. [OSPF Configuration](#ospf-configuration)
6. [Switch Configuration](#switch-configuration)
7. [Syslog Configuration](#syslog-configuration)

---

## About the Lab

| Topic | Description |
|---|---|
| **OSPF** | Single-area OSPF (Area 0) over point-to-point links and a multi-segment Ethernet network |
| **DHCP** | Dynamic IPv4 address assignment for clients |
| **DNS** | Name resolution for network services and hosts |
| **Syslog** | Centralized collection of device log messages for monitoring and troubleshooting |
| **Inter-VLAN Routing** | Communication between VLANs using router subinterfaces (router-on-a-stick) |
| **EtherChannel (PAgP)** | Bundling multiple physical switch links into one logical link with the Port Aggregation Protocol |

The aim is to practise how these services and technologies work **together** in a network, rather than configuring each topic in isolation.

---

## Topology

<img width="893" height="605" alt="CCNA lab topology" src="https://github.com/user-attachments/assets/9ffc56d7-a8f9-4e52-ae20-543cd205084c" />

- **Core:** six routers (R1–R6) running single-area OSPF in Area 0, connected by point-to-point and multi-segment networks.
- **Server segment:** DHCP, DNS, web, and Syslog services behind a server switch, with R7 as gateway.
- **Access layer:** VLAN 10 and VLAN 20 with inter-VLAN routing on R5.
- **EtherChannel:** PAgP bundles links between SW2 and SW3.

---

## Address Planning

### Server Addressing

**Network:** `10.0.4.0/24`

| Server | IP Address | Subnet Mask | Default Gateway | DNS Server |
|---|---|---|---|---|
| DHCP (SRV1) | `10.0.4.2` | `255.255.255.0` | `10.0.4.1` | — |
| DNS (SRV2) | `10.0.4.3` | `255.255.255.0` | `10.0.4.1` | — |
| Web (SRV3) | `10.0.4.4` | `255.255.255.0` | `10.0.4.1` | `10.0.4.3` |
| Syslog (SRV4) | `10.0.4.5` | `255.255.255.0` | `10.0.4.1` | — |

### R7 – Server Network Gateway

| Interface | IP Address | Subnet Mask | Purpose |
|---|---|---|---|
| GigabitEthernet0/0 | `203.0.113.2` | `255.255.255.252` | WAN link to R6 |
| GigabitEthernet0/1 | `10.0.4.1` | `255.255.255.0` | Server LAN gateway |

**Default route:** `0.0.0.0/0` via `203.0.113.1`

R7 is the default gateway for `10.0.4.0/24` and forwards upstream traffic to R6 with a static default route.

### OSPF Area 0 – Router Addressing

| Router | Interface | IP Address | Subnet Mask | Network |
|---|---|---|---|---|
| R1 | G0/0 | `10.0.0.1` | `255.255.255.0` | `10.0.0.0/24` |
| R1 | G0/1 | `10.0.3.2` | `255.255.255.0` | `10.0.3.0/24` |
| R1 | Loopback0 | `1.1.1.1` | — | Router ID |
| R2 | G0/0 | `10.0.0.2` | `255.255.255.0` | `10.0.0.0/24` |
| R2 | Loopback0 | `2.2.2.2` | — | Router ID |
| R3 | G0/0 | `10.0.0.3` | `255.255.255.0` | `10.0.0.0/24` |
| R3 | Loopback0 | `3.3.3.3` | — | Router ID |
| R4 | G0/0 | `10.0.0.4` | `255.255.255.0` | `10.0.0.0/24` |
| R4 | G0/1 | `10.0.2.1` | `255.255.255.0` | `10.0.2.0/24` |
| R4 | Loopback0 | `4.4.4.4` | — | Router ID |
| R5 | G0/0 | `10.0.2.2` | `255.255.255.0` | `10.0.2.0/24` |
| R5 | G0/1.10 | `192.168.10.10` | `255.255.255.0` | VLAN 10 |
| R5 | G0/1.20 | `192.168.20.10` | `255.255.255.0` | VLAN 20 |
| R5 | G0/1.99 | `192.168.99.2` | `255.255.255.0` | VLAN 99 |
| R5 | Loopback0 | `5.5.5.5` | — | Router ID |
| R6 | G0/0 | `203.0.113.1` | `255.255.255.252` | `203.0.113.0/30` |
| R6 | G0/1 | `10.0.3.1` | `255.255.255.0` | `10.0.3.0/24` |
| R6 | Loopback0 | `6.6.6.6` | — | Router ID |

> **Note:** R7 is outside OSPF Area 0 and connects to R6 over `203.0.113.0/30`. R7 uses `203.0.113.1` as its default-route next hop; R6 uses `203.0.113.2`.

### PC Address Assignment

PCs in VLAN 10 (`192.168.10.0/24`) and VLAN 20 (`192.168.20.0/24`) get their IPv4 settings (address, mask, gateway, DNS) automatically from the DHCP server. R5 provides inter-VLAN routing via router-on-a-stick subinterfaces.

---

## Server Segment Configuration

Lab services on `10.0.4.0/24`.

### Overview

| Service | IP Address | Status |
|---|---|---|
| DHCP | `10.0.4.2` | ✅ On |
| DNS | `10.0.4.3` | ✅ On |
| HTTP | `10.0.4.4` | ✅ On |
| Syslog | `10.0.4.5` | ✅ On |

### DHCP

| Pool | Gateway | DNS Server | Start IP | Subnet Mask | Max Users |
|---|---|---|---|---|---:|
| `VLAN-10` | `192.168.10.10` | `10.0.4.3` | `192.168.10.1` | `255.255.255.0` | 100 |
| `VLAN-20` | `192.168.20.10` | `10.0.4.3` | `192.168.20.1` | `255.255.255.0` | 100 |

### DNS

| Name | Type | Address |
|---|---|---|
| `www.google.com` | A | `10.0.4.4` |

### HTTP

| Setting | Value |
|---|---|
| File | `index.html` |
| Title | `SRV3` |

### Syslog

Example log received from R6 (`203.0.113.1`):

```text
%OSPF-5-ADJCHG: Process 6, Nbr 1.1.1.1 on GigabitEthernet0/1 from LOADING to FULL
```

Router command to send logs:

```text
logging host 10.0.4.5
```

---

## OSPF Configuration

Single-area OSPFv2 (Area 0) across R1–R6. R6 is the edge router and injects a default route toward the upstream next hop.

### Overview

| Router | Process ID | Loopback (Router ID) | Role |
|---|---:|---|---|
| R1 | 1 | `1.1.1.1/32` | Core, links `10.0.0.0/24` and `10.0.3.0/24` |
| R2 | 2 | `2.2.2.2/32` | Core, `10.0.0.0/24` |
| R3 | 3 | `3.3.3.3/32` | Core, `10.0.0.0/24` |
| R4 | 4 | `4.4.4.4/32` | Core, links `10.0.0.0/24` and `10.0.2.0/24` |
| R5 | 5 | `5.5.5.5/32` | Distribution to LANs (VLAN 10/20/90) |
| R6 | 6 | `6.6.6.6/32` | Edge / ASBR, default route originator |

> OSPF process IDs are locally significant and need not match between neighbors. Here each equals the router number for readability.

### Networks Advertised

| Network | Wildcard | Advertised By | Purpose |
|---|---|---|---|
| `1.1.1.1` – `6.6.6.6` | `0.0.0.0` | Each router (own loopback) | Loopback / stable router ID |
| `10.0.0.0/24` | `0.0.0.255` | R1, R2, R3, R4 | Shared core segment |
| `10.0.2.0/24` | `0.0.0.255` | R4, R5 | R4–R5 link |
| `10.0.3.0/24` | `0.0.0.255` | R1, R6 | R1–R6 link |
| `192.168.10.0/24` | `0.0.0.255` | R5 | VLAN 10 |
| `192.168.20.0/24` | `0.0.0.255` | R5 | VLAN 20 |
| `192.168.90.0/24` | `0.0.0.255` | R5 | Management network (syslog) |

### Passive Interfaces

| Router | Passive Interfaces | Reason |
|---|---|---|
| R1–R5 | `l0` | No neighbors on loopbacks |
| R5 | `g0/1` | LAN-facing, no OSPF neighbors expected |
| R6 | `g0/0`, `l0` | `g0/0` faces the upstream link; loopback needs no neighbors |

### Default Route

R6 holds a static default route and advertises it into OSPF:

```text
ip route 0.0.0.0 0.0.0.0 203.0.113.2
router ospf 6
 default-information originate
```

All other routers learn `0.0.0.0/0` as an **O\*E2** route.

### Router Configurations

<details>
<summary><strong>R6</strong> (edge / ASBR)</summary>

```text
router ospf 6
 network 6.6.6.6 0.0.0.0 area 0
 network 10.0.3.0 0.0.0.255 area 0
 passive-interface g0/0
 passive-interface l0
 default-information originate
!
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```
</details>

<details>
<summary><strong>R1</strong></summary>

```text
router ospf 1
 network 1.1.1.1 0.0.0.0 area 0
 network 10.0.0.0 0.0.0.255 area 0
 network 10.0.3.0 0.0.0.255 area 0
 passive-interface l0
```
</details>

<details>
<summary><strong>R2</strong></summary>

```text
router ospf 2
 network 2.2.2.2 0.0.0.0 area 0
 network 10.0.0.0 0.0.0.255 area 0
 passive-interface l0
```
</details>

<details>
<summary><strong>R3</strong></summary>

```text
router ospf 3
 network 3.3.3.3 0.0.0.0 area 0
 network 10.0.0.0 0.0.0.255 area 0
 passive-interface l0
```
</details>

<details>
<summary><strong>R4</strong></summary>

```text
router ospf 4
 network 4.4.4.4 0.0.0.0 area 0
 network 10.0.0.0 0.0.0.255 area 0
 network 10.0.2.0 0.0.0.255 area 0
 passive-interface l0
```

> The original line was `network 10.0.0.4 0.0.0.255`. It works because the wildcard matches the whole `/24`, but `10.0.0.0` is the correct, clearer form.
</details>

<details>
<summary><strong>R5</strong></summary>

```text
router ospf 5
 network 5.5.5.5 0.0.0.0 area 0
 network 10.0.2.0 0.0.0.255 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.90.0 0.0.0.255 area 0
 passive-interface l0
 passive-interface g0/1
```

> The `192.168.90.0/24` statement is for the management network used to send syslog. *(Details to be added.)*
</details>

### Verification

| Command | What to Check |
|---|---|
| `show ip ospf neighbor` | All adjacencies in FULL state |
| `show ip route ospf` | Loopbacks, `10.0.x.x` and `192.168.x.x` learned; `O*E2 0.0.0.0/0` present |
| `show ip protocols` | Passive interfaces and advertised networks |
| `show ip ospf interface brief` | Interfaces in Area 0, correct cost/state |
| `ping 6.6.6.6 source 5.5.5.5` | End-to-end reachability between loopbacks |

---

## Switch Configuration

### SW2

**Access VLANs**

```text
interface range f0/1-4
 switchport mode access
!
interface range f0/1-2
 switchport access vlan 20
!
interface range f0/3-4
 switchport access vlan 10
```

**EtherChannel (PAgP)**

```text
interface range f0/5-7
 channel-group 1 mode desirable
!
interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

### SW3

**Access VLANs**

```text
interface range f0/1-2
 switchport mode access
 switchport access vlan 10
```

**EtherChannel (PAgP)**

```text
interface range f0/5-7
 channel-group 1 mode desirable
!
interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10
```

---

## Syslog Configuration

### All Network Devices

Send log messages to the centralized Syslog server (`10.0.4.5`):

```text
logging host 10.0.4.5
logging traps 7
```

### Switch Management (SW2 and SW3)

Both switches need a management IP and default gateway to reach the Syslog server.

**SW2**

```text
vlan 99
!
interface vlan 99
 ip address 192.168.99.1 255.255.255.0
!
ip default-gateway 192.168.99.2
!
interface port-channel 1
 switchport trunk allowed vlan add 99
```

**SW3**

```text
vlan 99
!
interface vlan 99
 ip address 192.168.99.1 255.255.255.0
!
ip default-gateway 192.168.99.3
!
interface port-channel 1
 switchport trunk allowed vlan add 99
```

### R5 – Inter-VLAN Routing and Syslog

R5 uses router-on-a-stick subinterfaces to route between VLANs. The VLAN 99 management subinterface forwards Syslog messages from SW2 and SW3 toward the Syslog server (`10.0.4.5`).

### R5 – DHCP Relay

Configure `ip helper-address` on the LAN-facing interface to forward DHCP Discover messages to the DHCP server (`10.0.4.2`):

```text
interface g0/1
 ip helper-address 10.0.4.2
```

This lets clients in different VLANs obtain addresses from the centralized DHCP server.
