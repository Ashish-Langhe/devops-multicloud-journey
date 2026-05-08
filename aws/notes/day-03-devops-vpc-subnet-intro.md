# Day 03 — DevOps Role, VPC & Subnet Introduction

> 📅 **Date:** 14th April  
> 🏷️ **Topic:** DevOps Responsibility, On-Prem vs Cloud, VPC Concept, Subnet Intro

---

## 👨‍💻 Role of a DevOps Engineer

> **Core responsibility:** Deploy applications to servers so end-users can access them.

### Two ways to deploy
```
Option 1: On-Premises (On-Prem)
──────────────────────────────
Company owns physical servers
  e.g. Wipro's own data centres in Hyderabad & Delhi
  DevOps deploys app to company's own server

Option 2: Cloud
───────────────
Company rents servers from cloud provider
  e.g. AWS, Azure, GCP
  DevOps deploys app to cloud virtual machine
```

### On-Premises reality (e.g. Wipro)
```
Wipro (Company)
    ↓ needs servers
Wipro On-Prem Data Centre
    ├── Hyderabad DC → [S][S][S]  [S][S][S]
    └── Delhi DC     → [S][S][S]  [S][S][S]

DevOps must manage:
  ✗ Electricity supply
  ✗ Manpower for physical maintenance
  ✗ Ventilation & cooling
  ✗ Backup & disaster recovery
  ✗ High availability setup
  ✗ Scalability planning
```

### Cloud reality (AWS)
```
Wipro (Company)
    ↓ connects over internet
AWS Cloud
    ↓ DevOps selects
Region → AZ → Provision EC2

AWS manages everything physical ✅
DevOps manages only logical resources ✅
```

---

## ☁️ VPC — Virtual Private Cloud

### What is VPC?

> **VPC = A private cloud layer you create INSIDE the public cloud (AWS)**

Think of it like this:
```
AWS Public Cloud (like a huge shopping mall)
┌─────────────────────────────────────────────┐
│                                             │
│   Your VPC (like your private store)        │
│   ┌─────────────────────────────────────┐   │
│   │                                     │   │
│   │   Electronics  Home  Appliances     │   │
│   │   [subnet]    [subnet]  [subnet]    │   │
│   │                                     │   │
│   └─────────────────────────────────────┘   │
│                                             │
└─────────────────────────────────────────────┘
```

- VPC is a **virtual network** you own inside AWS
- Works at **Region level** (not AZ level)
- VPC = **main network** — everything else lives inside it
- You can create a **private cloud layer inside the public cloud**

### VPC Key Facts
```
┌──────────────────────────────────────────────┐
│  VPC = Virtual Private Cloud                 │
│                                              │
│  • Virtual layer on top of AWS cloud         │
│  • Region-level resource                     │
│  • VPC is the MAIN network                   │
│  • Max 5 VPCs per region per account         │
│  • Subnets live inside VPC                   │
│  • Servers live inside Subnets               │
└──────────────────────────────────────────────┘
```

### VPC Structure Visualised
```
Region: Mumbai (ap-south-1)
┌────────────────────────────────────────────────┐
│                  YOUR VPC                      │
│  ┌────────────┐  ┌────────────┐  ┌──────────┐ │
│  │  Subnet 1  │  │  Subnet 2  │  │ Subnet 3 │ │
│  │  (AZ: 1a)  │  │  (AZ: 1b)  │  │ (AZ: 1c) │ │
│  │  EC2 EC2   │  │  EC2 EC2   │  │  EC2     │ │
│  └────────────┘  └────────────┘  └──────────┘ │
└────────────────────────────────────────────────┘
```

---

## 🔀 Subnet — Dividing the VPC

### What is a Subnet?

> **Subnet = Sub-network created by dividing the main VPC network**

```
VPC (Main Network)
  └── Divided into Subnets
        ├── Subnet 1 → lives in AZ-1a
        ├── Subnet 2 → lives in AZ-1b
        └── Subnet 3 → lives in AZ-1c
```

### Amazon analogy (from class)
```
VPC = Amazon.com (main platform)
    ↓ divided into departments (subnets)
    ├── Electronics  [subnet]
    ├── Home         [subnet]
    ├── Appliances   [subnet]
    └── Toys & Baby  [subnet]
```

### Subnet Key Facts
| Property | Detail |
|----------|--------|
| Level | Availability Zone level (not region level) |
| Lives inside | VPC |
| Contains | EC2 servers / instances |
| Purpose | Logical network segmentation |
| Types | Public subnet (internet-facing) / Private subnet (isolated) |

---

## 🏗️ Full 3-Tier Production Architecture (Preview)

This is the architecture you'll learn to build:

```
                     Public Internet
                           │
                     Route 53 (DNS)
                     www.example.com
                           │
                    Internet Gateway
                           │
                    Load Balancer (ALB)
                   /                  \
          ┌────────────┐        ┌────────────┐
          │  AZ: 1a    │        │  AZ: 1b    │
          │            │        │            │
          │ Public      │        │ Public     │
          │ Subnet      │        │ Subnet     │
          │ (NAT GW)    │        │ (NAT GW)   │
          │            │        │            │
          │ Private     │        │ Private    │
          │ App Subnet  │        │ App Subnet │
          │ [EC2][EC2]  │        │ [EC2][EC2] │
          │            │        │            │
          │ Private     │        │ Private    │
          │ DB Subnet   │        │ DB Subnet  │
          │ [Aurora]    │        │ [Aurora    │
          │ (Primary)   │        │  Replica]  │
          └────────────┘        └────────────┘
```

> 💡 This is called a **3-Tier Architecture** — Web tier, App tier, Database tier — all in different subnets across multiple AZs.

---

## 📐 How VPC, Subnet, Server Relate

```
┌─── AWS Account ──────────────────────────────────┐
│                                                   │
│  ┌─── Region (ap-south-1) ─────────────────────┐ │
│  │                                              │ │
│  │  ┌─── VPC (10.0.0.0/16) ────────────────┐  │ │
│  │  │                                       │  │ │
│  │  │  ┌── Subnet (AZ-1a) ──┐              │  │ │
│  │  │  │  EC2  EC2  EC2     │              │  │ │
│  │  │  └───────────────────┘              │  │ │
│  │  │                                       │  │ │
│  │  │  ┌── Subnet (AZ-1b) ──┐              │  │ │
│  │  │  │  EC2  EC2          │              │  │ │
│  │  │  └───────────────────┘              │  │ │
│  │  │                                       │  │ │
│  │  └───────────────────────────────────────┘  │ │
│  └──────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| DevOps role | Deploy apps to servers for end users |
| On-Prem | Company owns servers — DevOps manages physical infra too |
| Cloud | Company rents servers — AWS manages physical, DevOps manages logical |
| VPC | Your private network inside AWS (region-level) |
| Subnet | Sub-division of VPC (AZ-level) |
| VPC → Subnet → Server | Hierarchy: VPC contains subnets, subnets contain servers |
| 5 VPCs max | Per region per AWS account |

---

> 📎 **Next:** Day 04 — Security Groups & Firewall Rules
