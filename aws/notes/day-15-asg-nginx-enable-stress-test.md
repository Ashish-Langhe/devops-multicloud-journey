# Day 15 | ASG — Nginx Auto-Start & Stress Testing

**Date:** 4th May  
**Topics:** systemctl enable, ASG health checks, stress testing with stress-ng

---

## 🔁 Problem with ASG + Custom AMI

When ASG creates a new server from your custom AMI, **Nginx might not start automatically** after the EC2 boots up.

If Nginx isn't running → health check fails → LB marks server unhealthy → no traffic sent.

**Solution:** Use `systemctl enable` to make Nginx start automatically on every boot.

---

## ⚙️ systemctl enable — Auto-Start Nginx

```bash
# Step 1: Start Nginx manually
systemctl start nginx

# Step 2: Go to web root and add your content
cd /usr/share/nginx/html
vi index.html

# Step 3: ENABLE Nginx to auto-start on every boot
systemctl enable nginx
# Output: Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service
#         → /usr/lib/systemd/system/nginx.service
```

### What does `enable` do?
- Links Nginx service to the **bin/bash boot process**
- Every time EC2 starts → Nginx starts **automatically**
- Without `enable` → Nginx stops after reboot, health check fails, ASG replaces the server in a loop

> ✅ Always run `systemctl enable nginx` before creating your Custom AMI  
> This ensures every new server ASG creates will have Nginx running from boot

---

## 🏥 ASG Health Check Types

In ASG settings, you can enable additional health checks:

| Health Check Type | What it monitors |
|-------------------|-----------------|
| **EC2 health check** | Default — checks if EC2 instance is running |
| **ELB health check** ✅ Recommended | Checks if app responds via Load Balancer |
| **VPC Lattice health check** | Checks if instance handles requests |
| **EBS health check** | Checks if attached volume is healthy |

> Enable **ELB health checks** in ASG — this ensures traffic only goes to servers where the **app is actually responding**, not just the EC2 being powered on.

**Health check grace period:** `300 seconds`  
→ ASG waits 300s after launching before starting health checks (gives app time to start)

---

## 🧪 Stress Testing ASG — Does it Scale?

To test if ASG actually creates new servers under load:

```bash
# Install stress-ng tool
yum install stress-ng -y

# Run CPU stress test
stress-ng --cpu $(nproc) --cpu-load 100
# This maxes out ALL CPU cores at 100% load

# OR run for limited time
stress --cpu 2 --timeout 60
```

### What happens next?
1. CPU usage spikes to 100%
2. CloudWatch alarm triggers (if CPU > threshold you set)
3. ASG scaling policy activates
4. ASG creates new EC2 from Launch Template
5. New server registers in Target Group
6. LB starts sending traffic to new server too

> Watch in **EC2 → Auto Scaling Groups → Activity** tab to see scale-out events live

---

## 📐 Full ASG Flow Recap

```
Custom AMI (OS + App + nginx enabled)
          ↓
    Launch Template
    (AMI + instance type + key + SG)
          ↓
   Auto Scaling Group
   (subnets + TG + min/desired/max + scaling policy)
          ↓
   EC2 boots → nginx auto-starts → health check passes
          ↓
   LB sends traffic ✅
```

---

## 📌 Key Takeaways

- Always `systemctl enable nginx` before creating AMI — without this, ASG servers won't serve traffic after reboot
- Enable **ELB health checks** in ASG for accurate app-level health monitoring
- Use `stress-ng` to simulate load and verify ASG scales out correctly
- Health check grace period = 300s — gives new servers time to initialize
- After stress stops → ASG will **scale in** (remove extra servers) after cooldown
