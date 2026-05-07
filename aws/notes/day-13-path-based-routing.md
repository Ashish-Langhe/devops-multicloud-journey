# Day 13 | Path-Based Routing (ALB)

**Date:** 1st May  
**Topics:** Types of Load Balancers, ALB Path-Based Routing

---

## ⚖️ Two Types of Load Balancers

| Feature | ALB (Application LB) | NLB (Network LB) |
|---------|----------------------|------------------|
| Protocol | HTTP/HTTPS | TCP |
| Path-based routing | ✅ Yes | ❌ No |
| Usage | ~95% globally | High throughput use cases |
| Public subnets needed | Min 2 (different AZs) | Min 2 (different AZs) |
| Traffic method | Round Robin | — |
| Request inspection | ✅ Inspects URL/path | ❌ Just forwards TCP |

---

## 🛣️ Path-Based Routing

ALB can route traffic to **different Target Groups** based on the **URL path**.

### Real-world example (Flipkart)

```
flipkart.com         → ALB DNS
flipkart.com/        → Home TG       (EC2s with home page)
flipkart.com/electronics → Electronics TG
flipkart.com/mobiles     → Mobiles TG
flipkart.com/appliances  → Appliances TG
```

### ALB Routing Rules

| Path | Target Group |
|------|-------------|
| `/` | Home page TG |
| `/electronics/*` | Electronics TG |
| `/mobiles/*` | Mobiles TG |
| `/appliances/*` | Appliances TG |

### Architecture — 1 ALB + 4 TGs

```
User → flipkart.com → ALB DNS
              ↓
         Listener (HTTP:80)
              ↓
    ┌─────────────────────────┐
    │ Rules:                  │
    │  path=/        → TG-Home│
    │  path=/electronics → TG-E│
    │  path=/mobiles → TG-M   │
    │  path=/appliances → TG-A│
    └─────────────────────────┘
         ↓       ↓      ↓       ↓
      TG-Home  TG-E  TG-M   TG-A
      (EC2x2)(EC2x2)(EC2x2)(EC2x2)
```

---

## 🛠️ How to Set Up Path-Based Routing

### Step 1: Deploy apps on each server

```bash
# On server for /aws path:
sudo su -
yum install nginx -y
systemctl start nginx
cd /usr/share/nginx/html/
mkdir aws
cd aws/
vi index.html   # add AWS content
systemctl restart nginx
```

### Step 2: Add rules in ALB Listener

In AWS Console:
1. Go to **EC2 → Load Balancers → your ALB**
2. Click **Listeners → HTTP:80 → View/Edit rules**
3. Add rule:
   - **Condition:** Path = `/aws/*`
   - **Action:** Forward to TG-AWS
4. Repeat for each path

### Step 3: Test

```
http://alb-dns.us-east-1.elb.amazonaws.com/        → Home
http://alb-dns.us-east-1.elb.amazonaws.com/aws/    → AWS page
http://alb-dns.us-east-1.elb.amazonaws.com/azure/  → Azure page
http://alb-dns.us-east-1.elb.amazonaws.com/gcp/    → GCP page
```

---

## 📁 Nginx Path Folder Structure on Server

```bash
/usr/share/nginx/html/
├── index.html        ← served at /
├── aws/
│   └── index.html    ← served at /aws/
├── azure/
│   └── index.html    ← served at /azure/
└── gcp/
    └── index.html    ← served at /gcp/
```

---

## 📌 Key Takeaways

- ALB = HTTP-based, supports path routing = best for web apps (95% usage)
- NLB = TCP-based, no path routing = best for high throughput, low latency
- Path-based routing: 1 ALB can serve multiple services/apps using rules
- ALB inspects the URL path and routes to the correct Target Group
- Each path has its own TG → each TG has its own EC2 servers

---

## 🔗 Practice Repo

Clone for path-based routing hands-on:
```
github.com/CloudTechDevOps/Pathbased-routing
```
