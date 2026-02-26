# AWS Well-Architected & Cloud Adoption Framework Assessment

### Two-Tier Web Application Migration — Dakore Gideon

---

## Project Overview

This repository contains a complete assessment of a two-tier web application migration from on-premises infrastructure to AWS. The evaluation applies both the **AWS Well-Architected Framework (WAF)** and the **AWS Cloud Adoption Framework (CAF)** to ensure the migration aligns with AWS best practices from day one.

---

## Repository Structure

```
aws-waf-caf-repo/
│
├── README.md                        ← This file — explains the approach
├── aws_waf_caf_assessment.md        ← Full report (Tasks 1–4 + Reflection)
└── architecture/
    └── architecture_description.md  ← Improved architecture design (Task 4)
```

---

## Approach

### Step 1 — Understand the Existing Architecture (Task 1)

I started by identifying the components of the existing two-tier on-premises workload: a frontend web server and a backend database server. I then catalogued the key risks and weaknesses inherent in a typical on-premises setup of this type — including single points of failure, lack of automated backups, overly permissive security configurations, and no auto-scaling capability.

### Step 2 — WAF Evaluation (Task 2)

I applied all five pillars of the AWS Well-Architected Framework — Operational Excellence, Security, Reliability, Performance Efficiency, and Cost Optimization — to the workload. For each pillar, I identified one strength (what the migration already does well), one area for improvement (what the current design lacks), and a specific AWS service or feature that addresses it. Findings are presented in a structured table.

### Step 3 — CAF Readiness Analysis (Task 3)

I assessed the organisation's readiness for cloud transformation across all six CAF perspectives: Business, People, Governance, Platform, Security, and Operations. Each perspective is documented with a 150–200 word summary identifying the current state, key gaps, and specific enablers needed for a successful migration.

### Step 4 — Improved Architecture Design (Task 4)

Based on the WAF and CAF findings, I designed a revised AWS architecture that addresses all five WAF pillars. The design introduces Multi-AZ deployment, managed services, automated backups, fine-grained IAM policies, auto-scaling groups, and a CloudFront CDN layer. The full architecture is described in `architecture/architecture_description.md`.

---

## Submission Details

| Field   | Value                                                           |
| ------- | --------------------------------------------------------------- |
| Learner | Dakore Gideon                                                   |
| Course  | AWS Cloud Practitioner / Solutions Architecture — AmaliTech LMS |
| Date    | February 25, 2026                                               |
| Lab     | Design and Evaluate an AWS Solution Using WAF and CAF           |

---

## Key Frameworks Applied

- **AWS Well-Architected Framework (WAF)** — 5 Pillars
- **AWS Cloud Adoption Framework (CAF)** — 6 Perspectives
- **AWS Well-Architected Tool** — Used as validation reference
