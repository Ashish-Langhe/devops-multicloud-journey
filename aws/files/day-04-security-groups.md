# Day 04 — Security Groups, Firewall & Protocols

> 📅 **Date:** 15th April  
> 🏷️ **Topic:** Firewall, Security Groups, Inbound/Outbound Rules, Ports, HTTP/HTTPS, Protocols

---

## 🔥 What is a Firewall?

> **Firewall = Security layer that allows authorised requests and blocks unauthorised requests**

In AWS → Firewall is called a **Security Group (SG)**

```
┌─────────────────────────────────────────────────┐
│                                                 │
│   Laptop ──── request ───→  [ SG / Firewall ]  │
│                               ↓         ↓      │
│                           ALLOW ✅   BLOCK ❌   │
│                               ↓               │
│                           EC2 Server           │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 🛡️ Security Group (SG)

### Key Points
- SG works at the **server (EC2 instance) level**
- You define **rules** specifying what traffic is allowed
- Every rule contains: **Source · Protocol · Port · Destination**
- SG has **two types of rules** — Inbound and Outbound

### Inbound vs Outbound

```
                    EC2 Server
                   ┌──────────┐
  Internet ──────→ │ INBOUND  │  ← Traffic coming IN to server
                   │          │
                   │ OUTBOUND │ ──────→ Internet
                   └──────────┘    Traffic going OUT from server
```

| Rule Type | Direction | Controls |
|-----------|-----------|---------|
| **Inbound** | Internet → Server | Who can access the server |
| **Outbound** | Server → Internet | What the server can send out |

---

## 🔌 What is a Port?

> **Port = a logical endpoint inside a server**

- Multiple applications can run on **one server** using different ports
- Total port range: **0 – 65535**

```
Server IP: 174.1.2.3
├── Port 3000  →  IRCTC app
├── Port 5000  →  Flipkart app
└── Port 6000  →  Amazon app
```

### Common Well-Known Ports
| Port | Protocol | Use |
|------|----------|-----|
| **22** | SSH | Linux server login |
| **3389** | RDP | Windows server login |
| **80** | HTTP | Web traffic (unsecured) |
| **443** | HTTPS | Web traffic (secured) |
| **3306** | MySQL | Database |
| **5432** | PostgreSQL | Database |

> ⚠️ **SG only checks if port 22 is open or not — it does NOT validate whether the key is correct**  
> If port 22 is blocked → Connection Timeout  
> If port 22 is open but wrong key → Permission Denied

---

## 🌐 What is a Protocol?

> **Protocol = Set of rules for data communication — defines how data is transferred**

### Reading a URL
```
http://192.3.4.5:5000
  ↑        ↑       ↑
protocol  server   port
(how)    (where)  (which app)
```

### HTTP vs HTTPS

```
HTTP  (Hyper Text Transfer Protocol)
──────────────────────────────────────
Data travels as PLAIN TEXT
Sender → "134617281719" → Receiver
Anyone intercepting can READ the data ❌

HTTPS  (Hyper Text Transfer Protocol Secure)
─────────────────────────────────────────────
Data travels as ENCRYPTED text
Sender → "trsqycvauhsijabibdi..." → Receiver → Decrypted
Interceptor sees garbage text only ✅
```

| Feature | HTTP | HTTPS |
|---------|------|-------|
| Full form | Hyper Text Transfer Protocol | Hyper Text Transfer Protocol Secure |
| Data | Plain text | Encrypted |
| Port | 80 | 443 |
| Security | ❌ Not secure | ✅ Secure |
| Use in production | ❌ Avoid | ✅ Always use |

---

## 🔒 SG Rule Structure

```
┌──────────────────────────────────────────────────────┐
│  SG Rule = Source + Protocol + Port + Destination    │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Example:                                            │
│  Source: 0.0.0.0/0  (any IP)                        │
│  Protocol: TCP                                       │
│  Port: 8502                                          │
│  → Allow ALL sources to reach this server on 8502   │
│                                                      │
│  Example 2:                                          │
│  Source: 63.2.34.5/32  (one specific IP)            │
│  Protocol: TCP                                       │
│  Port: 6000                                          │
│  → Allow ONLY this IP to reach port 6000            │
└──────────────────────────────────────────────────────┘
```

### Inbound Rule Examples (AWS Console)
| Type | Protocol | Port Range | Source | Effect |
|------|----------|-----------|--------|--------|
| SSH | TCP | 22 | 0.0.0.0/0 | Anyone can SSH |
| SSH | TCP | 22 | 63.2.34.5/32 | Only your IP can SSH ✅ |
| HTTP | TCP | 80 | 0.0.0.0/0 | Anyone can access web |
| All traffic | All | All | 0.0.0.0/0 | ⚠️ Dangerous — allow all |

> 💡 **Best practice:** Never open port 22 to `0.0.0.0/0` in production. Whitelist your specific IP only.

---

## 🧪 Troubleshooting Use Cases (from class)

### Use Case 1 — SG rule disabled
```
Developer → SSH to server (port 22 blocked in SG)
Result: Network error: Connection timed out ❌
Reason: SG blocked the packet before reaching server
```

### Use Case 2 — Wrong key used
```
Developer → SSH to server (port 22 allowed, wrong .pem key)
Result: Server refused our key / Permission denied ❌
Reason: SG passed, but server rejected the authentication
```

> Key insight: **SG is checked FIRST, then the key is validated**

---

## 📐 SG Architecture Diagram

```
AWS
┌─────────────────────────────────────────┐
│                                         │
│   Developer's Laptop (174.1.2.3)        │
│         │                               │
│         │  request (inbound)            │
│         ↓                               │
│   ┌───────────┐                         │
│   │    SG     │  ← firewall layer       │
│   └─────┬─────┘                         │
│         │ allowed ✅                    │
│         ↓                               │
│   ┌───────────┐                         │
│   │   EC2     │  ← your server          │
│   │  :3000    │  ← IRCTC                │
│   │  :5000    │  ← Flipkart             │
│   │  :6000    │  ← Amazon              │
│   └───────────┘                         │
│         │                               │
│         │  response (outbound)          │
│         ↓                               │
│   End User / Internet                   │
└─────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| Security Group | Firewall at EC2 level — controls inbound & outbound traffic |
| Inbound rule | Controls who can access your server |
| Outbound rule | Controls what your server can send out |
| Port | Logical endpoint — each app runs on a different port |
| `0.0.0.0/0` | Any IP from anywhere — use carefully |
| HTTP vs HTTPS | HTTP = plain text, HTTPS = encrypted — always use HTTPS in prod |
| SG & key | SG checks port → then key is verified. Two separate checks |
| Port 22 | SSH login for Linux — always restrict to your specific IP |

---

> 📎 **Next:** Day 05 — TCP/UDP Protocols, VPC Deep Dive & CIDR Calculation
