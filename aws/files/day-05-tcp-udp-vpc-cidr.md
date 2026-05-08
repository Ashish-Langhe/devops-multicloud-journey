# Day 05 — TCP/UDP, VPC Deep Dive & CIDR Calculation

> 📅 **Date:** 16th April  
> 🏷️ **Topic:** TCP vs UDP, VPC sizing, CIDR math, Subnet creation, IP ranges

---

## 📡 TCP vs UDP — Transport Protocols

### TCP (Transmission Control Protocol)

> **TCP verifies the connection FIRST, then transfers data**

```
Step 1: Client sends CONNECTION REQUEST  ──────→ Server
Step 2: Server sends CONNECTION ACCEPT   ←────── Server
Step 3: Data transfer begins via HTTP/HTTPS
```

```
┌──────────────────────────────────────────────┐
│  TCP                                         │
│                                              │
│  ✅ Establishes connection between           │
│     source and destination first             │
│  ✅ Data transfers only after connection     │
│  ✅ Uses HTTP/HTTPS                          │
│  ✅ No data loss — reliable                  │
│  ⚠️  Slightly slower (setup time)            │
└──────────────────────────────────────────────┘
```

### UDP (User Datagram Protocol)

> **UDP does NOT establish or verify any connection — sends data directly**

```
Client → data packets → Server  (no handshake, no confirmation)
```

```
┌──────────────────────────────────────────────┐
│  UDP                                         │
│                                              │
│  ❌ No connection established                │
│  ❌ No connection verification               │
│  ⚠️  Data loss possible (no dedicated path)  │
│  ✅ Faster (no setup overhead)               │
│  ✅ Good for real-time: video, gaming, DNS   │
└──────────────────────────────────────────────┘
```

### TCP vs UDP Side-by-Side

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Established before data transfer | No connection |
| Reliability | ✅ No data loss | ⚠️ Possible data loss |
| Speed | Slower (handshake overhead) | Faster |
| Protocol | HTTP, HTTPS, SSH, FTP | Video stream, DNS, Gaming |
| Use case | Web apps, file transfer, SSH | Live video, online games |

---

## 🏗️ VPC Deep Dive — Sizing & Limits

### VPC = Virtual Private Cloud
- Layer inside public cloud (AWS)
- **Region-level** resource
- Everything (subnets, servers) lives inside VPC

### VPC Key Questions from Class

```
┌─────────────────────────────────────────────────────────────┐
│  Q: How many VPCs can I create inside a region?             │
│  A: 5 VPCs per region per account                           │
│                                                             │
│  Q: How many subnets can I create inside a VPC?             │
│  A: Depends on the SIZE of the VPC (CIDR block)             │
│                                                             │
│  Q: How many servers can I create inside a subnet?          │
│  A: Depends on the SIZE of the subnet (CIDR block)          │
│     (250+ servers per subnet is common)                     │
└─────────────────────────────────────────────────────────────┘
```

### VPC — Full Architecture
```
AWS Account
  └── Region (ap-south-1)
        └── VPC (10.0.0.0/16)
              ├── Subnet-1 (AZ: 1a) → EC2, EC2, EC2 ...
              ├── Subnet-2 (AZ: 1b) → EC2, EC2 ...
              └── Subnet-3 (AZ: 1c) → EC2 ...
```

---

## 🔢 CIDR — How to Calculate IP Range

> **CIDR = Classless Inter-Domain Range**  
> The number after `/` is called the **Netmask Value**

### CIDR Format
```
10.0.0.0/24
    ↑      ↑
  IPv4   Netmask value
```

### IPv4 Structure
```
10  .  0  .  0  .  0
 ↑     ↑     ↑     ↑
8bits 8bits 8bits 8bits  =  32 bits total
```

### Why use 10.x.x.x?

> Private IP address ranges reserved globally:

```
┌────────────────────────────────────────────┐
│  Private Network IP Series                 │
│  ─────────────────────────────────────     │
│  10.x.x.x      → dedicated private IPs    │
│  172.x.x.x     → dedicated private IPs    │
│  192.168.x.x   → dedicated private IPs    │
│                                            │
│  All other series → used for public IPs   │
└────────────────────────────────────────────┘
```

> That's why AWS VPC uses `10.0.0.0` — it's a private IP range, safe from internet conflicts.

### CIDR Math Formula

```
Number of IPs  =  2 ^ (32 - netmask)
```

### Worked Examples

**Example 1: `10.0.0.0/24`**
```
32 - 24 = 8
2^8 = 256 IPs

IP Range: 10.0.0.0 → 10.0.0.255
Focus on last octet: 10.0.0.*
```

**Example 2: `10.0.0.0/26`**
```
32 - 26 = 6
2^6 = 64 IPs

IP Range: 10.0.0.0 → 10.0.0.63
```

**Example 3: `10.0.0.0/16`**
```
32 - 16 = 16
2^16 = 65,536 IPs

Large VPC — suitable for many subnets
```

### CIDR Quick Reference Table

| CIDR | Formula | IPs | Use |
|------|---------|-----|-----|
| /16 | 2^16 | 65,536 | Large VPC |
| /24 | 2^8 | 256 | Medium subnet |
| /25 | 2^7 | 128 | Split /24 in half |
| /26 | 2^6 | 64 | Small subnet |
| /28 | 2^4 | 16 | Smallest allowed in AWS |

> ⚠️ **AWS VPC CIDR must be between `/16` and `/28`**

---

## ✂️ Splitting a VPC into Subnets

### Example: Split `10.0.0.0/24` (256 IPs) into 2 subnets

```
VPC: 10.0.0.0/24  →  256 IPs total
           ↓
    Split into 2 equal subnets:

  Subnet-1: 10.0.0.0/25
  ┌──────────────────────────────────┐
  │  128 IPs: 10.0.0.0 → 10.0.0.127 │
  └──────────────────────────────────┘

  Subnet-2: 10.0.0.128/25
  ┌──────────────────────────────────────┐
  │  128 IPs: 10.0.0.128 → 10.0.0.255   │
  └──────────────────────────────────────┘
```

### ❌ Common Error — Duplicate CIDR

```
VPC: 10.0.0.0/24
  Subnet-1: 10.0.0.0/24  ✅ Created
  Subnet-2: 10.0.0.0/24  ❌ ERROR

ERROR: CIDR Address overlaps with existing Subnet CIDR: 10.0.0.0/24

Both subnets trying to register same CIDR
→ System does NOT allow same CIDR for two subnets inside same VPC
→ Each subnet MUST have a unique, non-overlapping CIDR
```

---

## 🖥️ Hands-On: Creating VPC & Subnets

### AWS Console Path
```
VPC Dashboard → Your VPCs → Create VPC
  Name: dev
  IPv4 CIDR: 10.0.0.0/24
  IPv6: No IPv6
→ Create

Then: Subnets → Create Subnet
  VPC: dev
  Name: subnet-1-dev
  AZ: us-east-1a
  CIDR: 10.0.0.0/25  (128 IPs)
→ Create
```

> 💡 **Default subnets** are auto-created by AWS in every region — ignore these, always create custom subnets.

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| TCP | Verifies connection first → reliable, used by HTTP/HTTPS |
| UDP | No connection → fast, possible data loss, used by video/gaming |
| VPC | Your private network in AWS (max 5 per region) |
| CIDR | Defines IP range — format: `IP/netmask` |
| Formula | `2^(32 - netmask)` = number of IPs |
| Private IPs | 10.x.x.x, 172.x.x.x, 192.168.x.x |
| AWS CIDR range | Must be between /16 and /28 |
| Subnet CIDR | Must be unique within same VPC — no overlapping allowed |
| 250+ servers | Can be created under one subnet (depends on CIDR size) |

---

> 📎 **Next:** Day 06 — EC2 Creation, Internet Gateway, Route Table & Public Subnet
