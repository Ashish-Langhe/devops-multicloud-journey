# Day 08-09 | Bastion Host & NAT Gateway

**Dates:** 21st April – 22nd April  
**Topics:** Public ↔ Private subnet communication, Bastion Host, NAT Gateway

---

## 🔒 Public vs Private Subnet Communication

A **private subnet server** has:
- ✅ Private IP only
- ❌ No public IP
- ❌ No direct internet access

To connect to a **private server from your laptop**, you cannot do it directly.  
You need to go through a **public server** inside the same VPC.

---

## 🏰 Bastion Host (Jump Host)

### What is it?
- A **Bastion Host** = a public EC2 instance used as a **gateway** to access private servers
- Also called **Jump Host** or **Jump Server**
- It acts as a **proxy** between the developer and the private server

### How it works
```
Your Laptop
    ↓ SSH (port 22, public IP)
Bastion Host (Public Subnet)
    ↓ SSH (port 22, private IP)
Private Server (Private Subnet)
    └── App is deployed here
```

### Requirements to connect to Private Server via Bastion

| Requirement | Detail |
|-------------|--------|
| Internet | Required on your laptop |
| Public IP | Bastion host's public IP |
| Private key | Upload `.pem` to bastion host |
| SG on Bastion | Allow port 22 from your IP |
| SG on Private | Allow port 22 from bastion's private IP |
| Username | `ec2-user` (Amazon Linux) |

### Steps to connect

```bash
# Step 1: SSH into bastion (public server)
ssh -i "key.pem" ec2-user@<bastion-public-ip>

# Step 2: Create the private key file on bastion
vi devpractivekey.pem   # press i, paste key content, :wq! to save

# Step 3: Set correct permissions
chmod 400 devpractivekey.pem

# Step 4: SSH into private server from bastion
ssh -i devpractivekey.pem ec2-user@<private-ip>
# Example: ssh -i devpractivekey.pem ec2-user@10.0.1.211
```

### Important Notes
- **Never deploy applications on the Bastion host** — it's only for access/jumping
- Always deploy applications on **private servers only**
- End users access the private server via **Load Balancer**, not Bastion
- `private key` → source end (your laptop / bastion)
- `public key` → destination end (on the server)

---

## 🔄 NAT Gateway (Network Address Translator)

### Why do we need it?

Private servers need internet to:
- Install packages (`yum install python3`, `apt install nginx`)
- Download dependencies
- Pull code from GitHub

But private servers have **no public IP** — so they can't directly reach the internet.  
→ **NAT Gateway** solves this!

### What is NAT Gateway?

- NAT = **Network Address Translator**
- An **AWS managed service** that provides **secure internet access to private servers**
- NAT **translates** the private IP to a public IP for outbound traffic

### How it works

```
Private Server (10.0.1.4)
    ↓ outbound request (ping google.com)
NAT Gateway (public subnet, has public IP: 174.2.3.5)
    ↓ translates 10.0.1.4 → 174.2.3.5
Internet (Google)
    ↓ response back to 174.2.3.5
NAT Gateway
    ↓ translates back to 10.0.1.4
Private Server ✅
```

### NAT is Stateful

- NAT **tracks the state** of each connection
- It remembers which private IP made which request
- Uses **port mapping** to differentiate multiple servers

```
10.0.1.5:5000 → 52.10.20.30:30001 (server 1 pinging Google)
10.0.1.6:5001 → 52.10.20.30:30002 (server 2 pinging Facebook)
```
→ No conflicts even if multiple servers use the same NAT!

### Key Points

| Point | Detail |
|-------|--------|
| NAT allows only | **Egress (outbound) traffic** |
| Inbound from internet | ❌ Not possible (hacker cannot initiate connection) |
| Private IP stays | Inside VPC — never exposed outside |
| NAT uses | Public IP on behalf of private server |
| Port range | 0–65535 |
| Always place NAT | In **public subnet** |

### NAT Gateway Architecture

```
VPC
 ├── Public Subnet
 │     ├── Bastion Host (EC2)
 │     └── NAT Gateway ← IG → Internet
 │
 └── Private Subnet
       └── App Server (EC2) → uses NAT for outbound
```

> ⚠️ **Attacker cannot initiate inbound connection through NAT** — private IP is never publicly exposed

---

## 📌 Key Takeaways

- Bastion Host = jump server to access private EC2 from internet
- Never deploy apps on Bastion — only use it for SSH access
- NAT Gateway = gives private servers outbound internet access only
- NAT is stateful — tracks connections using port mapping
- NAT must be placed in **public subnet**
- Private IP **never leaves** the VPC — NAT uses public IP on its behalf
