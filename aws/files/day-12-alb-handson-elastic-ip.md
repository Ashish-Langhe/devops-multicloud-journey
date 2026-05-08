# Day 12 — ALB Hands-On, Private Server Config & Elastic IP

> 📅 **Date:** 30th April  
> 🏷️ **Topic:** Configuring private server for ALB, Elastic IP, Listeners, health check config

---

## 🏗️ Full Production Architecture (from class)

```
                     Internet
                         ↓
              ┌──────────────────────┐
              │  External LB         │
              │  (Frontend/ALB)      │
              └──────────┬───────────┘
                         │
              ┌──────────────────────┐
              │  Internal LB         │
              │  (Backend/ALB)       │
              └──────────┬───────────┘
                         │
              ┌──────────────────────┐
              │  DB Layer            │
              │  (MongoDB, Redis)    │
              └──────────────────────┘
```

- **External ALB** → faces internet → routes to Frontend EC2s
- **Internal ALB** → private → routes to Backend EC2s
- **DB layer** → private → databases

---

## 🛠️ Step 1 — Configure Private Server (via Bastion)

> First get into the private server via Bastion, then install and configure Nginx:

```bash
# SSH into bastion first, then jump to private server
ssh -i devpractivekey.pem ec2-user@10.0.1.211  # private server

# Switch to root
sudo su -

# Install Nginx
yum install nginx -y

# Start Nginx
systemctl start nginx

# Go to web root
cd /usr/share/nginx/html/

# Edit index.html
vi index.html
# Add: <h1> first server response </h1>
# :wq! to save

# Verify locally
curl http://localhost
# → <h1> first server response </h1> ✅
```

---

## 🛠️ Step 2 — Create Target Group

```
EC2 Console → Load Balancing → Target Groups → Create target group

  Target type: Instances
  Target group name: TG-1
  Protocol: HTTP
  Port: 80
  VPC: select your VPC

Health check settings:
  Protocol: HTTP
  Health check path: /
  Healthy threshold:   2
  Unhealthy threshold: 2
  Timeout:             6 seconds
  Interval:            30 seconds

→ Next → Register your private EC2 instances → Create
```

---

## 🛠️ Step 3 — Create Application Load Balancer

```
EC2 Console → Load Balancing → Load Balancers → Create → Application Load Balancer

  Name: myalb
  Scheme: Internet-facing
  IP address type: IPv4

  Network mapping:
    VPC: your VPC
    AZ: us-east-1a → select public subnet 1a
    AZ: us-east-1b → select public subnet 1b
    (minimum 2 public subnets in DIFFERENT AZs)

  Security groups: select your SG (allow HTTP 80)

  Listeners and routing:
    Listener: HTTP:80
    Default action: Forward to TG-1

→ Create load balancer
```

---

## 🔗 Step 4 — Access via ALB DNS

```
After creation → copy ALB DNS name:
  myalb-936782470.us-east-1.elb.amazonaws.com

Open browser → http://myalb-936782470.us-east-1.elb.amazonaws.com

You should see: first server response ✅
```

> 💡 End users access via **ALB DNS name** — never via EC2 IP directly

---

## ⚙️ Listener Rules

> **Listener = the port ALB listens on and what it does with incoming traffic**

```
Listeners and rules (1)
  Protocol:Port    Default action
  HTTP:80     →    Forward to TG-1 (100%)
              →    Target group stickiness: Off
```

```
ALB Listener flow:
  User → http://alb-dns:80
    ↓  Listener on port 80
    ↓  Check rules
    ↓  Default action → Forward to TG-1
    ↓  TG-1 → EC2 (private) ✅
```

---

## 🏥 Health Check — What It Does

```
LB performs health checks to /  (default path)

If EC2 app responds  → status: healthy  → LB sends traffic ✅
If EC2 app fails     → status: unhealthy → LB stops traffic ❌

Health check settings in TG:
  Healthy threshold:   2   ← 2 consecutive successes = healthy
  Unhealthy threshold: 2   ← 2 consecutive failures = unhealthy
  Timeout:             6s  ← no response in 6s = failed check
  Interval:            30s ← check every 30 seconds
  Health check path:   /   ← LB pings this path
```

---

## 📌 Elastic IP — Static Public IP

> **Elastic IP = a static, fixed public IP address you can assign to a server**

### Why you need it
```
Normal EC2 public IP:
  Stop EC2 → IP changes
  Start EC2 → different public IP
  Problem: DNS entries, configs break

Elastic IP:
  Fixed IP that stays the same forever
  Even if you stop/start EC2 → same IP
  Great for servers that need a permanent address
```

### Elastic IP Lifecycle
```
Step 1: Allocate     → AWS gives you an Elastic IP from its pool
Step 2: Associate    → Attach it to your EC2 instance
Step 3: Disassociate → Detach from EC2 (IP still yours)
Step 4: Release      → Return IP back to AWS pool
Step 5: Delete       → Remove
```

```
In AWS Console:
  EC2 → Network & Security → Elastic IPs
  → Allocate Elastic IP address
  → Actions → Associate → select EC2 instance
```

> ⚠️ **AWS charges for Elastic IPs that are allocated but NOT associated to any resource**  
> Always release Elastic IPs when not in use

---

## 🏗️ Final Architecture with ALB

```
Internet
    ↓
ALB DNS (myalb-936782470.us-east-1.elb.amazonaws.com)
    ↓
ALB (public subnet 1a + 1b)
  [Listener: HTTP:80 → Forward to TG-1]
    ↓
Target Group (TG-1)
  [Health check: / every 30s]
    ↓         ↓
Private 1a   Private 1b
  EC2 (app)   EC2 (app)
  ← Nginx serving HTML →

Bastion Host (public subnet)
  ← Developer SSH access to private EC2s
```

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| Private server config | SSH via Bastion → install Nginx → edit index.html → `curl localhost` to verify |
| Target Group | Create TG → register private EC2s → set health check path to `/` |
| ALB creation | Needs min 2 public subnets in different AZs, SG, Listener on 80 |
| ALB DNS | End users use ALB DNS name — never EC2 IP directly |
| Listener | Port 80 → forward to TG — checks rules → routes traffic |
| Health check | LB pings `/` every 30s — unhealthy server removed from rotation |
| Elastic IP | Static fixed public IP — doesn't change on EC2 stop/start |
| Elastic IP lifecycle | Allocate → Associate → Disassociate → Release |
| Elastic IP cost | Charged when allocated but NOT associated — always release unused |

---

> 📎 **Next:** Day 13 — Path-Based Routing (ALB Rules)
