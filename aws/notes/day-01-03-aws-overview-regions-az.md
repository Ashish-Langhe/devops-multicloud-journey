# Day 01-03 | AWS Overview, Regions & Availability Zones

**Dates:** 12th April – 14th April  
**Topics:** AWS Intro, Global Infrastructure, Regions, AZs, VPC Intro

---

## ☁️ What is AWS?

- AWS (Amazon Web Services) is a **cloud platform** where we get resources on a **rental basis**
- AWS is **globally distributed** — network is covered across all countries
- AWS holds **30-33% market share globally** (largest cloud provider)
- Offers **200+ services**
- Free tier available for practice
- First cloud in the market (2006) — pay-as-you-go model
- Cloud infrastructure revenue: **$91B** (Q4 2024)

---

## 🌍 AWS Global Infrastructure

### Hierarchy
```
Global
  └── Continent
        └── Region
              └── Availability Zone (AZ)
                    └── Data Centre
                          └── Group of Servers
```

### Region
- A **Region** is a **geographical location** — NOT a country
- Each region contains **multiple Availability Zones**
- AWS has **33+ Regions** globally
- Examples:
  - `us-east-1` → N. Virginia (USA)
  - `ap-south-1` → Mumbai (India)
  - `ap-south-2` → Hyderabad (India)

### Availability Zone (AZ)
- An AZ = **one or multiple Data Centres**
- Each region has **multiple AZs** (usually 3)
- AWS has **108+ AZs globally**
- Example for `ap-south-1` (Mumbai):
  - `ap-south-1a`
  - `ap-south-1b`
  - `ap-south-1c`

### Data Centre
- A Data Centre = **Group of Servers**

---

## 🔑 Key Notes

> ⚠️ While deploying any resource in AWS, we must select:
> 1. **Region**
> 2. **Availability Zone**
> 
> Rest everything — AWS provider takes care of.

> ✅ **Always deploy resources in different AZs** for failure management / disaster recovery.  
> If one AZ goes down, application keeps running in the other AZ.

---

## 🏗️ Architecture — Multi-AZ for High Availability

```
AWS Account
  └── Region (e.g., us-east-1)
        └── VPC
              ├── AZ-1a
              │     └── Public Subnet → EC2 Instance
              └── AZ-1b
                    └── Public Subnet → EC2 Instance
                          ↑
                   Elastic Load Balancer (distributes traffic)
                          ↑
                      Internet Gateway
                          ↑
                      Route 53 (DNS)
                          ↑
                      www.yoursite.com
```

> **Note:** Multi-AZ gives high availability — even if 1a datacentre fails, application keeps running in 1b without issues.

---

## 🆚 On-Premises vs Cloud

| On-Premises | Cloud (AWS) |
|-------------|-------------|
| Need to manage electricity, manpower, ventilation | AWS manages everything |
| High upfront cost | Pay as you go |
| Scalability is hard | Scalable on demand |
| High availability requires multiple data centres | Built-in multi-AZ support |

---

## 🔗 Role of DevOps

- DevOps is **responsible for deploying applications** to servers for end-users
- Can deploy to:
  - On-prem server (company's own data centre)
  - Cloud (AWS, Azure, GCP)

---

## 📌 What's Next

- Understanding VPC (Virtual Private Cloud)
- EC2 Instances
- S3 Buckets
