# Day 11-12 | Application Load Balancer, Target Groups & Elastic IP

**Dates:** 29th April – 30th April  
**Topics:** ALB, Target Groups, Load Balancer features, Elastic IP

---

## ⚖️ What is a Load Balancer?

A Load Balancer (LB) is a service that:
- **Distributes incoming traffic** across multiple servers
- Prevents any single server from being overwhelmed
- Provides **high availability** and **fault tolerance**
- Acts as a **proxy** — servers and apps are NOT directly exposed to users

---

## 🏗️ ALB (Application Load Balancer)

### How end-users access apps in private servers?
```
User → Internet → ALB (public subnet) → Target Group → Private EC2 (app)
```

### ALB Requirements
- Minimum **2 public subnets** in **different AZs** (not same AZ)
- At least one EC2 instance in a Target Group

### Load Balancer Features

| Feature | Description |
|---------|-------------|
| **Load distribution** | Balances traffic across servers |
| **Failover routing** | If a server fails, stops sending traffic to it |
| **Health checks** | Checks if app and server are healthy before routing |
| **Security** | Works as proxy — server not exposed to public |
| **High availability** | Routes to healthy servers if one fails |
| **Round Robin** | Default method — sends traffic equally to each server in turn |

---

## 🎯 Target Groups (TG)

### What is a Target Group?
- TG = **group of servers (EC2 instances) that LB routes traffic to**
- You define which servers are "targets" for the load balancer

### Why use Target Groups?
- Say you have 4 servers in us-east-1
- But LB should only send traffic to 2 specific servers
- → Create a TG with only those 2 servers

```
Load Balancer
  └── Target Group (TG-1)
        ├── EC2 (private subnet 1a) — app running
        └── EC2 (private subnet 1b) — app running
```

### Important Note
> LB can only send traffic to **AZs it is registered with**  
> Even if a server is in the TG, if its AZ is not selected in LB → traffic won't reach it  
> AZs of LB and Target Servers **must match**

---

## 🏥 Health Checks

LB constantly checks server health:

| Setting | Default | Meaning |
|---------|---------|---------|
| Healthy threshold | 2 | 2 consecutive successes = healthy |
| Unhealthy threshold | 2 | 2 consecutive failures = unhealthy |
| Timeout | 6 sec | No response in 6s = failed check |
| Interval | 30 sec | Check every 30 seconds |

- If app is **healthy** → LB sends traffic
- If app is **unhealthy** → LB stops sending traffic to that server

---

## 📐 Architecture — ALB with Private Servers

```
                Internet
                    ↓
            Internet Gateway
                    ↓
       Application Load Balancer
       (public subnet 1a + 1b)
              /         \
         TG EC2         TG EC2
    (private 1a)    (private 1b)
     Flipkart app    Flipkart app
```

---

## 🔧 How to Configure Private Server with Load Balancer

```bash
# On private EC2 (accessed via Bastion):
sudo su -
yum install nginx -y
systemctl start nginx
cd /usr/share/nginx/html/
vi index.html       # add your app content
curl http://localhost  # verify app is running
```

Then in AWS Console:
1. Create Target Group → Protocol: HTTP, Port: 80
2. Register private EC2 instances
3. Create ALB → select public subnets (different AZs)
4. Set Listener: HTTP:80 → Forward to TG
5. Access via ALB DNS name (not EC2 IP)

---

## 📌 Elastic IP

### What is Elastic IP?
- **Elastic IP = Static Public IP** assigned to a server
- Normal public IPs change every time you stop/start EC2
- Elastic IP **stays fixed** even after restart

### Lifecycle
```
Allocate → Associate (to EC2) → Disassociate → Release → Delete
```

> ⚠️ **AWS charges for Elastic IPs that are allocated but NOT associated**

---

## 📌 Key Takeaways

- ALB needs min 2 public subnets in different AZs
- Target Group = list of servers LB should route to
- LB AZs and TG server AZs must match
- LB uses Round Robin by default
- Health checks ensure traffic goes only to healthy servers
- Elastic IP = fixed static IP for EC2 (doesn't change on restart)
- End users should NEVER directly access private servers — always through LB
