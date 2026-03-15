Current Date and Time (UTC - YYYY-MM-DD HH:MM:SS formatted): 2026-03-15 19:31:02
Current User's Login: vipin0998


GNS3 eBGP Load Balancing Lab – Complete Project Explanation
Project Overview
A comprehensive Cisco network simulation demonstrating eBGP load balancing with AS Path Prepending using 5 routers across two autonomous systems, teaching advanced BGP traffic engineering and redundancy.

Lab Topology
Code
┌─────────────────────────────────────┐
│       AS 123 (Internal)             │
│                                     │
│  R1 ─────┬────── R2                │
│ (Core)   │    (Edge-1)             │
│          │                         │
│          └────── R3                │
│         (Edge-2/Redundant)         │
└─────────────────────────────────────┘
      ↓            ↓
    eBGP          eBGP
   (Direct)    (Multihop)
      ↓            ↓
┌────────────���────────────────────────┐
│       AS 45 (External)              │
│                                     │
│  R4 ────────────── R5              │
│ (Edge-1)        (Edge-2)           │
│                                     │
└─────────────────────────────────────┘
Router Configurations Summary
🔴 R1 – Core Transit Router (AS 123)
Aspect	Details
Role	Central hub aggregating all internal routes
Loopback	1.1.1.1/32, 100.0.0.1/32 (test prefix)
Interfaces	Fa1/0 (R1-R2), Fa1/1 (R1-R3)
Protocols	OSPF 123 (Area 0), EIGRP 100, BGP 123
iBGP Peers	R2 (2.2.2.2), R3 (3.3.3.3)
Function	Redistribution hub between EIGRP/OSPF/BGP
Key Config:

cisco
router ospf 123
  network 100.0.0.0 0.0.0.0 area 0
  redistribute eigrp 100 subnets

router bgp 123
  neighbor 2.2.2.2 remote-as 123 (iBGP)
  neighbor 3.3.3.3 remote-as 123 (iBGP)
🟠 R2 – Primary eBGP Gateway (AS 123)
Aspect	Details
Role	Primary external connection to AS 45
Loopback	2.2.2.2/32
Interfaces	Fa1/0 (backbone to R1), Fa0/0 (eBGP to R4)
Protocols	OSPF 123 (Area 0 & 1), EIGRP 100, BGP 123
eBGP Peer	R4 (24.0.0.4) in AS 45 – Direct link
Function	Primary path for AS 123 ↔ AS 45 exchange
Key Config:

cisco
router bgp 123
  neighbor 24.0.0.4 remote-as 45 (eBGP - Direct)
  update-source Loopback2
🟡 R3 – Secondary eBGP Gateway (AS 123)
Aspect	Details
Role	Redundant external connection with multihop eBGP
Loopback	3.3.3.3/32
Interfaces	Fa1/1 (backbone to R1), Fa0/0 (multihop eBGP to R5)
Protocols	OSPF 123 (Area 0 & 1), EIGRP 100, BGP 123
eBGP Peer	R5 (35.0.0.5) in AS 45 – Multihop via Loopback
iBGP Peers	R1, R2 (route propagation)
Function	Backup path + load balancing control point
Key Config:

cisco
router bgp 123
  neighbor 35.0.0.5 remote-as 45 (eBGP - Multihop)
  neighbor 35.0.0.5 ebgp-multihop 2
  neighbor 1.1.1.1 remote-as 123 (iBGP)
  neighbor 2.2.2.2 remote-as 123 (iBGP)
🔵 R4 – Primary External Edge (AS 45)
Aspect	Details
Role	External network receiving routes from AS 123
Loopback	4.4.4.4/32
Interfaces	Fa0/0 (eBGP to R2), Fa1/0 (backbone to R5)
Protocols	OSPF 45, EIGRP 100 (dormant), BGP 45
eBGP Peer	R2 (24.0.0.2) in AS 123 – Direct link
Function	Primary receiver; prefers shorter AS paths
Key Config:

cisco
router bgp 45
  neighbor 24.0.0.2 remote-as 123 (eBGP - Direct)
🟣 R5 – Secondary External Edge WITH Load Balancing (AS 45)
Aspect	Details
Role	Redundant external connection + AS Path Prepending
Loopback	5.5.5.5/32
Interfaces	Fa0/0 (multihop eBGP to R3), Fa1/0 (backbone to R4)
Protocols	OSPF 45, EIGRP 100, BGP 45
eBGP Peer	R3 (35.0.0.3) in AS 123 – Multihop via Loopback
iBGP Peer	R4 (4.4.4.4) for route propagation
🎯 Load Balancing	Prepends AS 45 three times (45 45 45)
Function	Backup path + traffic engineering control
Key Config:

cisco
route-map vipin permit 10
  set as-path prepend 45 45 45

router bgp 45
  neighbor 35.0.0.3 remote-as 123 (eBGP - Multihop)
  neighbor 35.0.0.3 route-map vipin out  ← APPLIES PREPENDING
  neighbor 4.4.4.4 remote-as 45 (iBGP)
How eBGP Load Balancing Works
Scenario: AS 45 accessing prefix 100.0.0.0/24 from AS 123
Code
ROUTE OPTIONS:

Path 1 (Primary): R4 ← R2 ← R1
  AS_PATH = 123
  Status: ✓ PREFERRED (shortest)
  Traffic: ~80%

Path 2 (Backup):  R5 ← R3 ← R1
  AS_PATH = 123 45 45 45 (prepended by R5)
  Status: ✗ LESS PREFERRED (longer due to prepending)
  Traffic: ~20%
Why R5 Prepends?
R5 receives routes from R3 with AS_PATH: 123
R5 applies route-map to ADD (prepend) its own AS number 3 times
When R5 advertises to R3, R3 sees AS_PATH: 123 45 45 45
R3 propagates this to R1 and R2
External routers prefer shorter paths → traffic favors R2-R4 link
Routing Protocol Stack
Protocol	R1	R2	R3	R4	R5
OSPF	✅ Area 0	✅ Area 0,1	✅ Area 0,1	✅ Area 0,1	✅ Area 0,1
EIGRP	✅ AS 100	✅ AS 100	✅ AS 100	✅ AS 100	✅ AS 100
BGP	✅ AS 123 (Hub)	✅ AS 123 (Edge)	✅ AS 123 (Edge)	✅ AS 45	✅ AS 45 + Prepending
Traffic Engineering Features
Feature	Status	Implementation
Multi-Protocol Routing	✅	OSPF, EIGRP, BGP integrated
Route Redistribution	✅	EIGRP ↔ OSPF ↔ BGP
iBGP Mesh	✅	R1 peers with R2, R3; R5 peers with R4
eBGP Multihop	✅	R3-R5 and R3-R5 use multihop (loopback)
AS Path Prepending	✅	R5 prepends 45 45 45 via route-map "vipin"
Load Balancing	✅	Unequal split (80/20) via AS path manipulation
Redundancy	✅	Dual paths: R2-R4 (primary), R3-R5 (backup)
Key Learning Points
Core vs Edge Design: R1 is central hub; R2, R3, R4, R5 are edge routers
AS Path as Control Tool: AS path prepending influences external route selection
iBGP Propagation: Internal routes propagated within AS via iBGP
Multihop eBGP: Enables peering across non-adjacent routers using Loopback addresses
Unequal Load Balancing: Traffic split via BGP attributes, not just ECMP
Summary
Component	Details
Lab Type	eBGP Load Balancing with AS Path Prepending
Total Routers	5 (3 internal, 2 external)
Autonomous Systems	2 (AS 123, AS 45)
Primary Mechanism	AS path prepending on R5 (45 45 45)
Load Balance Ratio	~80% via R2-R4, ~20% via R3-R5
Protocols Used	OSPF (3 areas), EIGRP (1 AS), BGP (iBGP + eBGP)
Key Feature	Route-map "vipin" on R5 for traffic engineering
Use Case	Educational BGP traffic engineering lab
🎯 Objective Achieved: Demonstrates how network engineers use BGP attributes (specifically AS path prepending) to control inbound traffic distribution across multiple external links, achieving intelligent load balancing without equal-cost multipath (ECMP).
