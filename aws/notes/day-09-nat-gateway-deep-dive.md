# Day 09 — NAT Gateway Deep Dive

> 📅 **Date:** 22nd April  
> 🏷️ **Topic:** NAT Gateway internals, Stateful behaviour, Port mapping, Security model, Architecture

---

## 🔄 What is NAT Gateway?

> **NAT = Network Address Translator**  
> **NAT Gateway = AWS managed service that translates private IPs to public IPs for outbound traffic**

### Simple Analogy
```
Imagine a company office building:
  Employees (private IPs) cannot directly talk to the outside world
  They go through a receptionist (NAT Gateway)
  Receptionist has one public phone number
  Receptionist makes calls on behalf of employees
  Outside world only knows the receptionist's number, not the employees'
```

---

## 🔑 Why NAT Exists

Private servers have:
- ✅ Private IP (`10.0.1.4`) — internal to VPC
- ❌ No public IP
- ❌ No route to internet

But they need internet to:
- Install packages (`yum install`, `apt install`)
- Download dependencies
- Pull code from GitHub
- Send outbound API calls

```
Without NAT:
  Private EC2 → ping google.com → ❌ No route to host

With NAT:
  Private EC2 → NAT → Internet → response back → Private EC2 ✅
```

---

## ⚙️ How NAT Works — Step by Step

```
Step 1: Private server sends outbound request
        10.0.1.4  →  ping google.com

Step 2: Traffic reaches NAT Gateway (in public subnet)
        NAT has its own public IP: 174.2.3.5

Step 3: NAT TRANSLATES the source IP
        10.0.1.4  →  174.2.3.5
        (private becomes public)

Step 4: Request reaches Google
        Google sees: request from 174.2.3.5

Step 5: Google responds to 174.2.3.5

Step 6: NAT translates back
        174.2.3.5  →  10.0.1.4
        (public back to private)

Step 7: Private server receives response ✅
```

### Visual
```
VPC
┌────────────────────────────────────────────────────┐
│                                                    │
│  Private Subnet                                    │
│  ┌────────────────┐                               │
│  │  EC2           │  10.0.1.4                     │
│  │  (Flipkart app)│ ──→ outbound request          │
│  └────────────────┘         ↓                     │
│                         RT (pvt)                  │
│                             ↓                     │
│  Public Subnet              ↓                     │
│  ┌────────────────┐         ↓                     │
│  │  NAT Gateway   │ ←───────┘                     │
│  │  IP: 174.2.3.5 │ ──→  174.2.3.5 → Internet    │
│  └────────────────┘         ↑                     │
│                     response comes back           │
│                         174.2.3.5 → 10.0.1.4     │
└────────────────────────────────────────────────────┘
```

---

## 🧠 NAT is STATEFUL

> **Stateful = NAT tracks the state of every connection it makes**

### What does "stateful" mean?
```
NAT maintains a CONNECTION TRACKING TABLE:

  Private IP  | Private Port | Public IP    | Public Port
  ─────────────────────────────────────────────────────
  10.0.1.5    | 5000         | 52.10.20.30  | 30001
  10.0.1.6    | 5001         | 52.10.20.30  | 30002

When a response arrives at 52.10.20.30:30001
  → NAT looks up table → routes to 10.0.1.5:5000 ✅

When a response arrives at 52.10.20.30:30002
  → NAT looks up table → routes to 10.0.1.6:5001 ✅
```

### Why statefulness matters
- NAT uses the **SAME public IP** for ALL private servers
- Port mapping differentiates which server gets which response
- No conflicts even if multiple servers ping the same website simultaneously

---

## 🔌 Port Mapping — Multiple Servers, One Public IP

```
Two private servers, one NAT, one public IP:

  Server-1 (10.0.1.4) → pings google.com
  Server-2 (10.0.1.5) → pings facebook.com

NAT maps:
  10.0.1.5:5000 → 52.10.20.30:30001  (google connection)
  10.0.1.6:5001 → 52.10.20.30:30002  (facebook connection)

Response for google → port 30001 → routed to 10.0.1.5 ✅
Response for facebook → port 30002 → routed to 10.0.1.6 ✅

No conflicts. Port range: 0–65535 ← plenty of capacity
```

---

## 🔒 NAT Security Model — Outbound Only

> **NAT allows ONLY EGRESS (outbound) traffic**

```
✅ ALLOWED — Outbound initiated by private server:
  10.0.1.5 → NAT → 52.10.20.30 → Internet
  Internet → 52.10.20.30 → NAT → 10.0.1.5 (response)

❌ BLOCKED — Inbound initiated from internet:
  Hacker (192.3.4.6) → tries → NAT → 10.0.1.4
  NOT POSSIBLE — NAT will not forward inbound connections
```

### Key Security Points
```
┌────────────────────────────────────────────────────────┐
│  NAT Security Properties                               │
│                                                        │
│  ✅ Private IP never leaves your VPC                  │
│  ✅ NAT uses public IP on behalf of your server       │
│  ✅ Keeps a connection tracking table                 │
│  ✅ Works ONLY for outbound traffic                   │
│  ❌ Inbound connections from internet: BLOCKED        │
│  ❌ Attackers CANNOT initiate connection to           │
│     private servers through NAT                       │
└────────────────────────────────────────────────────────┘
```

---

## 🆚 NAT vs Internet Gateway

| Feature | Internet Gateway (IGW) | NAT Gateway |
|---------|----------------------|-------------|
| Direction | Both inbound + outbound | Outbound only |
| Who uses it | Public subnet EC2 | Private subnet EC2 |
| EC2 needs public IP | ✅ Yes | ❌ No |
| Security | Less secure (bidirectional) | More secure (outbound only) |
| Placement | Attached to VPC | In public subnet |
| Purpose | Make subnet public | Give private servers internet |

---

## 📍 Where to Place NAT Gateway

```
✅ CORRECT: NAT in PUBLIC subnet
┌──────────────────────────────────────────────────┐
│  VPC                                             │
│  ┌─────────────────┐     ┌──────────────────┐   │
│  │  Public Subnet  │     │  Private Subnet  │   │
│  │                 │     │                  │   │
│  │  IG             │     │  EC2 (app)       │   │
│  │  NAT ← here ✅  │     │  uses NAT ──────→│   │
│  │  (Bastion)      │     │  for internet    │   │
│  └─────────────────┘     └──────────────────┘   │
└──────────────────────────────────────────────────┘

Route Table for PRIVATE subnet must have:
  Destination: 0.0.0.0/0  →  Target: NAT Gateway
```

---

## 🏗️ Full NAT Architecture

```
                     Internet
                         ↑↓ (outbound responses only)
                    Internet Gateway
                         ↑↓
               VPC (10.0.0.0/16)
                         │
              ┌──────────┴──────────┐
              │                     │
         Public Subnet         Private Subnet
         ┌──────────────┐      ┌──────────────┐
         │ Bastion      │      │ EC2 (app)    │
         │ NAT Gateway  │←─────│ 10.0.1.4     │
         │ 174.2.3.5    │      │              │
         └──────────────┘      └──────────────┘
              ↑
         Routes outbound traffic
         from private EC2 to internet
         Returns responses back to private EC2
```

---

## 🔁 Recommended NAT Setup

```
From class: Attach NAT to public subnet OR at VPC level

Option 1: NAT in public subnet (most common)
  → Private subnet RT: 0.0.0.0/0 → NAT Gateway

Option 2: NAT at VPC level
  → All subnets can use it
  → More centralised
```

> 💡 **Class Tip:** "Recommended" NAT placement = public subnet. The NAT in public subnet has a public IP (Elastic IP) that it uses to communicate with internet on behalf of private servers.

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| NAT Gateway | AWS managed service — gives private EC2 outbound internet access |
| How it works | Translates private IP → public IP for outbound requests |
| Stateful | NAT tracks connections using a connection table |
| Port mapping | Uses different ports per server → no conflicts on same public IP |
| Security | Outbound ONLY — inbound connections from internet are BLOCKED |
| Private IP | Never exposed to internet — NAT uses its public IP |
| Placement | Always in PUBLIC subnet |
| Route needed | Private subnet RT: `0.0.0.0/0 → NAT Gateway` |
| IGW vs NAT | IGW = bidirectional (public subnet), NAT = outbound only (private subnet) |
| Port range | 0–65535 — enough ports for all concurrent connections |

---

> 📎 **Next:** Day 10 — Web Application Deployment (Nginx on EC2)
