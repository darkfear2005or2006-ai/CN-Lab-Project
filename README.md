# Multi‑Site Enterprise Network with VLANs, DHCP, DNS, OSPF, NAT & ACLs

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-blue) ![Network](https://img.shields.io/badge/Network-Enterprise-green) ![OSPF](https://img.shields.io/badge/Routing-OSPF-orange)

A **Cisco Packet Tracer** project that designs and implements a fully functional **multi‑site enterprise network** for a fictional company.  
It integrates **VLAN segmentation**, **DHCP**, **DNS**, **OSPF dynamic routing**, **NAT**, and **ACL‑based security** into a single, real‑world topology.

---

## 📁 Project Overview

This network simulates two branch offices (Site A and Site B) connected via a serial WAN link. Each site has three departments – HR, IT, and Sales – separated into VLANs. Core services such as automatic IP assignment, name resolution, dynamic routing, Internet access, and access control are configured and verified end‑to‑end.

The project was built as part of the **CSNC‑2413 Computer Communications & Networks** course.

---

## 🚀 Features

- **VLAN Segmentation** – Isolates HR (10), IT (20), Sales (30), and Servers (99) traffic using IEEE 802.1Q trunks.
- **Router‑on‑a‑Stick** – Inter‑VLAN routing via subinterfaces on Cisco 2911 routers.
- **DHCP** – Routers dynamically assign IP addresses to clients in each VLAN.
- **DNS** – A local DNS server resolves internal names (`server.sales.local`, `www.isp.com`).
- **OSPF (Area 0)** – Dynamically routes between the two sites and propagates a default route to the Internet.
- **NAT (PAT)** – Overload NAT on the Site A edge router allows internal hosts to access the simulated Internet.
- **ACL Security** – Extended ACL blocks IT department (VLAN 20) from accessing Sales (VLAN 30).
- **Verification** – DHCP leases, OSPF neighbors, ping, DNS resolution, and Internet connectivity tested.

---

## 🧱 Network Topology
PC‑A‑HR ──┐
PC‑A‑IT ──┤
Switch‑A (3560) ── Trunk (Gi0/1) ── R‑A (Gi0/0)
DNS‑Srv ──┘ (VLAN 99)

R‑A Gi0/1 ── ISP Gi0/1 (WAN)
ISP Gi0/0 ── ISP‑Web Server

R‑A Se0/3/0 ── R‑B Se0/3/0 (Serial WAN)

PC‑B‑HR ──┐
PC‑B‑IT ──┤ Switch‑B (3560) ── Trunk (Gi0/1) ── R‑B (Gi0/0)


## 🧰 Devices & IP Addressing

| Device | Interface | IP Address | Subnet Mask | VLAN |
|--------|-----------|------------|-------------|------|
| **R‑A** | Gi0/0.10 | 172.16.10.1 | /24 | 10 |
|  | Gi0/0.20 | 172.16.20.1 | /24 | 20 |
|  | Gi0/0.30 | 172.16.30.1 | /24 | 30 |
|  | Gi0/0.99 | 172.16.99.1 | /24 | 99 |
|  | Gi0/1 | 203.0.114.2 | /30 | – |
|  | Se0/3/0 | 192.168.1.1 | /30 | – |
| **R‑B** | Gi0/0.10 | 172.16.40.1 | /24 | 10 |
|  | Gi0/0.20 | 172.16.50.1 | /24 | 20 |
|  | Gi0/0.30 | 172.16.60.1 | /24 | 30 |
|  | Se0/3/0 | 192.168.1.2 | /30 | – |
| **ISP** | Gi0/0 | 203.0.113.1 | /24 | – |
|  | Gi0/1 | 203.0.114.1 | /30 | – |
| **DNS‑Server** | Fa0 | 172.16.99.10 | /24 | 99 |
| **ISP‑Web** | Fa0 | 203.0.113.10 | /24 | – |

*All PCs receive IPs via DHCP from their respective site routers.*

---

## ⚙️ Configuration Highlights

### VLAN & Trunking (Switch‑A)
vlan 10
name HR
vlan 20
name IT
vlan 30
name Sales
vlan 99
name Servers
interface Gi0/1
switchport mode trunk

### Router‑on‑a‑Stick (R‑A)
interface Gi0/0.10
encapsulation dot1Q 10
ip address 172.16.10.1 255.255.255.0

### DHCP Pool (R‑A)
ip dhcp pool HR_A
network 172.16.10.0 255.255.255.0
default-router 172.16.10.1
dns-server 172.16.99.10

### OSPF (R‑A & R‑B)
router ospf 1
network 172.16.10.0 0.0.0.255 area 0
network 192.168.1.0 0.0.0.3 area 0
default-information originate

### NAT Overload (R‑A)
ip nat inside source list 1 interface Gi0/1 overload
access-list 1 permit 172.16.0.0 0.0.255.255

### ACL Block IT → Sales
access-list 100 deny ip 172.16.20.0 0.0.0.255 172.16.30.0 0.0.0.255
access-list 100 permit ip any any
interface Gi0/0.30
ip access-group 100 out

## ▶️ How to Run

1. Install **Cisco Packet Tracer** (version 7.0 or later).
2. Clone this repository:
3.  git clone https://github.com/darkfear2005or2006-ai/enterprise-network-project.git
Open the .pkt file with Packet Tracer.

Wait for the network to converge (OSPF should form adjacencies within a minute).

Use the verification commands below to test connectivity.

### ✅ Verification
Test	Command	Expected
OSPF Neighbor	show ip ospf neighbor	Neighbor FULL
DHCP Lease	ipconfig /renew on any PC	IP in correct VLAN range
Inter‑VLAN	ping 172.16.99.10 (DNS)	Success
Cross‑Site	ping 172.16.40.1 (R‑B HR GW)	Success
Internet	ping 203.0.113.10	Success (NAT works)
DNS	nslookup www.isp.com	203.0.113.10
ACL	ping 172.16.30.1 from IT PC	Blocked

### 📁 Repository Structure
enterprise-network-project/
├── README.md
├── project.pkt
├── configs/
│   ├── Switch-A.txt
│   ├── Switch-B.txt
│   ├── R-A.txt
│   ├── R-B.txt
│   └── ISP.txt
├── screenshots/
│   ├── topology.png
│   ├── dhcp.png
│   ├── ospf.png
│   ├── nat.png
│   ├── acl.png
│   └── dns.png
└── report.pdf

### Acknowledgments
Special thanks to Professor Samar Ikram for her guidance, encouragement, and support throughout this Course. Your teaching made this complex network design manageable and fun!

👤 Author
Malik Umar Awan
Computer Networks (CSNC‑2413)
https://github.com/darkfear2005or2006-ai