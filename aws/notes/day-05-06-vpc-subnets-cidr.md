# Day 05-06 | VPC, Subnets & CIDR

**Dates:** 16th April – 17th April  
**Topics:** VPC, Subnets, CIDR calculation, Public vs Private Subnet

---

## 🏠 What is VPC (Virtual Private Cloud)?

- VPC = **Private Cloud layer inside Public Cloud**
- It is a **virtual network** you create inside AWS
- VPC works at **Region level** (not AZ level)
- VPC is the **main network** — everything lives inside it

```
Public Cloud (AWS)
  └── VPC (your private network)
        ├── Subnet 1 (AZ-1a)
        ├── Subnet 2 (AZ-1b)
        └── Subnet 3 (AZ-1c)
```

### VPC Limits
- Max **5 VPCs per region** per AWS account
- Number of subnets depends on **VPC size (CIDR)**
- Number of servers inside subnet depends on **subnet size**

---

## 🔢 CIDR — How to Calculate VPC Size

**CIDR** = Classless Inter-Domain Range (also called Netmask Value)

### Format
```
10.0.0.0/24
   ↑      ↑
  IPv4  Netmask value
```

### Why 10.x.x.x?
- IPs starting with **10, 172, 192** are **dedicated private network IPs**
- All other IP series are used for **public networks**
- AWS VPC CIDR must be between `/16` and `/28`

### How to calculate number of IPs?

**Formula:** `2^(32 - netmask) = number of IPs`

| CIDR | Calculation | Number of IPs |
|------|-------------|---------------|
| 10.0.0.0/24 | 2^(32-24) = 2^8 | **256 IPs** (10.0.0.0 → 10.0.0.255) |
| 10.0.0.0/26 | 2^(32-26) = 2^6 | **64 IPs** (10.0.0.0 → 10.0.0.63) |
| 10.0.0.0/16 | 2^(32-16) = 2^16 | **65,536 IPs** |

### Example — Splitting VPC into 2 Subnets
```
VPC: 10.0.0.0/24 → 256 IPs total

  Subnet 1: 10.0.0.0/25  → 128 IPs (10.0.0.0 → 10.0.0.127)
  Subnet 2: 10.0.0.128/25 → 128 IPs (10.0.0.128 → 10.0.0.255)
```

> ⚠️ Two subnets inside the same VPC **cannot have the same CIDR** — system will throw an error

---

## 🌐 What is a Subnet?

- Subnet = **Sub-network inside VPC**
- We divide the main VPC network into smaller subnets
- Subnets work at **Availability Zone level**
- Each subnet belongs to **one AZ only**

### Public vs Private Subnet

| | Public Subnet | Private Subnet |
|--|---------------|----------------|
| Internet access | ✅ Yes (via IGW) | ❌ No direct access |
| Has public IP | ✅ Yes | ❌ No |
| Use case | Bastion host, Load Balancer | App servers, Databases |
| Security | Less secure | More secure |

> ✅ **Best practice:** Always deploy your **applications in private subnets**  
> End users access them through a **Load Balancer** in the public subnet

---

## 🛠️ Steps to Create a Public Subnet

```
1. Create VPC (select region, assign CIDR e.g. 10.0.0.0/16)
2. Create Subnet (inside VPC, assign CIDR e.g. 10.0.0.0/24)
3. Create Internet Gateway (IGW) → attach to VPC
4. Create Route Table (RT) inside same VPC
5. Add route: 0.0.0.0/0 → IGW  (in RT)
6. Associate Route Table with Subnet
7. Launch EC2 inside subnet → enable Auto-assign Public IP
```

---

## 📌 Key Takeaways

- VPC = your private network in AWS (region-level)
- Subnet = division of VPC (AZ-level)
- CIDR decides how many IPs are available
- Use `/16` for large VPCs, `/24` or `/26` for subnets
- Never use same CIDR for two subnets in same VPC
- Public subnet = has route to IGW, private subnet = does not
