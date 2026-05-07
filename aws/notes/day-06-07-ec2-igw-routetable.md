# Day 06-07 | EC2, Internet Gateway & Route Table

**Dates:** 17th April – 20th April  
**Topics:** EC2 creation, AMI, Key Pair, IGW, Route Table, SSH connection

---

## 💻 EC2 (Elastic Compute Cloud)

- EC2 = **Virtual Machine / Server** on AWS
- EC2 = vm = server

### 5 Things Needed to Create an EC2

| Requirement | AWS Service |
|-------------|-------------|
| 1. OS (Operating System) | AMI (Amazon Machine Image) |
| 2. Hardware — CPU, RAM | Instance Type (e.g., t2.micro) |
| 3. Hard Disk (Storage) | EBS (Elastic Block Storage) |
| 4. Networking | VPC, Subnet, Security Group |
| 5. Authentication | Key Pair (.pem / .ppk) |

### AMI (Amazon Machine Image)
- AMI = **OS + pre-installed Applications**
- AMI is **region-specific** — cannot use AMI from one region in another
- Examples: Amazon Linux 2023, Ubuntu, Windows Server

### Instance Types
| Instance | vCPUs | RAM | Use Case |
|----------|-------|-----|---------|
| t2.micro | 1 | 1 GB | Free tier, testing |
| t2.medium | 2 | 4 GB | Light workloads |
| t2.large | 2 | 8 GB | Medium workloads |

---

## 🔑 Key Pair

- Used for **authentication** to connect to the server
- Two types:
  - **Public key** → stored on server (destination end)
  - **Private key** → kept on your laptop (source end)
- File formats:
  - `.pem` → for OpenSSH / terminal
  - `.ppk` → for PuTTY (Windows)

> ⚠️ **Never share your private key** — store it safely  
> You can only download the key once during creation

---

## 🌐 Internet Gateway (IGW)

- IGW = **door between VPC and the internet**
- Attached at **VPC level**
- Required for any subnet to have **internet access**
- Without IGW, even public subnet servers cannot reach the internet

---

## 🗺️ Route Table (RT)

- Route Table = **Router for VPC**
- Defines **where network traffic should go**
- Every VPC has a main route table by default

### Route Table Rules

| Destination | Target | Meaning |
|-------------|--------|---------|
| 10.0.0.0/16 | local | Traffic within VPC stays local |
| 0.0.0.0/0 | IGW | All other traffic goes to internet |

> `0.0.0.0/0` = **any IP** = route all internet traffic through IGW

---

## 🔗 How to SSH into EC2 (Linux)

### Prerequisites (5 things needed)
```
1. Internet connection
2. SG must allow port 22
3. Public IP of server
4. Private key (.pem file)
5. Username: ec2-user (Amazon Linux)
```

### SSH Command (Terminal / Mac / Linux)
```bash
# Set correct permissions on key
chmod 400 "keyname.pem"

# Connect to server
ssh -i "keyname.pem" ec2-user@<public-ip>

# Example
ssh -i "loginsh.pem" ec2-user@13.220.238.151
```

### Tools to Connect to EC2
1. **Terminal** (Mac/Linux) — use `.pem`
2. **MobaXterm** (Windows only)
3. **PuTTY** (Windows) — use `.ppk`
4. **VS Code** (with SSH extension)
5. **Cloud Connect (Console)** — for public servers only (browser-based)

---

## 🧪 Use Cases / Troubleshooting

| Scenario | Error | Reason |
|----------|-------|--------|
| SG rule for port 22 is disabled | Connection timed out | SG blocked the request |
| Wrong key used | Permission denied / Server refused key | Key mismatch |
| No IGW attached | Cannot connect | No internet path |
| Subnet not associated with RT | Cannot connect | Traffic has no route |

---

## 📌 Key Takeaways

- EC2 = VM, needs AMI + Instance Type + EBS + VPC/SG + Key
- AMI is region-specific
- IGW = internet access for VPC
- Route Table defines traffic routing — `0.0.0.0/0 → IGW` = public subnet
- SG checks port, not key validity — if port 22 blocked → timeout
- Private key stays with you, public key goes to server
