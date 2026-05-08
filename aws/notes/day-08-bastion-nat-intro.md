# Day 08 — Bastion Host & NAT Gateway

> 📅 **Date:** 21st April  
> 🏷️ **Topic:** Public ↔ Private subnet communication, Bastion Host pattern, NAT Gateway intro, app deployment on private server

---

## ❓ The Core Problem

You've deployed your application on a **private server** (as it should be).  
But now — how do you access it to install packages, deploy code, debug?

```
Your Laptop
    ↓
    ✗ Cannot directly reach private EC2 (no public IP, no internet route)
```

And the private server itself:
```
Private EC2
    ↓
    ✗ Cannot reach internet to install packages (python, nginx, dependencies)
```

Two different problems → Two different solutions:
- **Access problem** → Bastion Host
- **Internet problem** → NAT Gateway

---

## 🏰 Bastion Host (Jump Host)

> **Bastion Host = A public EC2 that acts as a gateway to reach private servers**  
> Also called: **Jump Server** or **Jump Host**

### How it works
```
Your Laptop
    ↓  SSH (port 22, public IP of bastion)
Bastion Host  ← public subnet, has public IP
    ↓  SSH (port 22, private IP of target)
Private EC2   ← private subnet, app deployed here ✅
```

### Why "Bastion"?
- The public server is NOT for deploying apps
- It is ONLY used as a **secure access point** to jump into private servers
- Think of it as a fortified gateway — no apps, just SSH access

> ⚠️ **Never deploy applications on the Bastion Host**  
> Applications ALWAYS go on private servers

---

## 🔧 Requirements to Use Bastion

```
To SSH into Bastion (public EC2):
├── Internet connection
├── SG on Bastion allows port 22
├── Bastion's public IP
├── Private key (.pem)
└── Username: ec2-user

To SSH from Bastion into Private EC2:
├── Private IP of the private EC2
├── SG on Private EC2 allows port 22 from Bastion's private IP
├── Private key (uploaded to Bastion)
└── Username: ec2-user
```

---

## 💻 Step-by-Step — Connecting via Bastion

### Step 1: SSH into Bastion (from your laptop)
```bash
ssh -i "key.pem" ec2-user@<bastion-public-ip>
# e.g.: ssh -i "devkey.pem" ec2-user@44.197.243.176
```

### Step 2: Create the private key file ON the Bastion
```bash
# Once inside bastion:
vi devpractivekey.pem
# Press 'i' to insert mode
# Paste your private key content
# Press Esc → :wq! → Enter (save and quit)

# Verify it was created
ls
# → devpractivekey.pem
```

### Step 3: Set correct permissions on the key
```bash
chmod 400 devpractivekey.pem
# If you skip this step → you get "bad permissions" error
```

### Step 4: SSH from Bastion into Private EC2
```bash
ssh -i devpractivekey.pem ec2-user@<private-ip>
# e.g.: ssh -i devpractivekey.pem ec2-user@10.0.1.211

# Success! You are now inside the private server:
[ec2-user@ip-10-0-1-211 ~]$  ✅
```

---

## ❌ Common Error — Wrong Path

```bash
# WRONG — trying to SSH to private EC2 directly from your laptop:
ssh -i "C:\Users\veera\Downloads\devpractivekey.pem" ec2-user@10.0.1.211
# Error: not accessible: No such file or directory

# WHY: Your laptop cannot reach 10.0.1.211 — it's a private IP!
# Private IPs are only reachable WITHIN the VPC
# You must go through Bastion first
```

---

## 🏗️ Bastion Architecture

```
Internet
    ↓
Your Laptop
    ↓ SSH → public IP (44.197.243.176)
┌─────────────────────────────────────────────────┐
│  VPC: 10.0.0.0/16                               │
│              IG                                  │
│              ↕  edit routes                     │
│             RT                                  │
│    subnet association                           │
│         /                  \                    │
│  Public Subnet          Private Subnet          │
│  ┌─────────────┐        ┌──────────────┐        │
│  │   Bastion   │        │  Private EC2 │        │
│  │   EC2       │        │  (App: Flip) │        │
│  │ pub: 44.x   │──SSH──→│ pvt: 10.0.1.211│      │
│  │ pvt: 10.0.x │   ↑    └──────────────┘        │
│  │  [SG]       │  upload pvt key                │
│  └─────────────┘                                │
│                                                 │
│  private key → source end (bastion/your laptop) │
│  public key  → destination end (private EC2)   │
└─────────────────────────────────────────────────┘
```

---

## 📦 Why Private Servers Need Internet — App Deployment

Even though private servers are isolated from inbound internet traffic, they still need **outbound internet** to install software:

```
Python App Deployment on Private EC2:
────────────────────────────────────
Step 1: install python         → needs internet to download ❌
Step 2: copy code to server    → done via Bastion ✅
Step 3: install dependencies   → needs internet to download ❌
Step 4: run application        → done locally ✅

Problem: Steps 1 and 3 require internet access
But private EC2 has NO internet route!
→ Solution: NAT Gateway
```

---

## 🔄 NAT Gateway — Introduction

> **NAT Gateway = Service that gives OUTBOUND internet access to private servers**  
> NAT = Network Address Translator

### The Problem It Solves
```
Private EC2 (10.0.1.4)
    wants to: sudo yum install python3-pip
    needs: outbound internet access
    problem: no public IP, no internet route

NAT Gateway:
    sits in PUBLIC subnet
    has its own public IP (e.g., 174.2.3.5)
    private EC2 uses NAT as proxy to reach internet ✅
```

### Where to Always Place NAT
```
✅ Always add NAT Gateway to PUBLIC subnet (or at VPC level)
❌ Never place NAT in private subnet
```

### NAT in Action
```bash
# Private EC2 trying to install package:
[ec2-user@ip-10-0-1-211 ~]$ sudo yum install python3-pip
# Amazon Linux 2023 repository...
# Downloading packages... ← this works because of NAT ✅
```

---

## 🔁 How NAT Works (Preview)

```
Private server (10.0.1.4)
    ↓  outbound request → ping google.com
NAT Gateway (public IP: 174.2.3.5)
    ↓  translates: 10.0.1.4 → 174.2.3.5
Internet (Google)
    ↓  response to 174.2.3.5
NAT Gateway
    ↓  translates back: 174.2.3.5 → 10.0.1.4
Private server receives response ✅

Attacker (192.3.4.6) tries to initiate inbound connection:
    → 192.3.4.6 → NAT → 10.0.1.4 ❌ NOT POSSIBLE
    NAT only allows OUTBOUND initiated connections
```

> NAT is **stateful** — it remembers which private IP made which request and routes responses back correctly.  
> Full NAT deep-dive → Day 09

---

## 🏗️ Full Architecture with Bastion + NAT

```
Internet
    ↓
VPC (10.0.0.0/16)
    ├── IGW (attached to VPC)
    │
    ├── Public Subnet
    │   ├── Bastion EC2   ← developer SSH access point
    │   └── NAT Gateway   ← outbound internet for private servers
    │
    └── Private Subnet
        └── EC2 (App)     ← application lives here
            ↑ SSH via Bastion
            ↑ Internet via NAT (outbound only)
```

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| Bastion Host | Public EC2 used as SSH gateway to private servers |
| Jump Host | Another name for Bastion Host |
| Bastion rule | NEVER deploy apps on Bastion — only for SSH access |
| SSH to private EC2 | Must go through Bastion — private IPs unreachable directly |
| Key on Bastion | Copy your `.pem` to Bastion using `vi` → `chmod 400` |
| Bad permissions error | `chmod 400 key.pem` fixes this |
| NAT Gateway | Gives private servers OUTBOUND internet access only |
| NAT placement | Always in PUBLIC subnet |
| Private app + internet | Private EC2 needs NAT to install packages |
| End-user access | End users access private server via Load Balancer (not Bastion) |

---

> 📎 **Next:** Day 09 — NAT Gateway Deep Dive (Stateful, Port Mapping, Architecture)
