# 🏢 Multi-Floor Office Network — Cisco Packet Tracer Lab

> **Beginner Guided Lab** · Dual ISP Failover · VLANs · OSPF · Inter-VLAN Routing · DHCP

![Network Topology](diagrams/topology-preview.png)

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [What You Will Learn](#-what-you-will-learn)
- [Network Topology](#-network-topology)
- [IP Address Plan](#-ip-address-plan)
- [Repo Structure](#-repo-structure)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Lab Phases](#-lab-phases)
- [Testing & Verification](#-testing--verification)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

---

## 📖 Project Overview

This repository contains a **complete guided lab project** for building a realistic multi-floor enterprise office network from scratch using **Cisco Packet Tracer**. The network connects three floors of an office building to the internet through **two separate ISPs**, with automatic failover — if ISP1 goes down, traffic instantly reroutes through ISP2 with zero manual intervention.

This is a hands-on, beginner-friendly lab designed to bridge the gap between IT helpdesk skills and cloud/network engineering — every command is explained, not just listed.

| Attribute | Detail |
|---|---|
| **Difficulty** | Beginner → Intermediate |
| **Estimated Time** | 3–4 hours |
| **Tool Required** | Cisco Packet Tracer 8.x+ |
| **Protocols Used** | OSPF, 802.1Q VLAN Trunking, DHCP, ICMP |
| **Topology** | Hierarchical (Access → Distribution → Core → Edge) |

---

## 🎯 What You Will Learn

- ✅ How to design and build a **hierarchical enterprise network**
- ✅ What **VLANs** are and how to use them to segment floor traffic
- ✅ How **trunk ports** carry multi-VLAN traffic between switches
- ✅ How **Layer 3 switches** perform inter-VLAN routing using SVIs
- ✅ How **OSPF** dynamically distributes routes across the network
- ✅ How **floating static routes** create automatic ISP failover
- ✅ How to configure a **DHCP server** on a Layer 3 switch
- ✅ How to **test, verify, and troubleshoot** a live network

---

## 🗺️ Network Topology

```
        ┌─────────────┐          ┌─────────────┐
        │  ISP1-Rogers │          │  ISP2-ATT   │
        │ 203.0.113.1  │          │ 198.51.100.1│
        └──────┬───────┘          └──────┬───────┘
               │ Se0/3/0 (Serial WAN)    │ Se0/3/0
               │                         │
        ┌──────┴───────┐          ┌──────┴───────┐
        │  Router-R1   │          │  Router-R2   │
        │ 203.0.113.2  │          │ 198.51.100.2 │
        │   10.0.0.1   │          │   10.0.0.5   │
        └──────┬───────┘          └──────┬───────┘
               │ Gi0/0 (LAN)             │ Gi0/0
               │                         │
        ┌──────┴───────┐  Trunk   ┌──────┴───────┐
        │  Core-SW1    ├──────────┤  Core-SW2    │
        │  10.0.0.2    │          │  10.0.0.6    │
        │ VLAN 10 GW   │          │              │
        │ VLAN 20 GW   │          │              │
        │ VLAN 30 GW   │          │              │
        └──┬───────┬───┘          └──────┬───────┘
           │       │                     │
     ┌─────┘       └──────┐       ┌──────┘
     │                    │       │
┌────┴──────┐      ┌──────┴──┐  ┌─┴─────────┐
│ ACC-SW1   │      │ ACC-SW2 │  │ ACC-SW3   │
│ FLOOR 1   │      │ FLOOR 2 │  │ FLOOR 3   │
│ VLAN 10   │      │ VLAN 20 │  │ VLAN 30   │
│192.168.10 │      │192.168.20  │192.168.30 │
└─────┬─────┘      └────┬────┘  └─────┬─────┘
      │                 │             │
  [PCs,APs,          [PCs,APs,    [PCs,APs,
   Printer-1]         Printer-3]   Printer-2]
```

### Link Legend

| Line Type | Meaning |
|---|---|
| Serial cable | WAN link (ISP to Router) |
| Straight-through copper | LAN links (Router → Switch, Switch → Switch) |
| Trunk port | Carries multiple VLANs (802.1Q tagged) |
| Access port | Single VLAN, connects to end devices |

---

## 📊 IP Address Plan

### WAN Interfaces

| Device | Interface | IP Address | Subnet | Role |
|---|---|---|---|---|
| ISP1 Cloud | Se0/3/0 | 203.0.113.1 | /30 | ISP1 gateway |
| Router-R1 | Se0/1/0 | 203.0.113.2 | /30 | Uplink to ISP1 |
| ISP2 Cloud | Se0/3/0 | 198.51.100.1 | /30 | ISP2 gateway |
| Router-R2 | Se0/1/0 | 198.51.100.2 | /30 | Uplink to ISP2 |

### LAN Point-to-Point Links

| Device | Interface | IP Address | Subnet | Role |
|---|---|---|---|---|
| Router-R1 | Gi0/0 | 10.0.0.1 | /30 | LAN uplink to Core-SW1 |
| Core-SW1 | Gi1/0/1 | 10.0.0.2 | /30 | Uplink to R1 |
| Router-R2 | Gi0/0 | 10.0.0.5 | /30 | LAN uplink to Core-SW2 |
| Core-SW2 | Gi1/0/1 | 10.0.0.6 | /30 | Uplink to R2 |

### VLAN SVIs (Default Gateways)

| VLAN | Name | SVI IP | Subnet | DHCP Range | Floor |
|---|---|---|---|---|---|
| VLAN 10 | Floor1-Users | 192.168.10.1 | /24 | .11 – .254 | Floor 1 |
| VLAN 20 | Floor2-Users | 192.168.20.1 | /24 | .11 – .254 | Floor 2 |
| VLAN 30 | Floor3-Users | 192.168.30.1 | /24 | .11 – .254 | Floor 3 |

### Static Device IPs (Reserved)

| Device | IP Address | VLAN |
|---|---|---|
| Printer-1 (Floor 1) | 192.168.10.5 | 10 |
| Printer-2 (Floor 3) | 192.168.30.5 | 30 |
| Printer-3 (Floor 2) | 192.168.20.5 | 20 |

---

## 📁 Repo Structure

```
packet-tracer-lab/
│
├── README.md                        ← You are here
├── LICENSE
├── .gitignore
│
├── .github/
│   ├── workflows/
│   │   ├── validate-configs.yml     ← Lints config files on every push
│   │   └── lab-release.yml          ← Packages lab files on tagged release
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.md            ← Report issues with the lab
│       └── lab_question.md          ← Ask questions about a specific step
│
├── configs/                         ← All device CLI configurations
│   ├── routers/
│   │   ├── Router-R1.txt
│   │   └── Router-R2.txt
│   ├── core-switches/
│   │   ├── Core-SW1.txt
│   │   └── Core-SW2.txt
│   └── access-switches/
│       ├── ACC-SW1-Floor1.txt
│       ├── ACC-SW2-Floor2.txt
│       └── ACC-SW3-Floor3.txt
│
├── docs/
│   ├── guided-lab.docx              ← Full step-by-step beginner guide
│   ├── phase-by-phase-guide.md      ← Markdown version of the lab guide
│   └── troubleshooting.md           ← Common issues and fixes
│
├── diagrams/
│   ├── topology-preview.png         ← Network diagram image
│   └── ip-address-plan.md           ← IP reference sheet
│
└── tests/
    ├── ping-test-checklist.md        ← Manual test cases to run
    └── verification-commands.md     ← All 'show' commands to validate config
```

---

## ✅ Prerequisites

Before starting this lab, make sure you have:

- [ ] **Cisco Packet Tracer 8.x** installed (free via [Cisco Networking Academy](https://www.netacad.com))
- [ ] Basic understanding of what an IP address and subnet mask are
- [ ] Familiarity with navigating the Packet Tracer GUI
- [ ] The guided lab document: `docs/guided-lab.docx`

No prior CLI experience required — every command is explained in the guide.

---

## 🚀 Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/YOUR_USERNAME/packet-tracer-lab.git
cd packet-tracer-lab

# 2. Open the guided lab document
open docs/guided-lab.docx

# 3. Open Packet Tracer and start a new blank project

# 4. Follow the phases in order (see Lab Phases below)

# 5. Use the config files in /configs as reference or to verify your work
```

> **Tip:** Build the network yourself first from the guide. Use the config files in `/configs` only to check your work or if you get stuck — you learn far more by typing the commands yourself.

---

## 🔬 Lab Phases

| Phase | Title | Key Topics |
|---|---|---|
| **Phase 1** | Build the Topology | Device placement, cabling, serial modules |
| **Phase 2** | Configure Access Switches | VLANs, access ports, trunk ports |
| **Phase 3** | Configure Core Switches | ip routing, SVIs, DHCP, inter-VLAN routing |
| **Phase 4** | Configure Routers | WAN interfaces, OSPF, floating static routes |
| **Phase 5** | Configure End Devices | DHCP clients, static printer IPs |
| **Phase 6** | Test & Verify | Ping tests, show commands, ISP failover simulation |
| **Phase 7** | Final Checklist | Full validation before sign-off |

---

## 🧪 Testing & Verification

### Key verification commands

```bash
# Check OSPF neighbors (should show FULL state)
show ip ospf neighbor

# Check routing table (look for O = OSPF routes)
show ip route

# Check VLAN assignments on access switches
show vlan brief

# Check which ports are trunking
show interfaces trunk

# Check DHCP leases
show ip dhcp binding
```

### ISP Failover Test

```bash
# Step 1 — Confirm primary path (AD=1 via ISP1)
Router-R1# show ip route 0.0.0.0

# Step 2 — Simulate ISP1 failure
Router-R1(config)# interface Se0/1/0
Router-R1(config-if)# shutdown

# Step 3 — Confirm failover to ISP2 (AD=200 route activates)
Router-R1# show ip route 0.0.0.0
# Expected: S* 0.0.0.0/0 [200/0] via 10.0.0.5

# Step 4 — Restore ISP1
Router-R1(config-if)# no shutdown
```

---

## 🛠️ Troubleshooting

See [`docs/troubleshooting.md`](docs/troubleshooting.md) for the full guide. Quick reference:

| Symptom | Likely Cause | Quick Fix |
|---|---|---|
| PC not getting DHCP IP | Port not in correct VLAN | `show vlan brief` on access switch |
| Inter-VLAN ping fails | `ip routing` not enabled | Run `ip routing` on Core-SW1 |
| OSPF neighbors not forming | IP mismatch on /30 link | Check both sides of the link IPs |
| Failover not working | Wrong AD or unreachable next-hop | `show ip route` — verify both routes exist |
| Can't ping ISP | No default route | Check `show ip route` for 0.0.0.0/0 |

---

## 🤝 Contributing

Found a mistake in the guide? Have a better config? Contributions are welcome.

1. Fork the repo
2. Create a branch: `git checkout -b fix/ospf-config-typo`
3. Commit your changes: `git commit -m "fix: correct OSPF wildcard mask in Core-SW1 config"`
4. Push and open a Pull Request

Please use the issue templates in `.github/ISSUE_TEMPLATE/` to report bugs or ask questions.

---

## 📄 License

MIT License — see [`LICENSE`](LICENSE) for details.

---

> Built for learning. Inspired by real enterprise network design.
> If this lab helped you, give it a ⭐ on GitHub!

