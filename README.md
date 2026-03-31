# 🏦 Bank Enterprise Network Design

> A full-scale enterprise bank network simulation built with **Cisco Packet Tracer**, featuring multi-branch connectivity, department-level VLAN segmentation, cloud-hosted server infrastructure, and inter-router WAN links.

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Network Architecture](#network-architecture)
3. [Branch Structure](#branch-structure)
4. [VLAN Design (Main Branch)](#vlan-design-main-branch)
5. [Cloud Server Room](#cloud-server-room)
6. [Routing & WAN Connectivity](#routing--wan-connectivity)
7. [IP Addressing Scheme](#ip-addressing-scheme)
8. [Device Inventory](#device-inventory)
9. [Security Considerations](#security-considerations)
10. [Technologies Used](#technologies-used)
11. [How to Open & Run the Simulation](#how-to-open--run-the-simulation)
12. [Project Files](#project-files)
13. [Design Decisions & Rationale](#design-decisions--rationale)
14. [Potential Extensions](#potential-extensions)

---

## Project Overview

This project simulates a **real-world bank enterprise network** using Cisco Packet Tracer. It models the complete network infrastructure of a bank that operates across multiple geographic locations — one **main branch** and **three sub-branches** — all interconnected via routers and linked to a centralized **cloud-based server room**.

The simulation covers:
- **Layer 2 segmentation** via VLANs within the main branch
- **Layer 3 inter-VLAN routing** using a router-on-a-stick or multilayer switch setup
- **WAN-style router interconnection** across all four branches
- **Cloud services** including DNS, FTP, and Web servers
- **End-to-end connectivity verification** via ping, DNS resolution, and service access

This project is ideal for understanding how mid-to-large enterprises structure their internal networks — particularly in regulated industries like banking, where **network isolation, access control, and service availability** are critical.

---

## Network Architecture

```
                          ┌─────────────────────────────┐
                          │        CLOUD / ISP           │
                          │  ┌────────┐  ┌────────────┐  │
                          │  │  Web   │  │    FTP     │  │
                          │  │ Server │  │   Server   │  │
                          │  └────────┘  └────────────┘  │
                          │       ┌─────────────┐         │
                          │       │  DNS Server │         │
                          │       └─────────────┘         │
                          └──────────────┬────────────────┘
                                         │
                                    [Cloud Router]
                                         │
              ┌──────────────────────────┼──────────────────────────┐
              │                          │                           │
        [Branch 1]               [Main Branch]                [Branch 2]
        Router + Switch         Router + 4 VLANs             Router + Switch
        End Devices             Department PCs               End Devices
                                         │
                                   [Branch 3]
                                   Router + Switch
                                   End Devices
```

The **Main Branch** is the hub of operations. It hosts the most complex layer-2 topology (VLAN segmentation), while all branches maintain simpler flat networks connected upward to the main router mesh.

---

## Branch Structure

### Main Branch
- The most complex node in the network.
- Hosts **4 separate departments**, each isolated in its own VLAN.
- Has a **Layer 3-capable router** handling inter-VLAN routing and WAN uplinks.
- A **managed Layer 2 switch** connects all department access ports and trunks back to the router.

### Branch 1, Branch 2, Branch 3
- Each remote branch has:
  - 1 **Router** for WAN connectivity back to main branch / cloud
  - 1 **Switch** for local device connectivity
  - Multiple **end-user PCs** (tellers, loan officers, general staff)
- Remote branches operate on a **flat (non-VLAN) LAN** unless extended.
- All branches communicate via **router-to-router serial or FastEthernet WAN links**.

---

## VLAN Design (Main Branch)

The Main Branch is divided into **4 departments**, each in its own VLAN. This ensures that:
- Broadcast domains are limited per department.
- Sensitive departments (e.g., Finance) cannot directly reach user workstations.
- Security policies and ACLs can be applied at the VLAN boundary.

| VLAN ID | Department            | Purpose                                     |
|---------|-----------------------|---------------------------------------------|
| VLAN 10 | Management / IT       | Network admin, IT helpdesk workstations     |
| VLAN 20 | Finance / Accounts    | Financial transactions, sensitive records   |
| VLAN 30 | Customer Service      | Teller PCs, customer-facing terminals       |
| VLAN 40 | HR / Administration   | Human resources, internal admin systems     |

> **Note:** VLAN IDs above are the most common industry-standard interpretation of a 4-department bank VLAN layout. Actual IDs in the simulation may vary slightly.

### Trunk Links
- The link between the **main branch switch** and the **router** is configured as a **trunk port** (802.1Q encapsulation), carrying all 4 VLANs.
- All other switch ports connected to PCs are **access ports** assigned to their respective VLAN.

### Inter-VLAN Routing
- The router uses **Router-on-a-Stick** (sub-interfaces per VLAN on a single physical port) **or** a multilayer switch handles inter-VLAN routing locally.
- Each VLAN sub-interface on the router is assigned the **default gateway IP** for that VLAN subnet.

---

## Cloud Server Room

The bank's centralized **server farm** is hosted on a simulated cloud and provides core services to all branches:

| Server     | Role                                                       |
|------------|------------------------------------------------------------|
| **Web Server**  | Hosts the bank's internal web portal/intranet. Accessible via HTTP from all branch PCs. |
| **FTP Server**  | File Transfer Protocol server for document sharing and backups across branches. |
| **DNS Server**  | Domain Name System resolution — translates domain names (e.g., `bank.local`) to IP addresses for all network devices. |

All servers are reachable from all branches via routed paths through the WAN. The DNS server ensures that PCs across the network can access services by name rather than raw IP.

---

## Routing & WAN Connectivity

### Protocol
The network uses **static routing** or **RIP (Routing Information Protocol)** to establish paths between all routers. In a typical Packet Tracer simulation of this scale:

- Each **router has a routing table** with entries for all remote subnets.
- Routes are either manually configured (static) or learned via RIP.
- RIP v2 is preferred as it supports **classless routing** (VLSM-compatible).

### WAN Links
- Routers are interconnected via **Serial DCE/DTE links** or **FastEthernet cross-connects**.
- Each WAN link uses a **dedicated /30 subnet** (4 addresses, 2 usable) — the industry standard for point-to-point links.
- Clock rate is configured on the **DCE side** of serial links.

### Connectivity Summary
```
Main Branch Router ──── Branch 1 Router
Main Branch Router ──── Branch 2 Router
Main Branch Router ──── Branch 3 Router
Main Branch Router ──── Cloud Router ──── Server Room
```

---

## IP Addressing Scheme

A typical addressing plan for this topology (actual values may differ in the simulation):

### LAN Subnets

| Location            | Network         | Subnet Mask       | Usable Range              | Gateway       |
|---------------------|-----------------|-------------------|---------------------------|---------------|
| Main Branch VLAN 10 | 192.168.10.0    | /24 (255.255.255.0) | .1 – .254               | 192.168.10.1  |
| Main Branch VLAN 20 | 192.168.20.0    | /24               | .1 – .254                 | 192.168.20.1  |
| Main Branch VLAN 30 | 192.168.30.0    | /24               | .1 – .254                 | 192.168.30.1  |
| Main Branch VLAN 40 | 192.168.40.0    | /24               | .1 – .254                 | 192.168.40.1  |
| Branch 1 LAN        | 192.168.1.0     | /24               | .1 – .254                 | 192.168.1.1   |
| Branch 2 LAN        | 192.168.2.0     | /24               | .1 – .254                 | 192.168.2.1   |
| Branch 3 LAN        | 192.168.3.0     | /24               | .1 – .254                 | 192.168.3.1   |
| Cloud/Server Room   | 192.168.100.0   | /24               | .1 – .254                 | 192.168.100.1 |

### WAN Point-to-Point Links

| Link                         | Network        | Router A IP    | Router B IP    |
|------------------------------|----------------|----------------|----------------|
| Main ↔ Branch 1              | 10.0.0.0/30    | 10.0.0.1       | 10.0.0.2       |
| Main ↔ Branch 2              | 10.0.1.0/30    | 10.0.1.1       | 10.0.1.2       |
| Main ↔ Branch 3              | 10.0.2.0/30    | 10.0.2.1       | 10.0.2.2       |
| Main ↔ Cloud                 | 10.0.3.0/30    | 10.0.3.1       | 10.0.3.2       |

---

## Device Inventory

| Device Type          | Quantity (Approx.) | Role                                            |
|----------------------|--------------------|-------------------------------------------------|
| Cisco Router (2811 / 1941) | 5         | 1 per branch + 1 cloud router                   |
| Cisco Switch (2950 / 2960) | 4+        | 1 per branch; managed switch in main branch     |
| PCs / Workstations   | 12–20              | End-user devices per department and branch      |
| Servers              | 3                  | Web, FTP, DNS (cloud)                           |
| Cloud (Packet Tracer component) | 1     | Hosts the server room                           |

---

## Security Considerations

While this is a simulation, it models several real-world security practices:

1. **VLAN Isolation** — Departments in the main branch cannot communicate directly; all traffic must pass through the router, where ACLs can be applied.
2. **Dedicated Server Subnet** — The server room is on a separate network, reachable only via a routed path, not directly exposed to any branch LAN.
3. **Access Port Lockdown** — End-user switch ports are configured as access ports with a single VLAN, preventing unauthorized VLAN hopping.
4. **Trunk Security** — Trunk links are explicitly configured and limited to the required VLANs only (no native VLAN vulnerabilities).
5. **No Split Tunneling / Direct Internet** — All inter-branch traffic is routed through defined paths; there are no uncontrolled external internet-facing links in the simulation.

In a production deployment, you would additionally configure:
- Port Security on switches (MAC address filtering)
- ACLs on router interfaces
- AAA (Authentication, Authorization, Accounting) via RADIUS/TACACS+
- Encrypted remote management (SSH instead of Telnet)
- DHCP Snooping and Dynamic ARP Inspection

---

## Technologies Used

| Technology              | Usage                                                    |
|-------------------------|----------------------------------------------------------|
| **Cisco Packet Tracer** | Network simulation and visual topology design            |
| **VLANs (802.1Q)**      | Layer 2 network segmentation at the main branch          |
| **Router-on-a-Stick**   | Inter-VLAN routing via router sub-interfaces             |
| **RIP v2 / Static Routes** | Layer 3 routing between all routers                  |
| **DNS**                 | Name resolution across the network                       |
| **HTTP (Web Server)**   | Internal web portal accessible from all branches         |
| **FTP**                 | File transfer and storage services                        |
| **Serial/FastEthernet WAN** | Point-to-point inter-router links                    |
| **DHCP** (optional)     | Dynamic IP assignment on branch LANs                     |
| **TCP/IP Suite**        | Foundational protocol stack across all devices           |

---

## How to Open & Run the Simulation

### Prerequisites
- **Cisco Packet Tracer** version **7.x or later** installed.
- Free download available at: [https://www.netacad.com](https://www.netacad.com) (requires free NetAcad account)

### Steps

1. **Clone or download** this repository:
   ```bash
   git clone https://github.com/<username>/Bank-Enterprise-Network-Design.git
   ```

2. **Open Cisco Packet Tracer**.

3. In Packet Tracer, go to **File → Open** and navigate to:
   ```
   Bank-Enterprise-Network-Design-main/Bank_Enterprise_Network.pkt
   ```

4. The full topology will load with all devices, links, and configurations.

5. **Test connectivity** using the simulation tools:
   - Use **PC Command Prompt** → `ping <IP>` to verify end-to-end reachability.
   - Use **PC Web Browser** → type the Web Server's IP or domain to test HTTP access.
   - Use the **Simulation Mode** (bottom-right clock icon) to trace individual packets through the network step-by-step.

6. **Explore device configs** by clicking any router or switch → CLI tab to view running configurations.

---

## Project Files

```
Bank-Enterprise-Network-Design-main/
│
├── Bank_Enterprise_Network.pkt    # Main Cisco Packet Tracer simulation file
└── README.md                      # Project documentation (this file)
```

| File | Description |
|------|-------------|
| `Bank_Enterprise_Network.pkt` | The complete network topology with all device configurations, IP addressing, VLANs, routing, and server setup. Open this in Cisco Packet Tracer. |
| `README.md` | Project overview and documentation. |

---

## Design Decisions & Rationale

### Why VLANs only at the Main Branch?
Remote branches are smaller (fewer staff, simpler operations) and typically don't require department-level isolation. Implementing VLANs at every branch would add configuration complexity without proportionate security benefit at this scale.

### Why a Cloud-hosted Server Room?
In real banking environments, core services (DNS, backup, web portals) are centralized — not replicated at every branch. Centralizing them on a "cloud" node in the simulation mirrors this architecture and lets all 4 branches reach the same services via routed paths, testing WAN reachability comprehensively.

### Why Router-on-a-Stick over a Layer 3 Switch?
For educational clarity. Router-on-a-Stick makes the inter-VLAN routing process explicit and visible (sub-interfaces are clearly labeled), which is better for learning. In production, a Layer 3 switch is preferred for performance.

### Why 4 Departments?
Banking environments typically segment: IT/Management, Finance/Ops, Customer-Facing Services, and HR/Admin. These four represent the minimum realistic set of isolated trust zones in a bank.

---

## Potential Extensions

If you want to expand on this project, consider adding:

- [ ] **DHCP Server** — Replace static IPs on end devices with dynamic assignment from a centralized DHCP server.
- [ ] **ACLs (Access Control Lists)** — Restrict which VLANs/branches can reach the FTP or Web server.
- [ ] **NAT/PAT** — Simulate internet access via Network Address Translation on the edge router.
- [ ] **OSPF Routing** — Replace RIP with OSPF for faster convergence and better scalability.
- [ ] **Wireless Access Points** — Add Wi-Fi capability for customer zones at each branch.
- [ ] **VPN Tunnels** — Simulate site-to-site VPN between branches for encrypted WAN traffic.
- [ ] **Redundant Links (STP)** — Add redundant switch uplinks with Spanning Tree Protocol to eliminate single points of failure.
- [ ] **AAA / RADIUS Server** — Centralized authentication for all network device logins.
- [ ] **Email Server** — Add internal mail service for bank staff communications.
- [ ] **IPv6 Dual-Stack** — Configure IPv6 addressing alongside IPv4 on all links.

---

## Author

Simulated and designed as an enterprise network architecture exercise using **Cisco Packet Tracer**.

> For learning purposes. Models real-world bank network topology concepts including VLAN segmentation, inter-branch routing, centralized cloud services, and multi-department access control.

---

*Simulation file: `Bank_Enterprise_Network.pkt` | Tool: Cisco Packet Tracer 7+*
