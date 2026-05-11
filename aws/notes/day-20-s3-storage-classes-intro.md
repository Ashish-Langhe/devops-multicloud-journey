# Day 20 — S3 Storage Classes: Introduction & Classification
**Date:** 9th May 2025
**Topic:** Amazon S3 — Storage Classes Overview

---

## 1. Why Storage Classes?

Not all data is accessed equally.
- Some files are needed **every day** (website assets)
- Some files are needed **once a month** (DR backups)
- Some files are **never accessed** but must be kept (legal archives)

S3 Storage Classes let you **pay only for what you need** — match the class to the access pattern.

---

## 2. The 5 Tiers (Concept)

```
Access Frequency  HIGH ──────────────────────────────────────► LOW
                  |                                              |
           S3 Standard                                  Deep Glacier
           (frequent)                                  (long-term backup)

Tiers:
  1. Standard (frequent)
  2. Infrequent (Standard-IA / One Zone-IA)
  3. Glacier Instant Retrieval
  4. Glacier Flexible Retrieval
  5. Deep Glacier
```

---

## 3. Storage Classes at a Glance

| Storage Class               | Use Case                | Retrieval Time  | Approx Cost/GB |
|-----------------------------|-------------------------|-----------------|----------------|
| S3 Standard                 | Frequently accessed     | Milliseconds    | $0.023         |
| S3 Intelligent-Tiering      | Unknown access patterns | Milliseconds    | ~$0.023        |
| S3 Standard-IA              | Infrequent access       | Milliseconds    | ~$0.0125       |
| S3 One Zone-IA              | Infrequent + single AZ  | Milliseconds    | ~$0.01         |
| Glacier Instant Retrieval   | Rare access             | Milliseconds    | ~$0.004        |
| Glacier Flexible Retrieval  | Archive                 | Minutes/Hours   | ~$0.0036       |
| Glacier Deep Archive        | Long-term backup        | Hours           | ~$0.00099      |

---

## 4. Each Class — Quick Notes

### S3 Standard (Default)
- For: websites, apps, frequently accessed files
- Multi-AZ: Yes | Low latency | High throughput
- No minimum storage duration

### S3 Intelligent-Tiering
- For: data with **unknown or changing** access patterns
- AWS automatically moves objects between tiers based on access
- Small monitoring fee per object

### S3 Standard-IA (Infrequent Access)
- For: backups, DR files, monthly access data
- Multi-AZ: Yes | Lower cost | **Retrieval fee applies**
- Minimum storage duration: **30 days**

### S3 One Zone-IA
- Same as Standard-IA but stored in **only 1 AZ**
- Cheaper but **not recommended** — data lost if AZ fails
- Use only for re-creatable / non-critical data

### Glacier Instant Retrieval
- For: archive data accessed ~once per quarter
- Retrieval: **Milliseconds** (unlike other Glacier types)
- Minimum storage duration: **90 days**

### Glacier Flexible Retrieval
- For: archive data, accessed ~once per year
- Retrieval: **Minutes to Hours**
- Minimum storage duration: **90 days**

### Glacier Deep Archive
- For: long-term backup, compliance data, 7–10 year retention
- Retrieval: **Hours**
- Minimum storage duration: **180 days**
- Cheapest option: ~$0.00099/GB (~96% cheaper than Standard)

---

## 5. Glacier Types — Retrieval Comparison

```
Glacier Instant Retrieval  → Milliseconds  (instant, like Standard)
Glacier Flexible Retrieval → Minutes/Hours (plan ahead)
Glacier Deep Archive       → Hours         (cheapest, slowest)
```

---

## 6. Cost Savings vs Standard

| Storage Class               | Approx Savings |
|-----------------------------|----------------|
| Standard                    | 0% (baseline)  |
| Intelligent-Tiering         | 0–10%          |
| Standard-IA                 | ~45% cheaper   |
| One Zone-IA                 | ~57% cheaper   |
| Glacier Instant Retrieval   | ~82% cheaper   |
| Glacier Flexible Retrieval  | ~84% cheaper   |
| Glacier Deep Archive        | ~96% cheaper   |

---

## 7. Minimum Waiting Period — Key Rule

```
Standard          → No waiting period (access anytime, no download charges)
Infrequent (IA)   → 30 days minimum billing period
Glacier           → 90 days minimum billing period
Deep Glacier      → 180 days minimum billing period
```

> **Example:** Upload a file to Standard-IA today. Delete it tomorrow.
> You are still **billed for 30 days** of Standard-IA storage.

---

## Key Takeaways

| Concept                     | What to Remember                                              |
|-----------------------------|---------------------------------------------------------------|
| Storage class = access tier | Match class to how often you access data                      |
| Standard                    | Default, no restrictions, no waiting, highest cost            |
| Standard-IA                 | 30-day min billing, retrieval fee, multi-AZ                   |
| One Zone-IA                 | Single AZ, cheapest IA, not recommended for critical data     |
| Glacier types               | 3 variants: Instant (ms), Flexible (min/hr), Deep (hours)     |
| Minimum billing             | IA = 30d, Glacier = 90d, Deep = 180d                          |
| Biggest saving              | Deep Archive = ~96% cheaper than Standard                     |

---

**Next:** [Day 21 — S3 Storage Classes Deep Dive & Lifecycle Rules](day-21-s3-storage-classes-lifecycle.md)
