<div align="center">

# 🚀 DevOps MultiCloud Journey

<br/>

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&pause=800&color=00D4FF&center=true&vCenter=true&multiline=false&width=600&lines=Learning+DevOps+%7C+One+Commit+a+Day+%E2%9C%85;AWS+%7C+Azure+%7C+GCP+%7C+MultiCloud+%E2%98%81%EF%B8%8F;Docker+%7C+K8s+%7C+Terraform+%7C+CI%2FCD+%F0%9F%9B%A0%EF%B8%8F;Notes+%7C+Diagrams+%7C+Projects+%7C+Scripts+%F0%9F%93%9D)](https://git.io/typing-svg)

<br/>

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![GCP](https://img.shields.io/badge/GCP-%234285F4.svg?style=for-the-badge&logo=googlecloud&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white)

<br/>

[![Last Commit](https://img.shields.io/github/last-commit/Ashish-Langhe/devops-multicloud-journey?style=flat-square&color=00d4ff&label=Last+Commit)](https://github.com/Ashish-Langhe/devops-multicloud-journey/commits)
[![Commit Activity](https://img.shields.io/github/commit-activity/w/Ashish-Langhe/devops-multicloud-journey?style=flat-square&color=00d4ff&label=Weekly+Commits)](https://github.com/Ashish-Langhe/devops-multicloud-journey/commits)
[![Repo Size](https://img.shields.io/github/repo-size/Ashish-Langhe/devops-multicloud-journey?style=flat-square&color=00d4ff)](https://github.com/Ashish-Langhe/devops-multicloud-journey)
[![Stars](https://img.shields.io/github/stars/Ashish-Langhe/devops-multicloud-journey?style=flat-square&color=yellow)](https://github.com/Ashish-Langhe/devops-multicloud-journey/stargazers)

</div>

---

## 👨‍💻 About This Repo

> A **day-by-day DevOps & Multi-Cloud learning journal** — built in public.
> Every topic has structured notes, real CLI commands, architecture diagrams, and hands-on projects.
> **Updated after every class session.**

```
📝  Notes          →  Structured markdown notes from every class
🏗️   Architectures  →  ASCII architecture diagrams of what I built
💻  Commands       →  Real CLI commands used during hands-on
🔬  Projects       →  End-to-end mini projects
```

---

## ☁️ AWS Architecture — Built So Far

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
                     │  Network Load       │
                     │  Balancer (NLB)     │  ← Static Elastic IP
                     │  TCP · public subnets│
                     └──────────┬──────────┘
                                │
                     ┌─────────────────────┐
                     │  App Load Balancer  │
                     │  (ALB) · HTTP/HTTPS │  ← Path-based routing
                     │  public subnet 1a+1b│
                     └────────┬────────────┘
                              │
             ┌────────────────┴─────────────────┐
             │                                  │
  ┌──────────────────┐               ┌──────────────────┐
  │  Private Subnet  │               │  Private Subnet  │
  │    AZ: 1a        │               │    AZ: 1b        │
  │  ┌────────────┐  │               │  ┌────────────┐  │
  │  │  EC2 (App) │  │               │  │  EC2 (App) │  │
  │  │  Nginx     │  │               │  │  Nginx     │  │
  │  └────────────┘  │               │  └────────────┘  │
  │  ASG scales ↕    │               │  ASG scales ↕    │
  └──────────────────┘               └──────────────────┘
             │
  ┌──────────────────┐
  │  Public Subnet   │
  │  Bastion Host  ←─┼── Developer SSH
  │  NAT Gateway   ──┼──→ Outbound internet for private servers
  └──────────────────┘
             │
  ┌──────────────────┐
  │  S3 Bucket       │  ← Logs · Backups · Artifacts · Static files
  └──────────────────┘
```

---

## 🗺️ Learning Roadmap

```
devops-multicloud-journey
│
├── 🔄 Phase 1 — AWS (In Progress — Day 18)
│   │
│   ├── ✅ Global Infrastructure
│   │     Regions · Availability Zones · Data Centres
│   │
│   ├── ✅ Networking
│   │     VPC · Subnets · CIDR · Route Tables · Internet Gateway
│   │
│   ├── ✅ Security
│   │     Security Groups · Protocols · HTTP/HTTPS · TCP/UDP
│   │
│   ├── ✅ Connectivity
│   │     Key Pair · Bastion Host · NAT Gateway · Elastic IP · ENI
│   │
│   ├── ✅ Load Balancing
│   │     ALB · NLB · Target Groups · Path-Based Routing
│   │     Health Checks · Round Robin · Listener Rules
│   │
│   ├── ✅ Auto Scaling
│   │     ASG · Launch Template · Dynamic Scaling
│   │     Horizontal vs Vertical · stress-ng testing
│   │
│   ├── 🔄 Storage
│   │     S3 · Buckets · Objects · Versioning · Access Control
│   │
│   └── 🔜 Coming Next
│         IAM · CloudWatch · EKS · RDS · Lambda
│
├── 🔜 Phase 2 — Containerization
│     Docker · Docker Compose · Container Registry
│
├── 🔜 Phase 3 — CI/CD
│     GitHub Actions · Jenkins · Pipelines
│
├── 🔜 Phase 4 — Infrastructure as Code
│     Terraform · CloudFormation · Ansible
│
├── 🔜 Phase 5 — Azure & GCP
│     Azure Core · GCP Core · Multi-Cloud strategies
│
└── 🔜 Phase 6 — Monitoring & Observability
      Prometheus · Grafana · CloudWatch · ELK Stack
```

---

## 📅 Daily Learning Log

| Day | Date | Topic | Key Concepts |
|-----|------|-------|-------------|
| 01 | 12 Apr | AWS Intro | Cloud platform, market share, global infra, pay-as-you-go |
| 02 | 13 Apr | Regions & AZs | 33+ regions, 108+ AZs, multi-AZ HA, disaster recovery |
| 03 | 14 Apr | DevOps + VPC Intro | DevOps role, on-prem vs cloud, VPC & subnet intro |
| 04 | 15 Apr | Security Groups | Firewall, inbound/outbound rules, ports, HTTP/HTTPS |
| 05 | 16 Apr | TCP/UDP + CIDR | TCP vs UDP, VPC sizing, CIDR calculation, subnet creation |
| 06 | 17 Apr | Key Pair + IGW + RT | Key pair auth, Internet Gateway, Route Table, public subnet |
| 07 | 20 Apr | Custom Networking | Public/private subnets, SSH hands-on, troubleshooting |
| 08 | 21 Apr | Bastion + NAT Intro | Jump host pattern, NAT gateway intro, app deployment need |
| 09 | 22 Apr | NAT Deep Dive | Stateful NAT, port mapping, outbound-only security model |
| 10 | 23 Apr | Web Deployment | Nginx install, HTML deploy, port 80, SG rules |
| 11 | 29 Apr | ALB + TG | Application LB, Target Groups, AZ matching, Round Robin |
| 12 | 30 Apr | ALB Hands-on | Private server config, Listener, health checks, Elastic IP |
| 13 | 1 May | Path Routing | ALB vs NLB, path-based routing, 1 ALB + multiple TGs |
| 14 | 2 May | ASG | Auto Scaling Group, Launch Template, horizontal scaling |
| 15 | 4 May | ASG Deep Dive | systemctl enable, ELB health checks, stress-ng testing |
| 16 | 5 May | NLB | Network LB, TCP protocol, static IP, NLB+ALB integration |
| 17 | 6 May | ENI + Multi-path | Elastic Network Interface, multi-app single server, OS challenges |
| 18 | 7 May | S3 | Simple Storage Service, buckets, objects, versioning |

---

## 📁 Repo Structure

```
devops-multicloud-journey/
│
├── README.md
├── 00-roadmap.md
├── .gitignore
│
├── aws/
│   └── notes/
│       ├── README.md
│       ├── day-01-aws-intro.md
│       ├── day-02-regions-az.md
│       ├── day-03-devops-vpc-subnet-intro.md
│       ├── day-04-security-groups.md
│       ├── day-05-tcp-udp-vpc-cidr.md
│       ├── day-06-ec2-igw-routetable.md
│       ├── day-07-custom-networking-ssh.md
│       ├── day-08-bastion-nat-intro.md
│       ├── day-09-nat-gateway-deep-dive.md
│       ├── day-10-web-app-deployment.md
│       ├── day-11-alb-target-groups.md
│       ├── day-12-alb-handson-elastic-ip.md
│       ├── day-13-path-based-routing.md
│       ├── day-14-asg-dynamic-scaling.md
│       ├── day-15-asg-nginx-enable-stress-test.md
│       ├── day-16-nlb.md
│       ├── day-17-multi-path-eni.md
│       └── day-18-s3.md
│
├── docker/            ← coming soon
├── kubernetes/        ← coming soon
├── terraform/         ← coming soon
├── azure/             ← coming soon
└── gcp/               ← coming soon
```

---

## 🛠️ Full Tech Stack

<div align="center">

**Cloud Platforms**

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![GCP](https://img.shields.io/badge/GCP-%234285F4.svg?style=for-the-badge&logo=googlecloud&logoColor=white)

**Containers & Orchestration**

![Docker](https://img.shields.io/badge/Docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)

**Infrastructure as Code**

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

## 📌 Connect

<div align="center">

[![GitHub](https://img.shields.io/badge/GitHub-Ashish--Langhe-181717?style=for-the-badge&logo=github)](https://github.com/Ashish-Langhe)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/ashish-langhe)

<br/>

**⭐ Star this repo if you find it helpful!**

`Learning in public` • `Updated daily` • `DevOps × MultiCloud`

</div>
