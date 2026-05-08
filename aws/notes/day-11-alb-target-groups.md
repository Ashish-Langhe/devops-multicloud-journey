# Day 11 — Application Load Balancer (ALB) & Target Groups

> 📅 **Date:** 29th April  
> 🏷️ **Topic:** ALB, Target Groups, Load Balancer features, AZ matching, health checks, Round Robin

---

## ❓ The Problem — How Do End Users Access Private Servers?

```
App is deployed on private EC2 (correct & secure)
  ↓
End user types yourapp.com in browser
  ↓
Private EC2 has NO public IP, NOT reachable directly ❌
  ↓
Solution → Load Balancer ✅
```

---

## ⚖️ What is a Load Balancer?

> **Load Balancer = sits in public subnet, receives all traffic, forwards it to private servers**

```
End User
    ↓
Internet
    ↓
Load Balancer  ← public facing (has public DNS)
    ↓  forwards request
Private EC2 (subnet 1a)  ← app running here
Private EC2 (subnet 1b)  ← app running here
```

- App servers are **never exposed** to the internet directly
- LB acts as a **proxy** — users only see the LB DNS, not the server IP

---

## 🏗️ ALB — Application Load Balancer

### Hard Requirement
```
┌──────────────────────────────────────────────────────┐
│  ALB needs MINIMUM 2 PUBLIC subnets                  │
│  → Must be in DIFFERENT AZs (not same AZ)            │
│                                                      │
│  ✅  subnet-1a (AZ: us-east-1a)                     │
│  ✅  subnet-1b (AZ: us-east-1b)                     │
│  ❌  subnet-1a + subnet-1a (same AZ — not allowed)  │
└──────────────────────────────────────────────────────┘
```

### ALB Architecture
```
                    Internet
                        ↓
              Application Load Balancer
              (public subnet 1a + 1b)
                    /         \
           subnet 1a           subnet 1b
          ┌──────────┐        ┌──────────┐
          │ Private  │        │ Private  │
          │ EC2      │        │ EC2      │
          │ (app)    │        │ (app)    │
          └──────────┘        └──────────┘
              Target Group (TG)
```

---

## 🎯 Target Groups (TG)

> **Target Group = the list of servers the Load Balancer sends traffic to**

### Why Target Groups?
```
Example:
  us-east-1 region has 4 servers running
  But LB should only send traffic to 2 specific servers

  → Create a TG
  → Add only those 2 servers to TG
  → LB sends traffic to only those servers ✅
  → Not to all 4 servers in the region
```

```
Load Balancer
    ↓
Target Group (TG-1)
    ├── Private EC2 (subnet 1a)  ← app here
    └── Private EC2 (subnet 1b)  ← app here
```

### TG Creation in AWS Console
```
EC2 → Load Balancing → Target Groups → Create target group
  Target group name: TG-1
  Protocol: HTTP
  Port: 80
  IP address type: IPv4
→ Register your private EC2 instances as targets
```

---

## ⚠️ Critical — AZ Matching Rule

> **LB AZs and Target Server AZs MUST match**

```
Scenario:
  LB is configured with: us-east-1a + us-east-1b
  Server is running in: us-east-1c
  Server is added to TG ✅

  BUT → LB cannot send traffic to us-east-1c ❌
  Because us-east-1c was NOT selected in LB

Rule:
  LB network-level AZ's must match target server AZ's
  Only then TG-level traffic will start flowing
```

| LB AZ selected | Server AZ | Traffic flows? |
|---------------|-----------|---------------|
| 1a, 1b | 1a | ✅ Yes |
| 1a, 1b | 1b | ✅ Yes |
| 1a, 1b | 1c | ❌ No — AZ mismatch |

---

## 🔧 Load Balancer Features

```
┌──────────────────────────────────────────────────────────────┐
│  Load Balancer Features                                      │
│                                                              │
│  1. Load distribution                                        │
│     Balances traffic — reduces load on single server         │
│                                                              │
│  2. Failover routing                                         │
│     If a server fails → LB stops sending traffic to it      │
│     Sends traffic to remaining healthy servers only          │
│                                                              │
│  3. Health checks                                            │
│     LB checks if app is healthy before sending traffic       │
│     Only healthy servers receive traffic                     │
│                                                              │
│  4. Security (Proxy)                                         │
│     LB acts as proxy → server and app NOT exposed to public  │
│     End user only sees LB DNS — never the server IP          │
│                                                              │
│  5. High Availability                                        │
│     If any server fails → traffic goes to healthy ones       │
│     No downtime for end users                                │
│                                                              │
│  6. Round Robin (default)                                    │
│     Traffic sent equally to each server in rotation          │
│     Request 1 → Server A                                     │
│     Request 2 → Server B                                     │
│     Request 3 → Server A  (back to start)                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 🏥 Health Checks

> LB constantly checks every target server — only sends traffic to healthy ones

```
LB → sends health check request → EC2 app
  ✅ App responds → server is HEALTHY → LB sends traffic
  ❌ App doesn't respond → server is UNHEALTHY → LB stops traffic

Health check settings:
  Healthy threshold:   2 (2 successes = healthy)
  Unhealthy threshold: 2 (2 failures = unhealthy)
  Timeout:             6 seconds
  Interval:            30 seconds
```

---

## 🏗️ Full ALB Architecture

```
                     End Users
                          ↓
                     Internet
                          ↓
              ┌───────────────────────┐
              │  Application LB (ALB) │
              │  public subnet 1a+1b  │
              │  Has: Listener (80)   │
              │  Has: SG              │
              └────────┬──────────────┘
                       ↓  forwards to TG
              ┌────────────────────┐
              │   Target Group     │
              │   (TG-1)           │
              └──────┬─────────────┘
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
   Private Subnet 1a      Private Subnet 1b
   ┌──────────────┐        ┌──────────────┐
   │  EC2 (app)   │        │  EC2 (app)   │
   │  Flipkart    │        │  Flipkart    │
   └──────────────┘        └──────────────┘
```

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| Load Balancer | Sits in public subnet, routes traffic to private servers |
| ALB | Application LB — works on HTTP — needs min 2 public subnets in different AZs |
| Target Group | List of servers LB should route traffic to |
| AZ rule | LB AZs and server AZs must match — otherwise no traffic flows |
| Round Robin | Default method — sends traffic to each server equally in rotation |
| Health check | LB checks app health — only healthy servers get traffic |
| Proxy | LB works as proxy — server IP never exposed to public |
| Failover | If server fails → LB routes to remaining healthy servers only |

---

> 📎 **Next:** Day 12 — ALB Hands-On, Elastic IP & Private Server Configuration
