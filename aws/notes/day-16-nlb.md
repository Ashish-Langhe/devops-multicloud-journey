# Day 16 | Network Load Balancer (NLB)

**Date:** 5th May  
**Topics:** NLB, NLB vs ALB, NLB + ALB integration, ASG warmup period

---

## 🌐 Network Load Balancer (NLB)

### Key Characteristics

| Feature | Detail |
|---------|--------|
| Protocol | **TCP** |
| Path-based routing | ❌ Not supported |
| Latency | Very **low latency** |
| Throughput | **High throughput** |
| IP | **Static/Elastic IP** (biggest advantage) |
| Speed | **Faster** than ALB |

> NLB does **NOT inspect** the request (no URL/path checking)  
> It simply forwards the TCP packet to the target → that's why no path routing

---

## 🆚 NLB vs ALB

| Feature | ALB | NLB |
|---------|-----|-----|
| Protocol | HTTP/HTTPS | TCP |
| Path-based routing | ✅ Yes | ❌ No |
| Request inspection | ✅ Inspects headers/paths | ❌ Just forwards |
| Latency | Higher | **Lower** |
| Throughput | Moderate | **High** |
| Static IP | ❌ No | ✅ Yes (via Elastic IP) |
| Use case | Web apps, microservices | Real-time apps, gaming, banks |
| Placement | Internal (private subnets) | External (public subnets) |

---

## 🏦 Why NLB? — The Static IP Advantage

Many systems require **IP whitelisting** (only specific IPs allowed):
- Firewalls
- Banks / Payment Gateways
- Partner APIs

**Problem with ALB:** ALB IP changes — you can't whitelist a changing IP  
**Solution with NLB:** NLB supports **Elastic IP** — fixed static IP

```
You give fixed IP → they whitelist → done ✅
```

---

## 🔗 NLB + ALB Integration

Best practice architecture for large-scale systems:

```
Internet
    ↓
NLB (public subnets) ← Elastic IP (static, whitelistable)
 TCP protocol
    ↓
Target Group (ALB as target)
    ↓
ALB (private subnets) ← handles HTTP routing
    ↓
Target Group (EC2 servers)
    ↓
Private EC2 instances (apps)
```

### Flow
```
Public NLB → Public TG → Private ALB → Private TG → EC2
NLB - TCP - TG - ALB - TG
```

### When to use this?
- You need **static IP** (for whitelisting) → NLB at front
- You need **path-based routing** → ALB internally
- You need **high throughput** → NLB handles raw TCP
- You need **HTTP intelligence** → ALB handles routing

---

## ⏱️ ASG Warmup Period

- **Default: 300 seconds (5 minutes)**
- When ASG creates a new server, it waits 300s before counting it as "ready"
- During warmup, the new server is not included in scaling decisions
- This prevents rapid scale-out loops

---

## 📋 Home Tasks (from class)

1. What is **Load Balancer Stickiness** and how to configure it?
2. How to set up **scheduled-based scaling** in ASG? (e.g., Mng 10:00am → create servers, 11:00am → delete)
3. What is **Lifecycle Policy** in ASG and how to integrate?
4. **NLB + ALB integration** — hands-on

---

## 📌 Key Takeaways

- NLB = TCP-based, no path routing, high throughput, low latency
- NLB's biggest advantage = **static Elastic IP** (good for whitelisting)
- ALB = HTTP-based, path routing, internal load balancing
- NLB (external, public) + ALB (internal, private) = enterprise architecture
- ASG warmup = 300s default before new server is considered active
