# Day 14 | Auto Scaling Group (ASG) & Dynamic Scaling

**Date:** 2nd May  
**Topics:** ASG, Horizontal vs Vertical Scaling, AMI, Launch Template

---

## 📈 What is Auto Scaling Group (ASG)?

- ASG = **Dynamic Scaling System** inside VPC
- Automatically **adds servers when load increases** and **removes when load decreases**
- Based on **horizontal scaling**

> "By using ASG we can add or remove servers automatically based on load"

---

## 🔼 Vertical vs Horizontal Scaling

### Vertical Scaling ❌ (Not Recommended)
- Upgrade the **same server** (e.g., t2.micro → t2.large)
- **Must stop** the server before changing instance type
- App is **unavailable** during upgrade
- Not suitable for **24/7 apps**

```
EC2 (t2.micro) → stop → upgrade → t2.large → restart
```

### Horizontal Scaling ✅ (Highly Recommended)
- **Add more servers** instead of upgrading the same one
- Existing server **keeps running** — no downtime
- Traffic distributed across multiple servers
- Best for **24/7 production apps**

```
EC2 (app) → load increases → add EC2 (app) → add EC2 (app)
```

> 💡 ASG follows **horizontal scaling**

---

## ⚡ Types of ASG

| Type | Trigger | Use Case |
|------|---------|---------|
| **Static / Schedule-based** | Time-based (e.g., 10am add servers, 11pm remove) | Known peak hours |
| **Dynamic** | Load-based (CPU %, memory, etc.) | Unpredictable traffic |

---

## 🧩 ASG Challenges (What it needs to solve)

| Challenge | Solution |
|-----------|---------|
| New server must have same application | **Custom AMI** |
| New server must be in same Target Group | Configured in ASG settings |
| New server must be in same AZs as ALB | Configured in ASG subnets |
| New server must have same configuration | **Launch Template (LT)** |

---

## 📸 AMI (Amazon Machine Image)

### What is AMI?
- AMI = **Snapshot/Backup of an EC2 instance**
- Contains: **OS + Applications + Disk data**
- Used to **create identical servers** quickly

### Why Custom AMI?
- Default AMI only has OS
- Custom AMI captures your EC2 with app already installed
- When ASG creates new server → it uses your custom AMI → app is ready instantly

### How to create Custom AMI
```
1. Launch EC2 → install your app (e.g., Nginx + Flipkart app)
2. Go to EC2 → Actions → Image and templates → Create Image
3. Name your AMI
4. AMI is saved → now use it in Launch Template
```

---

## 📋 Launch Template (LT)

### What is Launch Template?
- LT = **Blueprint for creating EC2 instances**
- Stores configuration: AMI, Instance Type, Key Pair, SG, etc.

| Component | Responsible |
|-----------|-------------|
| AMI | Application + OS |
| Launch Template | All configurations (instance type, key, SG, subnet, etc.) |

### What Launch Template stores
- Custom AMI ID
- Instance type (e.g., t2.micro)
- Key Pair name
- Security Group
- ⚠️ **Do NOT select AZ/Subnet in Launch Template** — set it in ASG only

---

## 🔧 ASG Setup Process

```
Step 1: Create EC2 with your app installed
Step 2: Create Custom AMI from that EC2
Step 3: Create Launch Template using custom AMI
Step 4: Create ASG:
         → Select Launch Template
         → Select Subnets (private, same AZs as ALB)
         → Attach to Target Group (same TG as ALB)
         → Set scaling limits: Min / Desired / Max
         → Set scaling policy: Target CPU (e.g., 50%)
Step 5: Test by stressing the server
```

---

## ⚙️ ASG Scaling Settings

| Setting | Meaning |
|---------|---------|
| **Min desired** | Minimum servers always running |
| **Desired** | Default number of servers |
| **Max desired** | Maximum servers allowed |
| **Target CPU** | Scale out if CPU > this % |
| **Warmup period** | Default 300 seconds (5 mins) before new server is counted |

---

## 🧪 Testing ASG (Stress Test)

```bash
# Install stress tool on EC2
yum install stress -y

# Run CPU stress for 60 seconds
stress --cpu 2 --timeout 60
```

→ Watch ASG create new servers automatically in AWS Console!

---

## 📌 Key Takeaways

- ASG = automatically adds/removes servers based on load
- Always use **horizontal scaling** for 24/7 apps
- Custom AMI = snapshot with your app baked in
- Launch Template = server blueprint (AMI + config)
- Never select subnet inside Launch Template — set it in ASG
- ASG + ALB + TG together = fully automated scalable architecture
