# Day 18 | S3 — Simple Storage Service

**Date:** 7th May  
**Topics:** S3 intro, Buckets, Objects, Versioning, Access control

---

## 🪣 What is S3?

- S3 = **Simple Storage Service**
- AWS's **object storage** service
- Store **any type of file** — images, videos, logs, backups, code, HTML, etc.
- **Unlimited storage** — no size limit
- **Global service** (console is global), but **Buckets are regional**

```
AWS (Global)
  └── S3 (Global Service)
        └── Bucket (Regional — created in specific region)
              └── Folder (optional — for organisation)
                    └── Objects (actual files)
```

---

## 🗂️ S3 Structure

| Component | Description |
|-----------|-------------|
| **S3** | The global storage service |
| **Bucket** | Storage container (like a hard drive partition) — **regional** |
| **Folder** | Optional — organise objects inside bucket |
| **Object** | The actual file stored (image, video, log, zip, etc.) |

---

## 🔑 Key S3 Properties

| Property | Detail |
|----------|--------|
| **Storage limit** | Unlimited — store as many objects as you want |
| **Bucket scope** | Regional — bucket belongs to one region |
| **Service scope** | Global — S3 console is global |
| **Default access** | ❌ Private — bucket is private by default |
| **Public access** | ✅ Can be changed to public if needed |
| **Bucket name** | Must be **globally unique** across all AWS accounts |
| **Versioning** | ✅ S3 offers versioning — keeps history of file changes |

---

## 🌍 Global vs Regional

```
S3 Service  ──────────────────────── GLOBAL
  └── Bucket (ap-south-1 / Mumbai) ─ REGIONAL
  └── Bucket (us-east-1 / N.Virginia) ─ REGIONAL
```

> Even though S3 is a global service, each **bucket** lives in a specific region.  
> Choose the region closest to your users for lower latency.

---

## 🔒 Access Control

- By default → bucket and all objects are **private**
- To make accessible publicly:
  1. Disable "Block all public access" on bucket
  2. Add a bucket policy allowing public read
  3. Or enable public access on individual objects

### Use cases
| Access Type | Example |
|-------------|---------|
| Private | Storing application logs, backups, Terraform state files |
| Public | Hosting a static website, sharing images/assets publicly |

---

## 📦 Versioning

- S3 can keep **multiple versions** of the same file
- If you upload a new version of `index.html`, the old version is still saved
- You can **restore** to any previous version
- Useful for: rollback, auditing, accidental delete protection

```
index.html v1 (original)
index.html v2 (updated)
index.html v3 (latest)  ← current
```

---

## 🏗️ S3 Use Cases in DevOps

| Use Case | How |
|----------|-----|
| **Static website hosting** | Store HTML/CSS/JS in S3, enable public access |
| **Terraform state storage** | Store `.tfstate` files securely in S3 |
| **Application logs** | Write app logs directly to S3 |
| **Backup & DR** | Store EC2 AMI backups, database dumps |
| **Artifact storage** | Store build artifacts from CI/CD pipelines |
| **Media storage** | Store images/videos for web apps |

---

## 📌 Key Takeaways

- S3 = unlimited object storage, global service, buckets are regional
- Bucket name must be **globally unique** — no two buckets anywhere in AWS can share a name
- Default = private; you control what becomes public
- Versioning = keeps old file versions — great for rollback
- S3 is one of the most used AWS services in real DevOps workflows (state files, logs, artifacts, backups)
