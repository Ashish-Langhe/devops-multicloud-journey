# Day 07 — Custom Networking, Public & Private Subnets & SSH Connection

> 📅 **Date:** 20th April  
> 🏷️ **Topic:** Custom VPC setup, Public vs Private subnet with EC2, IP types, SSH hands-on, troubleshooting

---

## 🏗️ Custom Networking — The Right Approach

> AWS provides a **default network** out of the box, but in real projects you always create **custom networking**.

### Step-by-step approach from class
```
1. First → DRAW your architecture on paper/whiteboard
2. Then  → Create VPC inside AWS region
3. Then  → Create subnets using CIDR sizing
4. Then  → Add IGW, Route Tables, SGs
5. Then  → Launch EC2 instances
```

> 💡 **Never start clicking in AWS without a clear architecture diagram first.**

---

## 🌐 Two Types of Subnets — Side by Side

### Scenario: VPC `10.0.0.0/16` with 2 subnets

```
VPC: 10.0.0.0/16  (Custom Network)
┌──────────────────────────────────────────────────────┐
│                                                      │
│   Public Subnet          Private Subnet              │
│   10.0.0.0/24            10.0.1.0/24                 │
│  ┌────────────┐          ┌────────────┐              │
│  │    EC2     │          │    EC2     │              │
│  │  publicIP  │          │ privateIP  │              │
│  │  privateIP │          │  only      │              │
│  │   [SG]     │          │   [SG]     │              │
│  └────────────┘          └────────────┘              │
│         ↑                      ✗                     │
│    IGW → RT              No internet route           │
│    Internet accessible    Not reachable externally   │
└──────────────────────────────────────────────────────┘
```

### Completely isolated (both private) — also possible
```
VPC: 10.0.0.0/16
┌──────────────────────────────────────────┐
│                                          │
│   Subnet 1 (private)  Subnet 2 (private) │
│  ┌────────────────┐  ┌────────────────┐  │
│  │  EC2           │  │  EC2           │  │
│  │  No internet   │  │  No internet   │  │
│  └────────────────┘  └────────────────┘  │
│                                          │
│   Both are completely isolated ❌        │
└──────────────────────────────────────────┘
```

---

## 🔐 Public IP vs Private IP

```
┌────────────────────────────────────────────────────────┐
│  Public IP                                             │
│  ──────────                                            │
│  • Managed by AWS public IP pool                       │
│  • NOT derived from your VPC CIDR range                │
│  • Changes every time you stop/start EC2               │
│  • Assigned only when you ENABLE auto-assign public IP │
│  • Required for EC2 to be reachable from internet      │
│                                                        │
│  Private IP                                            │
│  ──────────                                            │
│  • Comes from your subnet CIDR range                   │
│  • Always assigned to every EC2                        │
│  • Stays the same (persistent)                         │
│  • Used for internal communication within VPC          │
└────────────────────────────────────────────────────────┘
```

### What does `0.0.0.0/0` mean?
```
0.0.0.0/0  =  ALL IP addresses  =  anyone on internet

In Route Table:
  Destination: 0.0.0.0/0  →  Target: IGW
  = Any traffic going to internet → use IGW ✅

In Security Group:
  Source: 0.0.0.0/0  →  Port: 22
  = Allow SSH from ANY IP in the world ⚠️ (use carefully)
```

> 💡 **Best practice:** Never expose SSH (`0.0.0.0/0`) in production. Restrict to your specific IP.

---

## 🔑 5 Things Needed to SSH into EC2

```
┌──────────────────────────────────────────────────┐
│  SSH Checklist                                   │
├──────────────────────────────────────────────────┤
│  1. ✅ Internet connection on your laptop        │
│  2. ✅ SG must allow port 22 (inbound)           │
│  3. ✅ Public IP of the EC2 instance             │
│  4. ✅ Private key file (.pem)                   │
│  5. ✅ Username: ec2-user (Amazon Linux)         │
└──────────────────────────────────────────────────┘
```

> ⚠️ **Critical:** SG checks ONLY if port 22 is allowed or not — it does NOT check if the key is valid.  
> Key validation happens INSIDE the server, AFTER the SG check passes.

---

## 🛠️ Tools to Connect to EC2

| Tool | Platform | Key Format | Notes |
|------|----------|-----------|-------|
| **Terminal** | Mac / Linux | `.pem` | Built-in, recommended |
| **MobaXterm** | Windows only | `.pem` | GUI + terminal |
| **PuTTY** | Windows | `.ppk` | Needs key conversion |
| **VS Code** | Any | `.pem` | Remote SSH extension |
| **Cloud Connect** | Browser | No key needed | AWS Console, public EC2 only |

> 💡 How to get `.pem` key? → EC2 console → Key Pairs → Before creating EC2, create key → Download `.pem`

---

## 💻 SSH Commands — Hands-On

### From Terminal (Mac / Linux)

```bash
# Step 1: Set correct permissions on key file (REQUIRED)
chmod 400 "keyfortest.pem"

# Step 2: SSH into the EC2
ssh -i "keyfortest.pem" ec2-user@<public-ip>

# Real example from class:
ssh -i "keyfortest.pem" ec2-user@13.220.238.151

# Or with full path on Windows:
ssh -i "C:\Users\veera\Downloads\keyfortest.pem" ec2-user@13.220.238.151
```

### What success looks like
```
    #_
   _|  ####_     Amazon Linux 2023
  ~\_| ####\
  ~~\__####|
   ~~\#####/    https://aws.amazon.com/linux/amazon-linux-2023
     \/~~~'

[ec2-user@ip-10-0-0-130 ~]$   ← you are inside the server ✅
```

### Where to find your Public IP in AWS Console
```
EC2 → Instances → click your instance
  → Look for: "Auto-assigned IP address" or "Public IPv4 address"
  → Example: 13.220.238.151 [Public IP]
  → Private IP: 10.0.0.130
```

---

## 🔑 Key Pair Management

```bash
# Once inside EC2, navigate to SSH keys folder
cd .ssh
ls
# → authorized_keys  (contains the public key)

# View public key stored on server
cat authorized_keys

# Remove authorized keys (for testing - removes access)
rm -rf authorized_keys
```

> ⚠️ **Key pairs are RSA encrypted** — creates public + private key pair  
> `.pem` format → OpenSSH (terminal)  
> `.ppk` format → PuTTY (Windows)

---

## ❌ Troubleshooting SSH — 3 Common Errors

### Error 1 — Port 22 blocked in SG
```
Symptom: Network error: Connection timed out
Cause:   SG inbound rule for port 22 is disabled/missing
Fix:     Add inbound rule: SSH | TCP | 22 | 0.0.0.0/0 (or your IP)
```

### Error 2 — Wrong key used
```
Symptom: Server refused our key
         No supported authentication methods available
Cause:   .pem file used doesn't match the key pair on server
Fix:     Use the correct .pem that was selected during EC2 launch
```

### Error 3 — Removed IGW from Route Table
```
Symptom: Connection timed out (same as Error 1 but different cause)
Cause:   0.0.0.0/0 → IGW route removed from Route Table
Fix:     Edit routes → Add 0.0.0.0/0 → IGW back
```

---

## 🧪 Take-Home Tasks (from class)

```
Task 3: Remove IGW from Route Table (edit routes)
        → Try to SSH → What happens? (Connection timeout)
        → Re-add IGW → SSH works again

Task 4: Remove subnet from Route Table (subnet association)
        → Try to SSH → What happens? (Connection timeout)
        → Re-associate subnet → SSH works again
```

> These tasks teach you that BOTH the route AND the subnet association must be correct for connectivity.

---

## 🏗️ Full Custom Network Architecture

```
VPC: 10.0.0.0/16
┌─────────────────────────────────────────────────────────┐
│                         ①VPC                            │
│                    ③ IGW ←→ Internet                   │
│                         ↕  ⑤ edit routes               │
│                      ④ RT                               │
│              ⑥ subnet associations                      │
│                    /         \                          │
│   Public 10.0.0.0/24      Private 10.0.1.0/24          │
│  ┌──────────────────┐      ┌──────────────────┐        │
│  │  ② EC2           │      │  ② EC2           │        │
│  │  publicIP: 13.x  │      │  privateIP only  │        │
│  │  privateIP: 10.x │      │  (Flipkart app)  │        │
│  │  [SG] ⑦         │      │  [SG] ⑦         │        │
│  └──────────────────┘      └──────────────────┘        │
│  ↑                         ✗                           │
│  Reachable from internet    Not reachable externally    │
└─────────────────────────────────────────────────────────┘
```

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| Custom networking | Always draw architecture first, then build |
| Public subnet | Subnet + RT with `0.0.0.0/0 → IGW` |
| Private subnet | Subnet + RT without internet route |
| Public IP | From AWS pool, changes on restart, enables internet access |
| Private IP | From VPC CIDR, permanent, for internal communication |
| `0.0.0.0/0` | Any IP / anywhere — internet open access |
| SSH checklist | Internet + SG port 22 + public IP + `.pem` + `ec2-user` |
| SG & key | SG checks port FIRST — key is checked AFTER by server |
| Connection timeout | SG blocked OR IGW route missing OR subnet not associated |
| Permission denied | SG passed, but wrong key used |

---

> 📎 **Next:** Day 08 — Public ↔ Private Subnet Communication, Bastion Host & NAT Gateway
