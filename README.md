# BGP Lab from Scratch — EVE-NG 🌐

> **Companion repository for the [Network ThinkTank](https://networkthinktank.blog) blog post:**
> *"BGP Lab from Scratch (EVE-NG) — A Complete Step-by-Step Guide"*

---

## 📋 Overview

This lab demonstrates a fully functional **eBGP and iBGP** environment built inside **EVE-NG Community Edition**. You will configure four Cisco IOS routers across two autonomous systems (AS), establish BGP peering, advertise networks, and verify full reachability.

**What you will learn:**
- How to build a BGP topology from scratch in EVE-NG
- Configuring eBGP between two Autonomous Systems
- Configuring iBGP within a single AS
- Advertising networks via BGP
- Verifying BGP neighbors, table, and routes
- Troubleshooting common BGP issues

---

## 🗺️ Topology

```
         AS 65001                          AS 65002
  ┌──────────────────┐            ┌──────────────────┐
  │                  │            │                  │
  │  R1 (10.0.0.1)───┼────eBGP────┼───R3 (10.0.0.2)  │
  │       │          │            │        │         │
  │  iBGP │          │            │   iBGP │         │
  │       │          │            │        │         │
  │  R2 (10.0.1.1)   │            │   R4 (10.0.1.2)  │
  │  LAN: 192.168.1.0/24          │   LAN: 192.168.2.0/24
  └──────────────────┘            └──────────────────┘
```

### IP Addressing Table

| Device | Interface   | IP Address       | Description           |
|--------|-------------|------------------|-----------------------|
| R1     | e0/0        | 10.0.0.1/30      | Link to R3 (eBGP)     |
| R1     | e0/1        | 172.16.0.1/30    | Link to R2 (iBGP)     |
| R1     | Loopback0   | 1.1.1.1/32       | BGP Router-ID         |
| R2     | e0/0        | 172.16.0.2/30    | Link to R1 (iBGP)     |
| R2     | e0/1        | 192.168.1.1/24   | LAN (advertised)      |
| R2     | Loopback0   | 2.2.2.2/32       | BGP Router-ID         |
| R3     | e0/0        | 10.0.0.2/30      | Link to R1 (eBGP)     |
| R3     | e0/1        | 172.16.1.1/30    | Link to R4 (iBGP)     |
| R3     | Loopback0   | 3.3.3.3/32       | BGP Router-ID         |
| R4     | e0/0        | 172.16.1.2/30    | Link to R3 (iBGP)     |
| R4     | e0/1        | 192.168.2.1/24   | LAN (advertised)      |
| R4     | Loopback0   | 4.4.4.4/32       | BGP Router-ID         |

---

## 🛠️ Prerequisites

- **EVE-NG Community Edition** installed (bare metal or VMware/VirtualBox)
- **Cisco IOS image** (IOL or vIOS — c7200, IOSv all work)
- Basic knowledge of IP addressing and routing concepts
- SSH client (PuTTY, SecureCRT, or built-in EVE-NG web console)

---

## 📁 Repository Structure

```
bgp-lab-eve-ng/
├── README.md                      ← Full lab guide (you are here)
├── configs/
│   ├── R1-AS65001.txt             ← R1 full running config
│   ├── R2-AS65001.txt             ← R2 full running config
│   ├── R3-AS65002.txt             ← R3 full running config
│   └── R4-AS65002.txt             ← R4 full running config
├── verification/
│   └── verification-commands.txt  ← Show commands & expected outputs
└── topology/
    └── topology-notes.md          ← EVE-NG node setup notes
```

---

## 🚀 Step-by-Step Lab Guide

### Step 1 — Build the Topology in EVE-NG

1. Open your EVE-NG web UI at http://<eve-ng-ip>
2. Create a new lab: **Add New Lab** and name it BGP-Lab
3. Add 4 Cisco routers (IOSv or IOL image)
4. Connect them according to the topology:
   - R1 e0/0 to R3 e0/0 (eBGP link)
   - R1 e0/1 to R2 e0/0 (iBGP link, AS 65001)
   - R3 e0/1 to R4 e0/0 (iBGP link, AS 65002)
5. Start all nodes and connect via console

---

### Step 2 — Configure R1 (AS 65001 — eBGP + iBGP Hub)

```
hostname R1
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface Ethernet0/0
 description Link-to-R3-eBGP
 ip address 10.0.0.1 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 description Link-to-R2-iBGP
 ip address 172.16.0.1 255.255.255.252
 no shutdown
!
router bgp 65001
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 neighbor 10.0.0.2 remote-as 65002
 neighbor 10.0.0.2 description eBGP-to-R3
 neighbor 172.16.0.2 remote-as 65001
 neighbor 172.16.0.2 description iBGP-to-R2
 neighbor 172.16.0.2 next-hop-self
!
```

---

### Step 3 — Configure R2 (AS 65001 — LAN Advertiser)

```
hostname R2
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
interface Ethernet0/0
 description Link-to-R1-iBGP
 ip address 172.16.0.2 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 description LAN-192.168.1.0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
router bgp 65001
 bgp router-id 2.2.2.2
 bgp log-neighbor-changes
 neighbor 172.16.0.1 remote-as 65001
 neighbor 172.16.0.1 description iBGP-to-R1
 network 192.168.1.0 mask 255.255.255.0
!
```

---

### Step 4 — Configure R3 (AS 65002 — eBGP + iBGP Hub)

```
hostname R3
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
interface Ethernet0/0
 description Link-to-R1-eBGP
 ip address 10.0.0.2 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 description Link-to-R4-iBGP
 ip address 172.16.1.1 255.255.255.252
 no shutdown
!
router bgp 65002
 bgp router-id 3.3.3.3
 bgp log-neighbor-changes
 neighbor 10.0.0.1 remote-as 65001
 neighbor 10.0.0.1 description eBGP-to-R1
 neighbor 172.16.1.2 remote-as 65002
 neighbor 172.16.1.2 description iBGP-to-R4
 neighbor 172.16.1.2 next-hop-self
!
```

---

### Step 5 — Configure R4 (AS 65002 — LAN Advertiser)

```
hostname R4
!
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
!
interface Ethernet0/0
 description Link-to-R3-iBGP
 ip address 172.16.1.2 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 description LAN-192.168.2.0
 ip address 192.168.2.1 255.255.255.0
 no shutdown
!
router bgp 65002
 bgp router-id 4.4.4.4
 bgp log-neighbor-changes
 neighbor 172.16.1.1 remote-as 65002
 neighbor 172.16.1.1 description iBGP-to-R3
 network 192.168.2.0 mask 255.255.255.0
!
```

---

## Verification Commands

Run these commands to verify the lab is working:

    show ip bgp summary          -- Check all BGP neighbor states (Established = good)
    show ip bgp                  -- View the full BGP table
    show ip bgp neighbors        -- Detailed BGP neighbor info
    show ip route bgp            -- Check BGP routes in routing table
    ping 192.168.2.1 source 192.168.1.1   -- End-to-end connectivity test

### Expected BGP Summary on R1

    R1# show ip bgp summary
    BGP router identifier 1.1.1.1, local AS number 65001
    Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.0.0.2        4 65002      15      15        3    0    0 00:08:32        1
    172.16.0.2      4 65001      12      12        3    0    0 00:06:14        1

---

## Troubleshooting Guide

| Issue | Likely Cause | Fix |
|-------|-------------|-----|
| BGP stuck in Active state | No TCP connectivity to peer | Check IPs, no shutdown, ping peer |
| BGP stuck in Idle | Wrong remote-as | Verify neighbor remote-as matches peer AS |
| Routes missing from BGP table | Network not in routing table | Add static route or check network statement |
| iBGP routes not forwarding | Next-hop unreachable | Add next-hop-self on iBGP speaker |
| BGP session drops | Holdtime expired | Check MTU, ACLs, use neighbor timers |

---

## Key BGP Concepts

- **eBGP** — BGP between different Autonomous Systems (AD=20)
- **iBGP** — BGP within the same AS (AD=200, requires full mesh or route reflector)
- **BGP Router-ID** — Highest loopback or manual config; uniquely identifies router
- **next-hop-self** — Forces iBGP peers to use local router as BGP next-hop
- **network command** — Exact match must exist in routing table to advertise
- **BGP States**: Idle → Connect → Active → OpenSent → OpenConfirm → Established

---

## Related Resources

- Blog Post: https://networkthinktank.blog
- EVE-NG Docs: https://www.eve-ng.net/index.php/documentation/
- Cisco BGP Guide: https://www.cisco.com/c/en/us/tech/ip/ip-routing-border-gateway-protocol-bgp/index.html

---

## Author

Network ThinkTank | https://networkthinktank.blog
Practical networking labs for engineers at every level.

If this lab helped you, please star the repo!
