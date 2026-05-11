# ☁️ AWS Notes — Index

> Structured day-by-day notes from the AWS module of the DevOps MultiCloud course.
> Each note covers exactly what was taught that day — with ASCII diagrams, commands, tables, and key takeaways.

---

## 📅 Notes Index

| Day | Date | File | Topics Covered |
|-----|------|------|---------------|
| 01 | 12 Apr | [day-01-aws-intro.md](day-01-aws-intro.md) | Cloud basics, AWS market share, global infrastructure, pay-as-you-go |
| 02 | 13 Apr | [day-02-regions-az.md](day-02-regions-az.md) | Regions, Availability Zones, multi-AZ HA, disaster recovery |
| 03 | 14 Apr | [day-03-devops-vpc-subnet-intro.md](day-03-devops-vpc-subnet-intro.md) | DevOps role, on-prem vs cloud, VPC & subnet intro |
| 04 | 15 Apr | [day-04-security-groups.md](day-04-security-groups.md) | Security Groups, firewall, inbound/outbound rules, ports |
| 05 | 16 Apr | [day-05-tcp-udp-vpc-cidr.md](day-05-tcp-udp-vpc-cidr.md) | TCP vs UDP, VPC sizing, CIDR calculation, subnet creation |
| 06 | 17 Apr | [day-06-ec2-igw-routetable.md](day-06-ec2-igw-routetable.md) | Key pair auth, Internet Gateway, Route Table, public subnet |
| 07 | 20 Apr | [day-07-custom-networking-ssh.md](day-07-custom-networking-ssh.md) | Public/private subnets, SSH hands-on, troubleshooting |
| 08 | 21 Apr | [day-08-bastion-nat-intro.md](day-08-bastion-nat-intro.md) | Bastion host pattern, NAT gateway intro |
| 09 | 22 Apr | [day-09-nat-gateway-deep-dive.md](day-09-nat-gateway-deep-dive.md) | Stateful NAT, port mapping, outbound-only security model |
| 10 | 23 Apr | [day-10-web-app-deployment.md](day-10-web-app-deployment.md) | Nginx install, HTML deploy, port 80, SG rules |
| 11 | 29 Apr | [day-11-alb-target-groups.md](day-11-alb-target-groups.md) | ALB, Target Groups, AZ matching, Round Robin |
| 12 | 30 Apr | [day-12-alb-handson-elastic-ip.md](day-12-alb-handson-elastic-ip.md) | ALB hands-on, Listener, health checks, Elastic IP |
| 13 | 1 May  | [day-13-path-based-routing.md](day-13-path-based-routing.md) | ALB vs NLB, path-based routing, 1 ALB + multiple TGs |
| 14 | 2 May  | [day-14-asg-dynamic-scaling.md](day-14-asg-dynamic-scaling.md) | Auto Scaling Group, Launch Template, horizontal scaling |
| 15 | 4 May  | [day-15-asg-nginx-enable-stress-test.md](day-15-asg-nginx-enable-stress-test.md) | systemctl enable, ELB health checks, stress-ng testing |
| 16 | 5 May  | [day-16-nlb.md](day-16-nlb.md) | Network LB, TCP protocol, static IP, NLB+ALB integration |
| 17 | 6 May  | [day-17-multi-path-eni.md](day-17-multi-path-eni.md) | ENI, multi-app single server, OS challenges |
| 18 | 7 May  | [day-18-s3.md](day-18-s3.md) | S3 intro, buckets, objects, versioning basics |
| 19 | 8 May  | [day-19-s3-deep-dive.md](day-19-s3-deep-dive.md) | Multi-AZ replication, presigned URLs, S3 Select vs Athena, static website hosting |
| 20 | 9 May  | [day-20-s3-storage-classes-intro.md](day-20-s3-storage-classes-intro.md) | 7 storage classes overview, cost tiers, minimum billing periods |
| 21 | 11 May | [day-21-s3-storage-classes-lifecycle.md](day-21-s3-storage-classes-lifecycle.md) | Storage classes deep dive, lifecycle rules, reverse proxy with ALB ✅ |

---

## 🗂️ Topics Covered So Far

```
✅ Global Infrastructure    — Regions, AZs, Data Centres
✅ Networking               — VPC, Subnets, CIDR, Route Tables, IGW
✅ Security                 — Security Groups, Ports, TCP/UDP
✅ Connectivity             — Key Pairs, Bastion Host, NAT Gateway, Elastic IP, ENI
✅ Load Balancing           — ALB, NLB, Target Groups, Path-based Routing, Reverse Proxy
✅ Auto Scaling             — ASG, Launch Template, Dynamic Scaling, stress-ng
🔄 Storage                  — S3 Buckets, Objects, Versioning, Storage Classes, Lifecycle Rules
🔜 Coming Next              — IAM, CloudWatch, CloudFront, EKS, RDS, Lambda
```

---

> 📌 **Note:** Each file covers only what was taught that specific day.
> No topic is duplicated across notes.
