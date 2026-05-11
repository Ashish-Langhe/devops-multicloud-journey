# Day 19 — S3 Deep Dive: Versioning, Presigned URLs & Static Hosting
**Date:** 8th May 2025
**Topic:** Amazon S3 — Advanced Concepts & Hands-On

---

## 1. S3 — Quick Recap

```
S3 = Simple Storage Service
- Unlimited storage, auto-scalable
- Regional service (bucket created per region)
- Stores: Objects (files) inside Buckets
```

> **Rule:** Both the **Bucket** AND the **Object** must be public for an end user to access a file directly.

---

## 2. S3 Multi-AZ Replication (Default Behaviour)

S3 automatically stores every object across **minimum 3 AZs** in the region.

```
Region: Mumbai
Upload: app.py

        [app.py uploaded]
              |
     +--------+---------+
     |         |         |
  AZ-1a     AZ-1b     AZ-1c
  app.py    app.py    app.py
```

- You upload once → AWS handles replication behind the scenes
- No manual configuration needed for standard class

---

## 3. S3 Object Access Methods

| Method          | Description                                                                 | Access Type  |
|-----------------|-----------------------------------------------------------------------------|--------------|
| **Object URI**  | Permanent public URL — bucket + object both must have public access enabled | Public       |
| **Presigned URL** | Time-limited URL — works even if bucket/object is private               | Temporary    |

### Presigned URL
- Anyone with the presigned URL can access the object **until it expires**
- Bucket and object can remain **private**
- Use case: share a file securely for a limited time (e.g., download links)

```
Presigned URL = S3 Object URL + Signature + Expiry Token
```

---

## 4. S3 Versioning

```
Versioning = Keep multiple variants of the same object in one bucket
```

- Protects against: **accidental deletes** and **accidental overwrites**
- Every upload of the same key creates a new **Version ID**
- You can **restore** any previous version at any time

```
Bucket: testdevprodtrtfe
Object: app.py

  Version 1 (latest) → Version ID: YHjkKPEJk0CYjvBOZL08SKO9hLD...  [347 B]
  Version 2 (older)  → Version ID: d5tVfy8hrmOdrfD5SZ3f4ea4IBg...  [171 B]
```

> Enable versioning on bucket → Settings → Enable → Confirm

---

## 5. SQL Queries on S3 (S3 Select)

```
S3 Select = Run limited SQL queries directly on S3 objects (CSV, JSON, Parquet)
```

- Works for **simple, lightweight queries**
- For **complex queries** → use **Amazon Athena**

```
Simple query example:
  SELECT * FROM s3object WHERE marks > 30

Output:
  sno, date, score, marks
  1, 05-06-2026, 34, 80
```

| Tool          | Use Case                     |
|---------------|------------------------------|
| S3 Select     | Simple queries, limited data |
| Amazon Athena | Complex SQL, large datasets  |

---

## 6. S3 Static Website Hosting

S3 can host **static front-end projects** directly without any server.

### What S3 Can Host:
- HTML, CSS, JavaScript, Images

### Architecture:

```
Option A — Direct Public Hosting:
  User --> S3 Bucket (public) --> Static Website Endpoint

Option B — With CloudFront CDN (Recommended):
  User --> CloudFront --> S3 Bucket (private)
                              (like ALB in front of EC2)
```

### Steps to Enable Static Hosting:

```
1. Create bucket
2. Upload HTML/CSS/images
3. Disable "Block Public Access"
4. Enable Static Website Hosting (Properties tab)
5. Add Bucket Policy (allow s3:GetObject for everyone)
6. Open the website endpoint URL
```

---

## 7. S3 Static Website vs EC2

| Feature            | S3 Static Website      | EC2                  |
|--------------------|------------------------|----------------------|
| Server management  | No server needed       | Need server          |
| Cost               | Very low               | Higher               |
| Scaling            | Automatic              | Manual / ASG         |
| Maintenance        | None                   | OS patches needed    |
| High availability  | Built-in               | Need setup           |
| Load balancer      | Not needed             | Often needed         |
| Auto scaling       | Automatic              | Configure ASG        |
| Storage            | Unlimited              | Limited disk         |
| Performance        | Fast with CloudFront   | Depends on EC2       |
| Security patching  | AWS manages            | You manage           |

> Use S3 static hosting for front-end only apps (React, Angular, plain HTML).
> For backend/APIs → EC2 or Lambda.

---

## Key Takeaways

| Concept           | What to Remember                                                    |
|-------------------|---------------------------------------------------------------------|
| S3 Default        | Unlimited, scalable, regional, minimum 3 AZs per region            |
| Public Access     | Both bucket + object must be public for direct URL access           |
| Presigned URL     | Time-limited access without making bucket public                    |
| Versioning        | Multiple versions per object; easy rollback                         |
| S3 Select         | Simple SQL on S3; use Athena for complex queries                    |
| Static Hosting    | Host HTML/CSS/JS directly; no server, auto-scale, very cheap        |
| With CloudFront   | Keep bucket private, serve via CDN for performance + security       |

---

**Next:** [Day 20 — S3 Storage Classes Overview](day-20-s3-storage-classes-intro.md)
