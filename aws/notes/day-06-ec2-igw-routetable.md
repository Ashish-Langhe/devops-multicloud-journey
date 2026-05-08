# Day 06 — EC2 Creation, Internet Gateway & Route Table

> 📅 **Date:** 17th April  
> 🏷️ **Topic:** EC2 requirements, AMI, Key Pair, Internet Gateway, Route Table, Public Subnet setup

---

## 💻 EC2 = Your Server on AWS

> **EC2 (Elastic Compute Cloud) = Virtual Machine = Server**  
> `EC2 = vm = server`

---

## 🧩 5 Requirements to Create an EC2

```
┌─────────────────────────────────────────────────────────────┐
│              EC2 Creation Requirements                      │
├──────────────────────────┬──────────────────────────────────┤
│  What you need           │  AWS Service / Component         │
├──────────────────────────┼──────────────────────────────────┤
│  1. OS (Operating System)│  AMI (Amazon Machine Image)      │
│  2. Hardware CPU + RAM   │  Instance Type (e.g., t2.micro)  │
│  3. Hard Disk (Storage)  │  EBS (Elastic Block Storage)     │
│  4. Networking           │  VPC + Subnet + Security Group   │
│  5. Authentication       │  Key Pair (.pem / .ppk)          │
└──────────────────────────┴──────────────────────────────────┘
```

---

## ⚡ Requirement 2 — Instance Type (Hardware)

> **Instance Type = CPU + RAM configuration**

| Instance | vCPUs | RAM | Use Case |
|----------|-------|-----|---------|
| t2.micro | 1 | 1 GB | Free tier / testing |
| t2.small | 1 | 2 GB | Light workloads |
| t2.medium | 2 | 4 GB | Dev environments |
| t2.large | 2 | 8 GB | Medium workloads |
| t3.xlarge | 4 | 16 GB | Production apps |

---

## 💾 Requirement 3 — EBS (Elastic Block Storage)

> **EBS = Hard disk attached to your EC2 instance**

- Default: 8 GB root volume
- You can increase size as needed
- EBS persists even if EC2 is stopped (data not lost)

---

## 🌐 Requirement 4 — Networking (VPC + Subnet + SG)

```
EC2 must be placed inside:
├── VPC        → your private network
├── Subnet     → subdivision of VPC (which AZ)
└── Security Group → firewall rules for this server
```

---

## 🔑 Requirement 5 — Key Pair (Authentication)

> **Key Pair = Authentication mechanism to log into your server**

```
Key Pair
├── Public Key  →  stored ON the server (destination)
└── Private Key →  kept on YOUR laptop (source) — never share!

Private key file formats:
├── .pem  →  for OpenSSH / terminal (Mac, Linux)
└── .ppk  →  for PuTTY (Windows)
```

```
Your Laptop                         EC2 Server
┌──────────────┐                  ┌─────────────┐
│ private.pem  │ ←── SSH ──────→  │ public key  │
│ (secret)     │                  │ (stored)    │
└──────────────┘                  └─────────────┘
```

> ⚠️ **You can only download the private key ONCE** — at creation time. If lost, you cannot SSH into the server.

---

## 🌍 Internet Gateway (IGW)

> **IGW = The door between your VPC and the public internet**

- Attached at the **VPC level**
- Required for ANY subnet to have internet connectivity
- Without IGW → even a "public" EC2 cannot be reached from internet

```
Internet
    │
    ↓
Internet Gateway (IGW)
    │  ← attached to VPC
    ↓
VPC
    └── Public Subnet
          └── EC2 (now reachable from internet ✅)
```

> ⚠️ Just creating an IGW is not enough — you must also **attach it to the VPC** and **add a route in the Route Table**

---

## 🗺️ Route Table (RT)

> **Route Table = Router for the VPC — tells traffic WHERE to go**

- Every VPC has a default route table
- You create a **custom route table** for your public subnet
- Route table rules define where traffic is forwarded

### Route Table Rules
| Destination | Target | Meaning |
|-------------|--------|---------|
| `10.0.0.0/16` | local | Stay inside VPC |
| `0.0.0.0/0` | IGW | All other traffic → go to internet |

```
`0.0.0.0/0`  means "any IP address" = internet traffic
When destination = 0.0.0.0/0, target = IGW
→ This makes the subnet PUBLIC (internet-accessible)
```

---

## 🏗️ How to Create a Public Subnet — Step by Step

```
STEP ①  Create VPC
         Region: select your region
         CIDR: 10.0.0.0/16

STEP ②  Create Subnet inside VPC
         AZ: us-east-1a
         CIDR: 10.0.0.0/24

STEP ③  Create Internet Gateway → Attach to VPC

STEP ④  Create Route Table inside same VPC

STEP ⑤  Add Route in Route Table
         Destination: 0.0.0.0/0
         Target: Internet Gateway (IGW)

STEP ⑥  Associate Subnet with Route Table

STEP ⑦  Launch EC2 inside subnet
         Enable: Auto-assign Public IP = ON
```

### Visual: What gets created
```
VPC: 10.0.0.0/16
┌─────────────────────────────────────────┐
│                            ③ IGW        │
│                            ↕            │
│                         ⑤ RT           │
│                      (0.0.0.0/0 → IGW) │
│          ⑥ Subnet Association          │
│  ┌──────────────────────────────┐       │
│  │  ② Public Subnet             │       │
│  │  10.0.0.0/24                 │       │
│  │                              │       │
│  │  ⑦ EC2  (public server)     │       │
│  └──────────────────────────────┘       │
└─────────────────────────────────────────┘
```

---

## 🔑 What Makes a Subnet "Public"?

```
Public Subnet  = Subnet + Route Table with 0.0.0.0/0 → IGW
Private Subnet = Subnet + Route Table WITHOUT internet route

The subnet itself is NOT public or private by nature.
The ROUTE TABLE makes it public or private.
```

| | Public Subnet | Private Subnet |
|--|--------------|----------------|
| Has route to IGW | ✅ Yes | ❌ No |
| EC2 gets Public IP | ✅ Can be enabled | ❌ No public IP |
| Internet can reach EC2 | ✅ Yes | ❌ No |
| EC2 can reach internet | ✅ Yes | ❌ No (needs NAT) |
| Use case | Bastion, LB, NAT GW | App servers, DBs |

---

## 🌐 Visual Subnet Calculator Tool

> **Tip from class:** Use [cidr.xyz](https://cidr.xyz) or visual subnet calculator tools to verify your CIDR ranges before applying.

```
Example from class:
Network: 10.0.0.0 / Mask: /16
→ Range: 192.168.0.0 → 192.168.255.255
→ Usable IPs: 65,534
→ Hosts: 65,534
```

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| EC2 | Virtual machine — needs AMI + type + EBS + network + key |
| AMI | OS + apps packaged as image — region-specific |
| Instance type | CPU + RAM config (t2.micro = 1 vCPU, 1 GB RAM) |
| EBS | Hard disk for EC2 — persists after stop |
| Key pair | Private key on your laptop, public key on server — never share private |
| IGW | Door from VPC to internet — must be attached to VPC |
| Route Table | Router — `0.0.0.0/0 → IGW` makes subnet public |
| Public subnet | Subnet with RT pointing to IGW |
| Private subnet | Subnet WITHOUT internet route in RT |

---

> 📎 **Next:** Day 07 — Custom Networking, Public & Private Subnets Hands-On, SSH Connection
