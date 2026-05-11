# Day 21 — S3 Storage Classes Deep Dive, Lifecycle Rules & Reverse Proxy with ALB
**Date:** 11th May 2025
**Topic:** S3 Storage Classes (Hands-On) + S3 Lifecycle + Reverse Proxy Task Completed

---

## 1. S3 Standard — Deep Dive

```
S3 Standard = Default storage class
```

- Default for every uploaded object
- No size limit per object
- **No waiting period** — access data anytime
- **No download/retrieval charges**
- Multi-AZ: Yes | High throughput | Low latency

Use cases:
- Websites
- Apps
- Frequently accessed files

---

## 2. S3 Standard-IA — Deep Dive

```
IA = Infrequent Access
```

- Multi-AZ: Yes
- Lower storage cost than Standard
- **Retrieval fee applies** (per GB retrieved)
- Minimum billable storage: **30 days**

### 30-Day Minimum Billing — Explained:

```
Day 0 : Upload file to Standard-IA
Day 1 : You delete the file
Result: You are still billed for 30 days of Standard-IA pricing
```

```
Day 0 : Upload file to Standard-IA
Day 40: You delete the file
Result: Billed for 40 days (actual usage > minimum)
        → get full Standard-IA discount
```

Approx cost: ~$0.0125 per GB/month (~45% cheaper than Standard)

Use cases: Backups, DR files, monthly access data

---

## 3. S3 One Zone-IA

- Same as Standard-IA but stored in **1 AZ only**
- **Not recommended** — if that AZ fails, data is lost
- Only for **re-creatable or non-critical** data
- ~57% cheaper than Standard

> Avoid for production data — use Standard-IA instead.

---

## 4. Glacier Storage Classes — Detailed

```
Glacier = Archive storage. Cheap. Not for daily access.

Minimum waiting periods:
  Glacier Instant / Flexible  →  90 days
  Glacier Deep Archive        → 180 days
```

| Glacier Type              | Retrieval Time  | Min Storage | Cost/GB   |
|---------------------------|-----------------|-------------|-----------|
| Glacier Instant Retrieval | Milliseconds    | 90 days     | ~$0.004   |
| Glacier Flexible Retrieval| Minutes / Hours | 90 days     | ~$0.0036  |
| Glacier Deep Archive      | Hours           | 180 days    | ~$0.00099 |

### Key Difference:

```
Glacier Instant   → Fast retrieval (ms), but still archive pricing
Glacier Flexible  → Plan ahead (minutes to hours)
Glacier Deep      → Bulk backup, cheapest possible (~96% off Standard)
```

---

## 5. Storage Class Comparison (AWS Console View)

From AWS Console → S3 → Edit Storage Class:

```
+-------------------------------+-----------+--------------+------------------+
| Storage Class                 | Min AZs   | Min Duration | Min Obj Size     |
+-------------------------------+-----------+--------------+------------------+
| Standard                      | ≥ 3       | None         | None             |
| Intelligent-Tiering           | ≥ 3       | None         | None             |
| Standard-IA                   | ≥ 3       | 30 days      | 128 KB           |
| One Zone-IA                   | 1         | 30 days      | 128 KB           |
| Glacier Instant Retrieval     | ≥ 3       | 90 days      | 128 KB           |
| Glacier Flexible Retrieval    | ≥ 3       | 90 days      | None             |
| Glacier Deep Archive          | ≥ 3       | 180 days     | None             |
+-------------------------------+-----------+--------------+------------------+
```

> Note: Standard-IA and Glacier Instant have a **128 KB minimum billable object size**.
> Small files (<128 KB) are billed as 128 KB.

---

## 6. S3 Lifecycle Rules

```
S3 Lifecycle = Automate the movement and deletion of objects
               across storage classes over time.
```

### What Lifecycle Rules Can Do:

```
1. Transition objects → move file from one storage class to another
2. Expiration        → delete current version after N days
3. Delete old versions → remove non-current (previous) versions
```

### Typical Lifecycle Flow:

```
Day 0      → Upload to S3 Standard
Day 30     → Auto-move to Standard-IA     (infrequently accessed)
Day 90     → Auto-move to Glacier         (archive)
Day 180    → Auto-move to Deep Archive    (long-term backup)
Day 365    → Auto-delete (expiration)
```

### Lifecycle Rule — Diagram:

```
[S3 Standard]
      |
      | (after 30 days)
      v
[Standard-IA]
      |
      | (after 90 days)
      v
[Glacier Flexible]
      |
      | (after 180 days)
      v
[Deep Archive]
      |
      | (after 365 days)
      v
[DELETED]
```

### Configuring a Lifecycle Rule (Console Steps):

```
S3 → Bucket → Management tab → Lifecycle Rules → Create rule

Options:
  - Apply to: all objects / prefix / tag
  - Transition: choose class + days
  - Expiration: delete after N days
  - Non-current versions: manage old versions separately
```

---

## 7. Reverse Proxy with ALB — Task Completed ✅

Completed hands-on task: configure **Nginx as Reverse Proxy** with **ALB** in front.

### What is a Reverse Proxy?

```
Normal:
  Client → Web Server (directly serves content)

Reverse Proxy:
  Client → Nginx (reverse proxy) → Backend App / EC2
```

- Nginx sits **in front** of the backend
- Forwards requests to the actual application server
- Hides internal server details from the client

### Architecture with ALB:

```
Internet
   |
   v
  ALB  (port 80/443)
   |
   v
 Nginx  (reverse proxy, port 80)
   |
   v
 App Server  (e.g., Node.js on port 3000)
```

### Nginx Reverse Proxy Config:

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## Key Takeaways

| Concept                      | What to Remember                                                         |
|------------------------------|--------------------------------------------------------------------------|
| S3 Standard                  | Default, no waiting, no retrieval fee, highest cost                      |
| Standard-IA                  | 30-day min billing, retrieval fee, good for monthly backups              |
| One Zone-IA                  | Single AZ only — avoid for critical data                                 |
| Glacier Instant              | Archive + fast (ms) retrieval, 90-day min                                |
| Glacier Flexible             | Archive, minutes/hours retrieval, 90-day min                             |
| Glacier Deep Archive         | Cheapest (~96% off), hours retrieval, 180-day min                        |
| 128 KB minimum               | Standard-IA & Glacier Instant bill small files as 128 KB                 |
| S3 Lifecycle                 | Automate transitions + deletions based on age of object                  |
| Lifecycle: transition        | Move between storage classes automatically (e.g., Standard → Glacier)   |
| Lifecycle: expiration        | Auto-delete current or non-current versions after N days                 |
| Reverse Proxy (Nginx + ALB)  | ALB → Nginx (port 80) → Backend App; hides internals, enables routing    |

---

**Next:** [Day 22 — S3 continued / CloudFront](day-22-cloudfront.md)
