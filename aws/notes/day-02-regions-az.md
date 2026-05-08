# Day 02 — AWS Global Infrastructure: Regions & Availability Zones

> 📅 **Date:** 13th April  
> 🏷️ **Topic:** Regions, Availability Zones, Data Centres, Multi-AZ Architecture

---

## 🌍 AWS Global Infrastructure — The Hierarchy

```
                        🌐 GLOBAL
                            │
                    ┌───────┴────────┐
                 Continent        Continent
              (North America)    (Asia Pacific)
                    │                  │
              ┌─────┴──────┐    ┌──────┴──────┐
           Region        Region  Region     Region
         (us-east-1)  (us-west-2) (ap-south-1) (ap-northeast-1)
              │
        ┌─────┴──────────────┐
        │                    │
   AZ (1a)              AZ (1b)              AZ (1c)
  (Datacentre)        (Datacentre)        (Datacentre)
        │
   Group of Servers
```

### Numbers to remember
```
┌──────────────────────────────────┐
│  33+  Regions globally           │
│  108+ Availability Zones         │
└──────────────────────────────────┘
```

---

## 📍 What is a Region?

- A **Region** = a **geographical location** that contains AWS infrastructure
- **Region ≠ Country** — a region is a cluster of data centres in a specific area
- Each region has a unique **code name**

### Region Examples
| Region Code | Location |
|-------------|----------|
| `us-east-1` | N. Virginia, USA |
| `us-east-2` | Ohio, USA |
| `us-west-1` | N. California, USA |
| `us-west-2` | Oregon, USA |
| `ap-south-1` | **Mumbai, India** |
| `ap-south-2` | **Hyderabad, India** |
| `ap-northeast-1` | Tokyo, Japan |
| `ap-southeast-1` | Singapore |
| `eu-west-1` | Ireland |
| `ca-central-1` | Canada |

### India specifically
```
INDIA
├── Mumbai     →  ap-south-1
│     ├── ap-south-1a
│     ├── ap-south-1b
│     └── ap-south-1c
│
└── Hyderabad  →  ap-south-2
      ├── ap-south-2a
      ├── ap-south-2b
      └── ap-south-2c
```

> 🎯 **Focus:** Don't think about countries — think about Regions. Your data centres live inside regions.

---

## 🏢 What is an Availability Zone (AZ)?

- AZ = **one or multiple Data Centres** physically located close together
- Each Region contains **multiple AZs** (usually 3)
- AZs within a region are **physically separate** but connected by high-speed private network
- Named by appending a letter: `ap-south-1a`, `ap-south-1b`, `ap-south-1c`

```
Region: ap-south-1 (Mumbai)
┌─────────────────────────────────────────┐
│                                         │
│   AZ: ap-south-1a    AZ: ap-south-1b   │
│   ┌─────────────┐    ┌─────────────┐   │
│   │ Data Centre │    │ Data Centre │   │
│   │ ┌─┐ ┌─┐ ┌─┐│    │ ┌─┐ ┌─┐ ┌─┐│   │
│   │ │S│ │S│ │S││    │ │S│ │S│ │S││   │
│   │ └─┘ └─┘ └─┘│    │ └─┘ └─┘ └─┘│   │
│   └─────────────┘    └─────────────┘   │
│              S = Server                 │
└─────────────────────────────────────────┘
```

---

## 🖥️ What is a Data Centre?

> **Data Centre = Group of Servers**

- Physical building housing hundreds/thousands of servers
- AWS owns and manages all physical infrastructure
- You never interact with physical hardware — only logical resources

---

## 🚀 Deploying Resources in AWS

> ⚠️ **Rule:** Before creating any server in AWS, you MUST select:
> 1. **Which Region**
> 2. **Which Availability Zone**
>
> AWS provider takes care of everything else (physical hardware, cooling, power, etc.)

```
You (DevOps Engineer)
    ↓
AWS Console
    ↓
Select Region → ap-south-1 (Mumbai)
    ↓
Select AZ     → ap-south-1a
    ↓
Launch EC2 (server)
    ↓
AWS provisions the physical server in that AZ ✅
```

---

## 🛡️ Multi-AZ — Why It Matters (Disaster Management)

> ✅ **Golden Rule:** Always deploy resources across **different AZs** for failure management.

### What happens without Multi-AZ?
```
Single AZ deployment:
  All servers in ap-south-1a
       ↓
  AZ-1a has a power outage
       ↓
  ALL servers down ❌
  Application completely unavailable ❌
```

### What happens with Multi-AZ?
```
Multi-AZ deployment:
  Servers in ap-south-1a  +  Servers in ap-south-1b
              ↓
  ap-south-1a has a power outage
              ↓
  ap-south-1a servers down
  ap-south-1b servers still running ✅
  Application still available ✅  ← High Availability!
```

### Reference Architecture (AWS Cloud Single-Tier Scalable)
```
                    www.cloudoers.com
                           │
                     Route53 (DNS)
                           │
                  Internet Gateway (IGW)
                           │
                  Elastic Load Balancer
                    /              \
          AZ: us-east-1a        AZ: us-east-1b
          ┌──────────────┐      ┌──────────────┐
          │ Public Subnet│      │ Public Subnet│
          │   EC2        │      │   EC2        │
          │   (app)      │      │   (app)      │
          └──────────────┘      └──────────────┘
              Auto Scaling Group (spans both AZs)
```

---

## 📋 AWS Continents Overview

| Continent | Key Regions |
|-----------|-------------|
| North America | US East (N. Virginia, Ohio), US West (Oregon, N. California), Canada |
| Asia Pacific | Mumbai, Tokyo, Singapore, Sydney, Jakarta, Osaka, Seoul |
| Europe | Frankfurt, Ireland, London, Paris, Stockholm, Milan, Spain, Zurich |
| Middle East | Bahrain, UAE, Israel (Tel Aviv) |
| Africa | Cape Town, South Africa |
| South America | São Paulo, Brazil |
| Specialized | AWS GovCloud (US), China regions |

> 🎯 **Class tip:** Don't worry about all continents — focus on regions. Your data centres are in regions.

---

## ✅ Key Takeaways

| Concept | One-liner |
|---------|-----------|
| Region | Geographical location containing AZs (33+ globally) |
| AZ | One or multiple data centres within a region (108+ globally) |
| Data Centre | Group of physical servers |
| Multi-AZ | Deploy across AZs for high availability & disaster recovery |
| Deployment rule | Always select Region + AZ before creating any resource |
| AWS manages | All physical infrastructure — you only manage logical resources |

---

> 📎 **Next:** Day 03 — DevOps Role, VPC Introduction & Subnet Overview
