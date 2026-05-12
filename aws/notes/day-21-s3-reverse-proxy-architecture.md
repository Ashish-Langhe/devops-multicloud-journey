# 🏗️ Three EC2 Reverse Proxy Architecture
> **Custom Load Balancer + Frontend + Backend** — A production-grade, private-subnet-first design on AWS.

[![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=flat-square&logo=amazon-aws)](https://aws.amazon.com/ec2/)
[![Nginx](https://img.shields.io/badge/Nginx-Reverse_Proxy-009639?style=flat-square&logo=nginx)](https://nginx.org)
[![VPC](https://img.shields.io/badge/AWS-VPC-8C4FFF?style=flat-square&logo=amazon-aws)](https://aws.amazon.com/vpc/)
[![Security](https://img.shields.io/badge/Security-Groups-E83E3E?style=flat-square&logo=amazon-aws)](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [Architecture Overview](#-architecture-overview) |
| 2 | [EC2 #1 — Custom Load Balancer](#-ec2-1--custom-load-balancer) |
| 3 | [EC2 #2 — Frontend](#-ec2-2--frontend) |
| 4 | [EC2 #3 — Backend](#-ec2-3--backend) |
| 5 | [Request Flow — Step by Step](#-request-flow--step-by-step) |
| 6 | [Security Group Rules](#-security-group-rules) |
| 7 | [Nginx Configs](#-nginx-configs) |
| 8 | [Setup Commands](#-setup-commands) |
| 9 | [Key Takeaways](#-key-takeaways) |

---

## 🗺️ Architecture Overview

```
                        ┌─────────────────────┐
                        │   Internet / Client  │
                        │  (Browser / API)     │
                        └──────────┬──────────┘
                                   │
                              HTTP :80 / :443
                                   │
                                   ▼
                   ┌───────────────────────────────┐
                   │      EC2 #1 — Custom LB        │  ◀── Only machine
                   │        (Public IP)             │      with Public IP
                   │   Nginx  |  port 80            │
                   │   Reverse Proxy / Distributor  │
                   └───────────────┬───────────────┘
                                   │
                    proxy_pass → FE private IP :3000
                                   │
              ┌────────────────────▼────────────────────┐
              │                                          │
              ▼                                          ▼
 ┌────────────────────────┐             ┌────────────────────────┐
 │   EC2 #2 — Frontend    │  ─────────▶ │   EC2 #3 — Backend     │
 │     (Private IP only)  │  API :8080  │     (Private IP only)  │
 │  React / Vue / Node    │             │  Node / Python / Java  │
 │  port 3000             │ ◀─────────  │  port 8080             │
 │                        │  JSON resp  │  Business Logic + DB   │
 └────────────────────────┘             └────────────────────────┘

 ◀ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ Response flows back same path ─ ─ ─ ─ ─ ─ ─ ─ ─▶
```

### The Golden Rule

> ⚡ **Only EC2 #1 (LB) has a Public IP.**
> EC2 #2 (Frontend) and EC2 #3 (Backend) live on **private IPs only** — completely invisible to the internet.

---

## 🟣 EC2 #1 — Custom Load Balancer

### What it is
This EC2 acts as the **single entry point** for all internet traffic. It runs **Nginx** purely as a reverse proxy — no app code, no frontend files live here. Its only job is to receive requests and forward them internally.

### Responsibilities
- Accept all incoming HTTP/HTTPS traffic from the internet
- Terminate SSL (optional — can offload TLS here)
- Forward (proxy) requests to the Frontend EC2's private IP
- Can be extended later to **round-robin** across multiple frontend instances
- Hides all internal topology from the client

### Specs (Recommended)
| Property | Value |
|---|---|
| IP | Public IP (Elastic IP recommended) |
| OS | Amazon Linux 2023 / Ubuntu 22.04 |
| Software | Nginx |
| Port exposed | 80 (and 443 for HTTPS) |
| Security Group | Inbound: 0.0.0.0/0 on 80/443 |

---

## 🔵 EC2 #2 — Frontend

### What it is
The **presentation layer**. This EC2 runs your frontend application — React, Vue, Next.js, plain HTML — whatever serves the UI. It never touches the internet directly.

### Responsibilities
- Receive proxied requests from the LB EC2
- Serve HTML, CSS, JavaScript assets to the client (via LB)
- Make **internal API calls** to the Backend EC2's private IP when data is needed
- Server-Side Rendering (SSR) if using Next.js / Nuxt

### Specs (Recommended)
| Property | Value |
|---|---|
| IP | **Private IP only** (no public IP) |
| OS | Amazon Linux 2023 / Ubuntu 22.04 |
| Software | Node.js / React / Vue / Next.js |
| Port listening | 3000 (internal only) |
| Security Group | Inbound: port 3000 from **LB Security Group only** |

---

## 🟢 EC2 #3 — Backend

### What it is
The **brain** of the application. This EC2 runs your API server — all business logic, database queries, authentication, and data processing happen here. It is the most protected machine in the stack.

### Responsibilities
- Receive API requests from the Frontend EC2 (internal VPC only)
- Run business logic (auth, validation, computation)
- Query databases (RDS, DynamoDB, ElastiCache)
- Return JSON responses to the Frontend EC2
- Never directly exposed to the client or the internet

### Specs (Recommended)
| Property | Value |
|---|---|
| IP | **Private IP only** (no public IP) |
| OS | Amazon Linux 2023 / Ubuntu 22.04 |
| Software | Node.js / Python Flask/FastAPI / Java Spring |
| Port listening | 8080 (internal only) |
| Security Group | Inbound: port 8080 from **Frontend Security Group only** |

---

## 🔄 Request Flow — Step by Step

### Full Request Lifecycle

```
STEP 1:  Client ──────────────────────────▶ EC2 #1 LB (Public IP :80)
         Browser hits http://<LB-Public-IP>
         DNS resolves to LB's Elastic IP

STEP 2:  EC2 #1 LB ──────────────────────▶ EC2 #2 Frontend (:3000)
         Nginx: proxy_pass http://<FE-private-IP>:3000
         Request forwarded over private VPC network

STEP 3:  EC2 #2 Frontend serves the page
         Returns HTML/CSS/JS for static content
         ─ ─ For dynamic data, continues to STEP 4 ─ ─

STEP 4:  EC2 #2 Frontend ────────────────▶ EC2 #3 Backend (:8080)
         Internal API call: http://<BE-private-IP>:8080/api/...
         Private VPC network — never leaves AWS

STEP 5:  EC2 #3 Backend ─────────────────▶ EC2 #2 Frontend
         Returns JSON response
         Frontend renders it, sends final HTML back

STEP 6:  EC2 #2 Frontend ─────────────────▶ EC2 #1 LB ─────▶ Client
         LB forwards final response to the client
         Client only ever sees the LB's public IP
```

---

### Detailed Step Breakdown

#### Step 1 — Client Sends Request

```
User opens: http://<LB-Public-IP>  or  https://yourdomain.com
                    │
                    ▼
         EC2 #1 LB receives on port 80
         (Only this EC2 is reachable from internet)
```

- The client's browser resolves the domain to the LB's **Elastic IP**
- The Security Group on the LB allows `0.0.0.0/0` on port 80/443
- **EC2 #2 and EC2 #3 are invisible** — they have no public IP to reach

---

#### Step 2 — LB Forwards to Frontend

```
EC2 #1 LB (Nginx)
         │
         │  proxy_pass http://10.0.1.50:3000
         │  (Frontend private IP)
         ▼
EC2 #2 Frontend receives the request
```

- Nginx on the LB reads the incoming request
- Executes `proxy_pass` to the **Frontend's private IP** on port 3000
- Passes along original headers: `Host`, `X-Real-IP`, `X-Forwarded-For`
- This happens entirely **inside the VPC** — zero internet exposure

---

#### Step 3 — Frontend Serves the Page

```
EC2 #2 Frontend (Node.js / React / Next.js)
         │
         │  Serves: HTML + CSS + JS bundle
         │  For static pages → response goes back immediately
         │
         ▼
  (For dynamic data → continues to Step 4)
```

- For a **static page** request — Frontend renders and sends back HTML
- For a **dynamic page** (user data, products, etc.) — Frontend makes an internal API call to the Backend

---

#### Step 4 — Frontend Calls Backend API

```
EC2 #2 Frontend
         │
         │  axios.get('http://10.0.2.80:8080/api/users')
         │  (Backend private IP — internal VPC call)
         ▼
EC2 #3 Backend receives API request
         │
         │  Runs: auth check → DB query → business logic
         ▼
         Returns: JSON response
```

- This is a **server-to-server** call over the private VPC network
- The client/browser never knows this call happened
- Backend can safely connect to RDS, ElastiCache, S3 etc. from here

---

#### Step 5 & 6 — Response Travels Back

```
EC2 #3 Backend  ──JSON──▶  EC2 #2 Frontend  ──HTML──▶  EC2 #1 LB  ──▶  Client
   (10.0.2.80)              (10.0.1.50)              (Public IP)       (Browser)
```

- Backend sends JSON → Frontend renders into final HTML
- Frontend sends response to LB
- LB forwards to the original client
- **Client only ever saw one IP** — the LB's public IP

---

## 🔒 Security Group Rules

> This is the **defense-in-depth** model. Each layer only accepts traffic from the layer directly above it.

### EC2 #1 — Load Balancer SG

| Direction | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| Inbound | TCP | 80 | 0.0.0.0/0 | HTTP from internet |
| Inbound | TCP | 443 | 0.0.0.0/0 | HTTPS from internet |
| Inbound | TCP | 22 | Your-IP/32 | SSH for admin only |
| Outbound | All | All | 0.0.0.0/0 | Allow all outbound |

### EC2 #2 — Frontend SG

| Direction | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| Inbound | TCP | 3000 | **LB Security Group ID** | Traffic from LB only |
| Inbound | TCP | 22 | Your-IP/32 | SSH for admin only |
| Outbound | All | All | 0.0.0.0/0 | Allow all outbound |

> ⚠️ **Do NOT use `0.0.0.0/0` as source** — reference the LB's Security Group ID directly (e.g. `sg-0abc1234`). This means only traffic originating from the LB EC2 is accepted.

### EC2 #3 — Backend SG

| Direction | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| Inbound | TCP | 8080 | **Frontend Security Group ID** | Traffic from Frontend only |
| Inbound | TCP | 22 | Your-IP/32 | SSH for admin only |
| Outbound | All | All | 0.0.0.0/0 | Allow all outbound (for DB, S3 etc.) |

> ⚠️ **Same rule** — reference Frontend's Security Group ID (e.g. `sg-0def5678`), not an IP range.

---

## ⚙️ Nginx Configs

### EC2 #1 — Load Balancer Nginx Config

```nginx
# /etc/nginx/conf.d/loadbalancer.conf

upstream frontend {
    server 10.0.1.50:3000;          # Frontend EC2 private IP
    # Add more for round-robin:
    # server 10.0.1.51:3000;
    # server 10.0.1.52:3000;
}

server {
    listen 80;
    server_name _;                  # Accept all (or set your domain)

    location / {
        proxy_pass         http://frontend;
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;

        proxy_connect_timeout  10s;
        proxy_read_timeout     60s;
    }

    # Health check endpoint
    location /health {
        return 200 "LB OK\n";
        add_header Content-Type text/plain;
    }
}
```

---

### EC2 #2 — Frontend Nginx Config (optional, if serving static build)

```nginx
# /etc/nginx/conf.d/frontend.conf
# Use this if you build React/Vue and serve static files via Nginx
# Otherwise skip if using Node.js directly

server {
    listen 3000;
    server_name _;

    root /var/www/frontend/build;   # React build output
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;   # SPA fallback
    }

    # Proxy API calls from frontend to Backend EC2
    location /api/ {
        proxy_pass         http://10.0.2.80:8080;   # Backend EC2 private IP
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    }
}
```

---

### EC2 #3 — Backend (no Nginx needed)

```bash
# Backend app runs directly — Nginx not required here
# Example: Node.js Express listening on port 8080

node server.js          # or
pm2 start server.js     # production (with process manager)

# server.js listens on:
# app.listen(8080, '0.0.0.0')
```

---

## 🛠️ Setup Commands

### EC2 #1 — Load Balancer Setup

```bash
# 1. SSH into LB EC2
ssh -i ~/.ssh/mykey.pem ec2-user@<LB-PUBLIC-IP>

# 2. Install Nginx
sudo yum update -y
sudo yum install nginx -y

# 3. Create LB config
sudo vim /etc/nginx/conf.d/loadbalancer.conf
# (paste the LB config from above)

# 4. Remove default config
sudo rm -f /etc/nginx/conf.d/default.conf

# 5. Test config syntax
sudo nginx -t

# 6. Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# 7. Verify
sudo systemctl status nginx
curl http://localhost/health
```

---

### EC2 #2 — Frontend Setup

```bash
# 1. SSH into Frontend EC2 (via LB as jump host, or SSM)
ssh -i ~/.ssh/mykey.pem ec2-user@<FE-PRIVATE-IP>

# 2. Install Node.js
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install nodejs -y

# 3. Clone and install your app
git clone https://github.com/your-org/frontend-app.git /opt/frontend
cd /opt/frontend
npm install

# 4. Build (for React/Vue)
npm run build

# 5. Install PM2 (process manager — keeps app alive)
sudo npm install -g pm2

# 6. Start app
pm2 start npm --name "frontend" -- start  # Next.js
# or
pm2 start server.js --name "frontend"     # Plain Node

# 7. Auto-start on reboot
pm2 startup
pm2 save

# 8. Verify app is running on port 3000
curl http://localhost:3000
```

---

### EC2 #3 — Backend Setup

```bash
# 1. SSH into Backend EC2
ssh -i ~/.ssh/mykey.pem ec2-user@<BE-PRIVATE-IP>

# 2. Install Node.js (or Python, Java — your choice)
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install nodejs -y

# 3. Clone and install your API
git clone https://github.com/your-org/backend-api.git /opt/backend
cd /opt/backend
npm install

# 4. Set environment variables
sudo vim /opt/backend/.env
# DB_HOST=<RDS-endpoint>
# DB_PASS=<secret>
# PORT=8080

# 5. Start with PM2
pm2 start server.js --name "backend"
pm2 startup
pm2 save

# 6. Verify API is running on port 8080
curl http://localhost:8080/api/health
```

---

### Useful Verification Commands

```bash
# On LB EC2 — check Nginx is proxying correctly:
curl -v http://localhost                      # Should return frontend HTML

# Check Nginx error logs:
sudo tail -f /var/log/nginx/error.log

# On Frontend EC2 — check app is running:
pm2 status
curl http://localhost:3000

# On Backend EC2 — check API is running:
pm2 status
curl http://localhost:8080/api/health

# From LB EC2 — test reach to Frontend private IP:
curl http://10.0.1.50:3000

# From Frontend EC2 — test reach to Backend private IP:
curl http://10.0.2.80:8080/api/health

# Check Security Group is enforcing correctly:
# (This should FAIL — internet cannot reach Frontend directly)
curl http://10.0.1.50:3000    # from your laptop — should timeout
```

---

## 🎯 Key Takeaways

| Concept | What to Remember |
|---|---|
| **Public IP** | Only EC2 #1 (LB) has one — all internet traffic enters here |
| **Private IP** | EC2 #2 and EC2 #3 are private — unreachable from internet |
| **Nginx on LB** | Pure reverse proxy — `proxy_pass` to Frontend private IP |
| **Frontend** | Serves UI; makes internal API calls to Backend private IP |
| **Backend** | Business logic + DB — only Frontend's SG can reach it |
| **Security Groups** | Reference SG IDs (not IP ranges) for layer-to-layer rules |
| **Defense in depth** | Each layer only trusts the layer directly above it |
| **PM2** | Use it on Frontend and Backend EC2s to keep apps alive |
| **Scalability** | Add more Frontend IPs to Nginx `upstream` block for scaling |
| **No ALB needed** | This is a fully custom setup — Nginx replaces AWS ALB |

---

## 🔁 How It Differs from ALB Setup

| Feature | This Setup (Custom LB EC2) | AWS ALB |
|---|---|---|
| Cost | EC2 cost only | ALB hourly + LCU charges |
| Control | Full — edit Nginx freely | Limited to ALB settings |
| SSL | Manual (Certbot / manual cert) | ACM certificates (easy) |
| Health checks | Manual Nginx config | Built-in, automatic |
| Scaling | Manual — add IPs to upstream | Automatic with Target Groups |
| Use case | Learning, small apps, custom routing | Production at scale |

> 💡 **This custom setup is ideal for learning** how reverse proxying and load balancing actually work under the hood — before moving to managed AWS ALB.

---

## 📌 Tips & Gotchas

> 💡 Always use **Elastic IP** on the LB EC2 — public IPs change on stop/start without it.

> 💡 Use **AWS SSM Session Manager** to SSH into private EC2s without a bastion host.

> 💡 **`proxy_set_header X-Real-IP $remote_addr`** — without this, your backend logs will show the LB's IP instead of the real client IP.

> 💡 Use **PM2** on Frontend and Backend EC2s — ensures the app restarts automatically after a crash or reboot.

> ⚠️ **Never open port 3000 or 8080 to `0.0.0.0/0`** in Security Groups — these ports must only accept traffic from their upstream EC2's SG.

> ⚠️ If Nginx shows **502 Bad Gateway** — the backend app is not running. Check `pm2 status` on the target EC2.

> ⚠️ If Nginx shows **504 Gateway Timeout** — the backend app is running but taking too long. Tune `proxy_read_timeout`.

---

*Three EC2s. One public IP. Zero exposed internals. This is how real apps are built.* 🚀
