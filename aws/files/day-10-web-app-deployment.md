# Day 10 — Web Application Deployment

> 📅 **Date:** 23rd April  
> 🏷️ **Topic:** Web servers, Nginx installation, deploying HTML app on EC2, SG for web traffic

---

## 🌐 What is a Web Server?

> **Web server = a process running inside EC2 that delivers web content to end users**

```
Developer writes HTML code
        ↓
Deploys it inside web server directory on EC2
        ↓
Web server process listens on port 80
        ↓
End user hits Public IP:80 in browser
        ↓
Web server responds with HTML page ✅
```

### Web Server Examples
```
┌─────────────────────────────────────┐
│  Popular Web Servers                │
│  ─────────────────────────────────  │
│  • NGINX        ← most common       │
│  • Apache HTTPD ← also common       │
└─────────────────────────────────────┘
```

> By default web servers run on **port 80** (HTTP)  
> After whitelisting port 80 in SG → end users can access via public IP

---

## 📋 3-Step Deployment Process

```
Step 1 → Create EC2 instance
Step 2 → Install web server to deliver content
Step 3 → Deploy app inside web server
```

---

## ⚠️ Important Note from Class

> **For now we deploy on public server for learning purposes.**  
> **Recommended:** Deploy on **private server** and expose via **Load Balancer.**  
> We will do this once we learn Load Balancer.

---

## 🔗 Keyless Connectivity to Public EC2

> Before starting — you can connect to public EC2 directly from browser without a `.pem` key:

```
AWS Console → EC2 → Instances → Select your EC2
  → Click "Connect"
  → Connection type: Connect using a Public IP
  → Username: ec2-user
  → Click "Connect"
  → Browser terminal opens directly ✅
```

This only works for **public EC2 instances** — no key needed.

---

## 🛠️ Full Deployment Commands — Step by Step

```bash
# Step 1: Switch to root user
sudo su -
# Prompt changes from $ to # (root)

# Step 2: (Optional) Install git if you want to clone code
yum install git -y

# Step 3: Install Nginx web server
yum install nginx -y

# Step 4: Start Nginx
systemctl start nginx

# Step 5: Check Nginx is running
systemctl status nginx
# Look for: Active: active (running)

# Step 6: Navigate to web root directory
cd /usr/share/nginx/html/

# Step 7: List existing files
ls
# → 404.html  50x.html  index.html  nginx-logo.png  ...

# Step 8: Edit the index.html with your content
vi index.html
# Press 'i' to insert
# Add your HTML content
# Press Esc → type :wq! → Enter (save and quit)

# Step 9: Restart Nginx to apply changes
systemctl restart nginx
```

---

## 📄 Sample HTML App (from class)

```html
<html>
<body>
  <h1>Welcome to devops and cloud</h1>
  <p>by naresh</p>
</body>
</html>
```

---

## 🔐 SG Rules Required

> You must whitelist port 80 in Security Group for end users to access the app:

| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| SSH | TCP | 22 | 0.0.0.0/0 | Your SSH access |
| HTTP | TCP | 80 | 0.0.0.0/0 | End user web access |

```
In SG → Edit inbound rules → Add rule:
  Type: HTTP
  Protocol: TCP
  Port: 80
  Source: 0.0.0.0/0  (anyone can access)
→ Save rules
```

---

## ✅ Access the App

```
Open browser → http://<public-ip>:80
or simply  → http://<public-ip>

e.g. → http://54.198.145.249:80

You should see your HTML page ✅
```

---

## 📋 Overall Steps Summary (from class)

```bash
sudo su -                          # switch to root
yum install nginx -y               # install nginx web server
systemctl start nginx              # start nginx
systemctl status nginx             # check running status
cd /usr/share/nginx/html/          # go to web root
vi index.html                      # edit default HTML file
# paste your HTML → :wq! to save
# access with: http://<public-ip>

# Note: by default web server runs on port 80
# After whitelisting port 80 in SG → access via public IP
```

---

## 📌 Key Takeaways

| Concept | One-liner |
|---------|-----------|
| Web server | Process inside EC2 that delivers web content to users |
| Nginx / Apache | Two most common web servers |
| Default port | Web servers run on port 80 by default |
| Web root | `/usr/share/nginx/html/` — all HTML files go here |
| `index.html` | Default file served by Nginx |
| SG rule | Must allow port 80 inbound for users to access |
| Deploy on public | Only for learning — production always uses private + LB |
| Keyless connect | AWS console browser terminal — works for public EC2 only |
| Load balancer | Needed minimum 2 public subnets with server — coming next |

---

> 📎 **Next:** Day 11 — Application Load Balancer (ALB) & Target Groups
