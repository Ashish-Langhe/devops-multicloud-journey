<div align="center">

# 🚀 DevOps MultiCloud Journey

### Learning DevOps hands-on — notes, diagrams & projects across AWS, Azure, GCP, Docker, K8s & CI/CD

[![GitHub last commit](https://img.shields.io/github/last-commit/Ashish-Langhe/devops-multicloud-journey?style=for-the-badge&color=00d4ff&labelColor=0d1117)](https://github.com/Ashish-Langhe/devops-multicloud-journey)
[![GitHub commit activity](https://img.shields.io/github/commit-activity/w/Ashish-Langhe/devops-multicloud-journey?style=for-the-badge&color=00d4ff&labelColor=0d1117)](https://github.com/Ashish-Langhe/devops-multicloud-journey)
[![GitHub repo size](https://img.shields.io/github/repo-size/Ashish-Langhe/devops-multicloud-journey?style=for-the-badge&color=00d4ff&labelColor=0d1117)](https://github.com/Ashish-Langhe/devops-multicloud-journey)
[![GitHub stars](https://img.shields.io/github/stars/Ashish-Langhe/devops-multicloud-journey?style=for-the-badge&color=yellow&labelColor=0d1117)](https://github.com/Ashish-Langhe/devops-multicloud-journey)

</div>

---

## 👋 About This Repo

> A **daily log** of my hands-on DevOps and Multi-Cloud learning journey.
> Every folder = a topic. Every file = a day of learning. Every commit = progress.

```
📝 Notes & Explanations    →  Markdown notes after every class
🏗️  Architecture Diagrams   →  ASCII architecture diagrams
💻 Scripts & Commands      →  Real CLI commands used hands-on
🔬 Projects                →  End-to-end hands-on projects
```

---

## 🛠️ Tech Stack

<div align="center">

**Cloud**

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![GCP](https://img.shields.io/badge/GCP-%234285F4.svg?style=for-the-badge&logo=googlecloud&logoColor=white)

**DevOps & Containers**

![Docker](https://img.shields.io/badge/Docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-%23EE0000.svg?style=for-the-badge&logo=ansible&logoColor=white)

**CI/CD**

![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-%23D24939.svg?style=for-the-badge&logo=jenkins&logoColor=white)

**OS & Scripting**

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Bash](https://img.shields.io/badge/Bash-%234EAA25.svg?style=for-the-badge&logo=gnubash&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)

**Monitoring**

![Grafana](https://img.shields.io/badge/Grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-%23E6522C.svg?style=for-the-badge&logo=prometheus&logoColor=white)

</div>

---

## 🗺️ Learning Roadmap

```
DevOps MultiCloud Journey
│
├── ✅ Phase 1 — AWS (In Progress)
│     ├── ✅ Global Infrastructure (Regions, AZs)
│     ├── ✅ VPC, Subnets, CIDR
│     ├── ✅ EC2, IGW, Route Tables
│     ├── ✅ Security Groups & Protocols
│     ├── ✅ Bastion Host & NAT Gateway
│     ├── ✅ Application Load Balancer (ALB)
│     ├── ✅ Target Groups & Elastic IP
│     ├── ✅ Path-Based Routing
│     ├── ✅ Auto Scaling Group (ASG)
│     ├── ✅ Network Load Balancer (NLB)
│     └── 🔜 S3, IAM, CloudWatch, EKS
│
├── 🔜 Phase 2 — Containerization
│     ├── 🔜 Docker
│     └── 🔜 Docker Compose
│
├── 🔜 Phase 3 — CI/CD
│     ├── 🔜 GitHub Actions
│     └── 🔜 Jenkins
│
├── 🔜 Phase 4 — Infrastructure as Code
│     ├── 🔜 Terraform
│     └── 🔜 CloudFormation
│
├── 🔜 Phase 5 — Azure & GCP
│
└── 🔜 Phase 6 — Monitoring
      ├── 🔜 Prometheus
      └── 🔜 Grafana
```

---

## ☁️ AWS Architecture Learned So Far

```
                        www.yourapp.com
                               │
                          Route 53 (DNS)
                               │
                    ┌─────────────────────┐
                    │   Internet Gateway  │
                    └──────────┬──────────┘
                               │
                    ┌─────────────────────┐
                    │  Application Load   │
                    │  Balancer  (ALB)    │
                    │  pub-subnet-1a + 1b │
                    └────────┬────────────┘
                             │
              ┌──────────────┴───────────────┐
              │                              │
   ┌──────────────────┐          ┌──────────────────┐
   │  Private Subnet  │          │  Private Subnet  │
   │    AZ: 1a        │          │    AZ: 1b        │
   │  ┌────────────┐  │          │  ┌────────────┐  │
   │  │  EC2 (App) │  │          │  │  EC2 (App) │  │
   │  └────────────┘  │          │  └────────────┘  │
   │   ASG manages ↕  │          │   ASG manages ↕  │
   └──────────────────┘          └──────────────────┘
              │
   ┌──────────────────┐
   │  Public Subnet   │
   │  Bastion Host ←──┼── Developer SSH
   │  NAT Gateway  ───┼──→ Private servers get internet
   └──────────────────┘
```

---

## 📅 Daily Commit Log

| Day | Date | Topic | Summary |
|-----|------|-------|---------|
| Day 01–02 | 12 Apr | AWS Intro | AWS overview, market share, global infra |
| Day 03 | 14 Apr | Regions & AZs | Regions, AZs, data centres, multi-AZ HA |
| Day 04 | 15 Apr | Security Groups | Firewall, inbound/outbound rules, ports |
| Day 05 | 16 Apr | Protocols & VPC | TCP/UDP, HTTP/HTTPS, VPC, CIDR |
| Day 06 | 17 Apr | EC2 + Networking | EC2 creation, IGW, Route Table, SSH |
| Day 07 | 20 Apr | Custom Networking | Public/Private subnets hands-on |
| Day 08 | 21 Apr | Bastion + NAT | Bastion host, NAT Gateway setup |
| Day 09 | 22 Apr | NAT Deep Dive | NAT stateful, port mapping, architecture |
| Day 10 | 23 Apr | Web Deployment | Nginx install, HTML app deploy on EC2 |
| Day 11 | 29 Apr | ALB + TG | Load balancer, target groups, health checks |
| Day 12 | 30 Apr | ALB Hands-on | Private server → ALB config, Elastic IP |
| Day 13 | 1 May | Path Routing | ALB path-based routing, 1 ALB + 4 TGs |
| Day 14 | 2 May | ASG | Auto Scaling, AMI, Launch Template |
| Day 16 | 5 May | NLB | Network LB, NLB + ALB integration |

---

## 📁 Repo Structure

```
devops-multicloud-journey/
│
├── README.md                          ← You are here
├── 00-roadmap.md                      ← Full learning checklist
├── .gitignore
│
├── aws/
│   └── notes/
│       ├── day-01-03-aws-overview-regions-az.md
│       ├── day-04-05-security-groups-protocols.md
│       ├── day-05-06-vpc-subnets-cidr.md
│       ├── day-06-07-ec2-igw-routetable.md
│       ├── day-08-09-bastion-nat-gateway.md
│       ├── day-10-web-app-deployment.md
│       ├── day-11-12-load-balancer-tg.md
│       ├── day-13-path-based-routing.md
│       ├── day-14-asg-dynamic-scaling.md
│       └── day-16-nlb.md
│
├── docker/            ← coming soon
├── kubernetes/        ← coming soon
├── terraform/         ← coming soon
├── azure/             ← coming soon
└── gcp/               ← coming soon
```

---

## 📌 Connect

<div align="center">

[![GitHub](https://img.shields.io/badge/GitHub-Ashish--Langhe-181717?style=for-the-badge&logo=github)](https://github.com/Ashish-Langhe)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/ashish-langhe)

**⭐ Star this repo if you find it helpful — it motivates daily learning!**

`Updated daily` • `Learning in public` • `DevOps × MultiCloud`

</div>
