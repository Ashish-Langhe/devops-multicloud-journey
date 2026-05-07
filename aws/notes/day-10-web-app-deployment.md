# Day 10 | Web Application Deployment (Nginx)

**Date:** 23rd April  
**Topics:** Web servers, Nginx installation, deploying HTML app on EC2

---

## 🌐 What is a Web Server?

- A **web server** = a process running inside EC2 that **delivers web content to end users**
- Examples of web servers: **Nginx**, **Apache HTTPD**
- By default web servers run on **port 80** (HTTP)

---

## 🚀 Deployment Steps

### Overall Flow
```
1. Create EC2 instance (public subnet for now)
2. Install web server (Nginx)
3. Deploy application inside web server directory
4. Allow port 80 in Security Group
5. Access via Public IP:80
```

---

## 🛠️ Commands — Deploy HTML App on EC2

```bash
# Step 1: Switch to root user
sudo su -

# Step 2: Install Git (optional, for cloning repos)
yum install git -y

# Step 3: Install Nginx web server
yum install nginx -y

# Step 4: Start Nginx
systemctl start nginx

# Step 5: Check status
systemctl status nginx

# Step 6: Navigate to web root directory
cd /usr/share/nginx/html/

# Step 7: Edit/create index.html
vi index.html
# press 'i' to insert, paste HTML, press Esc, type :wq! to save

# Step 8: Restart Nginx to apply changes
systemctl restart nginx
```

### Sample HTML File
```html
<html>
<body>
  <h1>Welcome to DevOps and Cloud</h1>
  <p>by Naresh</p>
</body>
</html>
```

### Access the app
```
http://<public-ip>:80
or simply:
http://<public-ip>
```

---

## 🔧 Security Group Rule Required

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | 22 | Your IP |
| HTTP | TCP | 80 | 0.0.0.0/0 (anyone) |

> Note: For testing, allow HTTP from `0.0.0.0/0`. In production, use Load Balancer.

---

## 📝 Keyless Connectivity (AWS Console)

- Go to EC2 → Instances → Select instance → **Connect**
- Choose **EC2 Instance Connect** (browser-based terminal)
- Only works for **public instances**

---

## ⚠️ Important Notes

- Temporarily deploying to public server for learning purposes
- **Recommended:** Deploy applications to **private servers** and expose via **Load Balancer**
- By default Nginx serves content from `/usr/share/nginx/html/`
- Default file served: `index.html`

---

## 📌 Key Takeaways

- Web server = process that serves HTML/app to end users (port 80)
- Nginx/Apache HTTPD are most common web servers
- Steps: Install → Start → Deploy HTML → Allow port 80 in SG → Access via IP
- In production: app should be in **private subnet**, accessed via **Load Balancer**
