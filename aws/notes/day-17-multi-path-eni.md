# Day 17 | Multi-Path on Single Server & ENI

**Date:** 6th May  
**Topics:** Multiple apps on one EC2 using paths/ports, ENI (Elastic Network Interface)

---

## 🧪 Task — 3 Apps on 1 Server, 3 TGs, 1 ALB

### Setup
Deploy **3 applications** on a **single EC2** using different paths:

```
/usr/share/nginx/html/index.html        ← default path (port 80)
/usr/share/nginx/html/path1/index.html  ← path1
/usr/share/nginx/html/path2/index.html  ← path2
```

### Questions to answer:
```
LB dns         → redirects to which path?  → default (/)
LB dns/path1   → redirects to which path?  → path1
LB dns/path2   → redirects to which path?  → path2
```

---

## 🏗️ Architecture — 1 ALB + Multiple TGs + 1 EC2

### Option A — Same port (80), different paths
```
User → ALB Listener (HTTP:80)
         ↓
    ┌────────────────────┐
    │  Rules:            │
    │  /        → TG-1   │  (health check: /)
    │  /path1/* → TG-2   │  (health check: /path1)
    │  /path2/* → TG-3   │  (health check: /path2)
    └────────────────────┘
         ↓       ↓      ↓
       TG-1    TG-2   TG-3
          \      |     /
           \     |    /
          Single EC2 (port 80)
          /usr/share/nginx/html/
          ├── index.html
          ├── path1/index.html
          └── path2/index.html
```

> ⚠️ LB performs health checks only on the **default path (/)** — make sure `/` responds correctly

### Option B — Different ports for each path
```
ALB Listener (HTTP:80)
    ↓
TG-1 → port 80  → /usr/share/nginx/html/index.html
TG-2 → port 3000 → /usr/share/nginx/html/path1/index.html
TG-3 → port 5000 → /usr/share/nginx/html/path2/index.html
```

Each TG targets the same EC2 but on **different ports** — and different apps can run on those ports.

---

## 🛠️ Setup Commands

```bash
sudo su -
yum install nginx -y
systemctl start nginx
systemctl enable nginx

# Create path directories
cd /usr/share/nginx/html/
mkdir path1 path2

# Default page
vi index.html        # add: <h1>Default Home Page</h1>

# Path1 page
vi path1/index.html  # add: <h1>Path 1 Page</h1>

# Path2 page
vi path2/index.html  # add: <h1>Path 2 Page</h1>

systemctl restart nginx

# Verify locally
curl http://localhost/
curl http://localhost/path1/
curl http://localhost/path2/
```

---

## 🖥️ OS Challenges — Multiple Apps on One Server

When you deploy 3–4 apps on the same OS, common problems arise:

| # | Problem |
|---|---------|
| 1 | **Dependency issues** — App A needs Python 3.8, App B needs Python 3.11 |
| 2 | **Version conflicts** — different apps need different versions of same library |
| 3 | **Hardware management is difficult** — hard to allocate CPU/RAM per app |
| 4 | **Security management is difficult** — one compromised app can affect others |
| 5 | **Compatibility issues** — OS-level configs conflict between apps |
| 6 | **Can't scale individual apps** — scaling the server scales everything, not just one app |

> 💡 This is exactly why **Docker containers** exist — each app gets its own isolated environment on the same OS. Coming in Phase 2!

---

## 🔌 ENI — Elastic Network Interface

### What is ENI?
- ENI = **Virtual NIC (Network Interface Card)** attached to an EC2 instance
- Every EC2 has at least one ENI by default
- ENI holds: **SG details + Subnet details + Private IP**

```
Subnet
  └── EC2 Instance
        └── ENI (NIC Card)
              ├── Private IP
              ├── Security Group
              └── Subnet association
```

### Why does ENI matter?

ENI allows you to **preserve a private IP** even after an EC2 is terminated.

### Key Rules

| Question | Answer |
|----------|--------|
| Can I attach one EC2's ENI to another EC2 while it's running? | ❌ NO |
| Can I detach/delete ENI while server is running? | ❌ NO |
| Can I transfer ENI after terminating the server? | ✅ YES |

### Use Case — Preserving Private IP

**Scenario:** Your app is hardcoded to use IP `10.0.1.50`. Server gets corrupted.

**Solution with ENI:**

**Option 1:**
```
1. Terminate corrupted server → private IP released from ENI
2. Create new ENI manually with same IP (10.0.1.50)
3. Launch new EC2 using that ENI
4. New server gets same IP ✅
```

**Option 2:**
```
1. Before terminating → disable "Delete ENI on termination" setting
2. Terminate server → ENI stays alive with IP intact
3. Launch new EC2 → attach the preserved ENI
4. New server gets same private IP ✅
```

---

## 📌 Key Takeaways

- 1 ALB can route to multiple TGs on the same EC2 using path rules
- LB health check runs on **default path only** — ensure `/` always responds
- Multiple apps on one OS = dependency/version conflicts → Docker solves this
- ENI = virtual NIC that holds the private IP of an EC2
- ENI can be preserved after server termination to keep the same IP on a new server
- Cannot detach or delete ENI while server is running
