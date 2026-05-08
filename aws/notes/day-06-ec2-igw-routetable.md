# Day 06 — Key Pair, Internet Gateway, Route Table & Public Subnet

> 📅 **Date:** 17th April  
> 🏷️ **Topic:** Key Pair authentication, Internet Gateway, Route Table, Creating a Public Subnet

---

## 🔑 Key Pair — Server Authentication

> **Key Pair = the mechanism used to authenticate and log into your EC2 server**

### How it works
```
Key Pair creates TWO keys:

  Public Key   →  stored ON the EC2 server  (destination end)
  Private Key  →  downloaded to YOUR laptop (source end)
                  NEVER share this with anyone
```

```
Your Laptop                           EC2 Server
┌──────────────────┐                ┌──────────────┐
│  privatekey.pem  │ ──── SSH ────→ │  public key  │
│  (keep secret)   │                │  (on server) │
└──────────────────┘                └──────────────┘
```

### Key file formats
```
.pem  →  for OpenSSH / Terminal (Mac, Linux)
.ppk  →  for PuTTY (Windows)
```

> ⚠️ **Download the private key ONCE only** — at creation time.  
> If you lose it, you permanently lose SSH access to that server.

### How to create a Key Pair in AWS
```
EC2 Console → Network & Security → Key Pairs
  → Create key pair
  → Name: loginsh  (any name)
  → Type: RSA
  → Format: .pem  (OpenSSH) or .ppk (PuTTY)
  → Create → .pem downloads automatically to your laptop
```

---

## 🌍 Internet Gateway (IGW)

> **IGW = the door between your VPC and the public internet**

- Attached at the **VPC level**
- Without IGW → even if you launch an EC2 with a public IP, it is unreachable
- Just creating IGW is not enough → you must **attach it to VPC** AND **add a route in the Route Table**

```
Internet
    ↓
Internet Gateway (IGW)
    ↓  ← attached to VPC
VPC (10.0.0.0/16)
    └── Public Subnet
          └── EC2 (now reachable from internet ✅)
```

---

## 🗺️ Route Table (RT)

> **Route Table = Router for the VPC — tells traffic WHERE to go**

- Every VPC gets a **default route table** automatically
- You create a **custom route table** for your public subnet
- Route table entries define where each traffic type is forwarded

### Route Table Rules
| Destination | Target | Meaning |
|-------------|--------|---------|
| `10.0.0.0/16` | local | Traffic within VPC stays inside VPC |
| `0.0.0.0/0` | IGW | All other traffic → goes to internet |

```
0.0.0.0/0  =  ANY IP address  =  internet traffic

Adding  0.0.0.0/0 → IGW  to a Route Table
  = all internet-bound traffic goes out through IGW
  = this is what makes a subnet PUBLIC
```

---

## 🏗️ What Makes a Subnet "Public" or "Private"?

```
The subnet itself has NO inherent nature.
The ROUTE TABLE attached to it defines its behaviour.

Public Subnet  →  Route Table HAS  0.0.0.0/0 → IGW
Private Subnet →  Route Table does NOT have an internet route
```

| | Public Subnet | Private Subnet |
|--|--------------|----------------|
| Route to IGW | ✅ Yes | ❌ No |
| EC2 gets Public IP | ✅ Can be enabled | ❌ No |
| Reachable from internet | ✅ Yes | ❌ No |
| Can reach internet | ✅ Yes | ❌ No (needs NAT) |
| Use case | Bastion, Load Balancer, NAT GW | App servers, Databases |

---

## 🛠️ Creating a Public Subnet — Step by Step

> Exact steps from class — numbered to match the architecture diagram:

```
STEP ①  Create VPC
          Select your region
          CIDR: 10.0.0.0/16

STEP ②  Create EC2 (inside the subnet you will create)
          Instance type, SG, select VPC

STEP ③  Create Internet Gateway → Attach to VPC

STEP ④  Create Route Table inside the same VPC

STEP ⑤  Add Route inside Route Table
          Destination: 0.0.0.0/0
          Target: Internet Gateway (select your IGW)
          → Edit routes → Save changes

STEP ⑥  Associate Subnet with Route Table
          → Subnet associations → Edit subnet associations
          → Select your subnet → Save

STEP ⑦  Enable Auto-assign Public IP on the subnet
          Subnet settings → Enable auto-assign public IPv4 address
          → Now EC2 launched in this subnet gets a public IP ✅
```

### Architecture — What these steps build

```
VPC: 10.0.0.0/16
┌──────────────────────────────────────────────────┐
│                                                  │
│                     ③ IGW                        │
│                       ↕  ⑤ edit routes          │
│                     ④ RT                         │
│               (0.0.0.0/0 → IGW)                 │
│          ⑥ subnet association                   │
│  ┌───────────────────────────┐  ┌─────────────┐ │
│  │  Public Subnet            │  │   Private   │ │
│  │  10.0.0.0/24              │  │   Subnet    │ │
│  │  ② EC2  [SG] ⑦          │  │  EC2 (app)  │ │
│  │  public server            │  │  Flipkart   │ │
│  └───────────────────────────┘  └─────────────┘ │
│        ↑ internet reachable       ✗ isolated     │
└──────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| Key Pair | Public key on server, private key on your laptop — download once only |
| `.pem` | For Mac/Linux terminal (OpenSSH) |
| `.ppk` | For Windows PuTTY |
| IGW | Door between VPC and internet — must be attached to VPC |
| Route Table | Router for VPC — defines where traffic goes |
| `0.0.0.0/0 → IGW` | The single rule that makes a subnet public |
| Public subnet | Subnet whose RT has a route to IGW |
| Private subnet | Subnet whose RT has NO internet route |
| Subnet association | RT must be explicitly linked to subnet to take effect |
| Auto-assign public IP | Must be enabled on subnet so EC2 gets a public IP |

---

> 📎 **Next:** Day 07 — Custom Networking Hands-On, Public & Private Subnets, SSH Connection
