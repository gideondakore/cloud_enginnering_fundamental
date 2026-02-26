# aws_waf_caf_assessment.md
# AWS Well-Architected & Cloud Adoption Framework Assessment Report

**Learner:** Dakore Gideon
**Date:** February 25, 2026
**Lab:** Design and Evaluate an AWS Solution Using the Well-Architected and Cloud Adoption Frameworks
**Workload:** Two-Tier Web Application (Frontend + Backend Database) — On-Premises to AWS Migration

---

## Task 1 — Review of the Existing Architecture

### 1.1 Workload Components

The existing on-premises two-tier web application consists of the following components:

| Component | Description |
|---|---|
| **Frontend Web Server** | A single physical or virtual server running the web application (e.g., Apache/Nginx + application code). Handles all HTTP/HTTPS traffic from users. |
| **Backend Database Server** | A single on-premises database server (e.g., MySQL or PostgreSQL). Stores all application data including user records, transactions, and content. |
| **Network Layer** | On-premises network with a hardware firewall. Traffic flows directly from the internet to the web server, and from the web server to the database. |
| **Storage** | Local disks attached to each server. No centralised or managed storage solution. |
| **Backup** | Manual or ad-hoc backup processes. No automated, policy-driven backup strategy. |
| **Monitoring** | Limited to server-level OS monitoring tools (e.g., Nagios, or none at all). No application-level observability. |

---

### 1.2 Identified Risks and Weaknesses

| # | Risk / Weakness | Category |
|---|---|---|
| 1 | **Single point of failure** — both the web server and database run on single machines. Any hardware failure causes total downtime. | Reliability |
| 2 | **No redundancy / single-AZ equivalent** — all compute and storage is co-located. There is no geographic or availability zone separation. | Reliability |
| 3 | **No automated backup strategy** — data loss risk is high. Recovery from failure would be slow and manual. | Reliability / Operational Excellence |
| 4 | **Overly permissive firewall rules** — on-premises firewalls are often configured with broad allow rules (e.g., 0.0.0.0/0 on port 22 or 3306), exposing servers to attack. | Security |
| 5 | **No encryption at rest or in transit** — database and web traffic may be unencrypted, exposing sensitive data. | Security |
| 6 | **No auto-scaling** — the architecture cannot handle traffic spikes. Over-provisioning wastes money; under-provisioning causes outages. | Performance Efficiency / Cost Optimization |
| 7 | **Manual deployments and operations** — no CI/CD pipeline, no infrastructure-as-code. Changes are manual and error-prone. | Operational Excellence |
| 8 | **No centralised monitoring or alerting** — incidents are detected reactively, increasing mean time to recovery (MTTR). | Operational Excellence |
| 9 | **Fixed capital expenditure** — on-premises hardware requires large upfront investment with no elasticity. | Cost Optimization |

---

## Task 2 — AWS Well-Architected Framework Evaluation

### WAF Assessment Table

| Pillar | Observation (Strength → Weakness) | Improvement Recommendation | Supporting AWS Service |
|---|---|---|---|
| **Operational Excellence** | **Strength:** The team has existing runbooks and a defined deployment process, providing a baseline for operational procedures. **Weakness:** All deployments are manual, with no infrastructure-as-code or automated pipelines, making operations slow, error-prone, and difficult to audit. | Adopt Infrastructure-as-Code (IaC) to define and provision all AWS resources in a repeatable, version-controlled manner. Implement a CI/CD pipeline for automated application deployments. Enable centralised logging and tracing for full operational visibility. | **AWS CloudFormation** (IaC); **AWS CodePipeline + CodeDeploy** (CI/CD); **Amazon CloudWatch** (monitoring & alerting); **AWS X-Ray** (distributed tracing) |
| **Security** | **Strength:** The organisation has an existing firewall at the network perimeter, demonstrating some awareness of network security controls. **Weakness:** Security groups are likely overly permissive (e.g., port 3306 open to 0.0.0.0/0), there is no encryption at rest or in transit, no identity federation, and no secrets management for database credentials. | Apply the principle of least privilege across all layers. Restrict security group rules so the database tier accepts connections only from the application tier's security group. Encrypt all data at rest and in transit. Rotate database credentials automatically using a secrets manager. Enable threat detection and audit logging. | **AWS IAM** (least-privilege access control); **Amazon VPC with Security Groups & NACLs** (network segmentation); **AWS KMS** (encryption key management); **AWS Secrets Manager** (credential rotation); **Amazon GuardDuty** (threat detection); **AWS CloudTrail** (audit logging) |
| **Reliability** | **Strength:** The application is functional and has been serving users on-premises, indicating that the core logic is proven and stable. **Weakness:** The architecture has a single point of failure at every layer — one web server, one database server, one AZ equivalent. There is no automated failover, no backup strategy, and no defined Recovery Time Objective (RTO) or Recovery Point Objective (RPO). | Deploy across multiple Availability Zones (Multi-AZ) to eliminate single points of failure. Use a managed relational database with automated failover and daily automated backups. Place web servers behind a load balancer with health checks. Define and test RTO/RPO targets. | **Elastic Load Balancing (ALB)** (traffic distribution & health checks); **Amazon RDS Multi-AZ** (automated database failover & backups); **Amazon Route 53** (DNS failover routing); **AWS Backup** (centralised backup management & policies) |
| **Performance Efficiency** | **Strength:** The existing architecture is simple, which makes it easy to reason about performance bottlenecks (the web server or the database). **Weakness:** The architecture cannot scale horizontally. During traffic spikes, the single web server becomes a bottleneck. There is no caching layer, no CDN, and no right-sizing strategy — the servers are provisioned for peak load and idle the rest of the time. | Introduce auto-scaling for the web tier so capacity matches actual demand. Add a caching layer in front of the database to reduce query load. Serve static assets through a CDN to reduce latency for end users globally. Right-size compute instances using AWS Compute Optimizer recommendations. | **Amazon EC2 Auto Scaling** (horizontal scaling); **Amazon ElastiCache for Redis** (in-memory caching); **Amazon CloudFront** (CDN for static assets and API acceleration); **AWS Compute Optimizer** (right-sizing recommendations) |
| **Cost Optimization** | **Strength:** Migrating to AWS immediately eliminates the large capital expenditure (CapEx) of on-premises hardware refreshes, replacing it with a predictable operational expenditure (OpEx) model. **Weakness:** Without a cost management strategy, cloud spend can grow uncontrolled. Idle or over-provisioned resources, lack of Reserved Instance commitments, and running resources 24/7 without scheduling can lead to overspending. | Implement cost visibility from day one using AWS Cost Explorer and billing alerts. Use Reserved Instances or Savings Plans for stable, predictable baseline workloads to reduce costs by up to 72% compared to On-Demand pricing. Enable auto-scaling so the web tier only runs what is needed. Tag all resources for cost allocation reporting. | **AWS Cost Explorer** (cost analysis & forecasting); **AWS Budgets** (spend alerts & thresholds); **EC2 Reserved Instances / Savings Plans** (commitment-based discounts); **AWS Trusted Advisor** (cost optimisation checks) |

---

## Task 3 — AWS Cloud Adoption Framework (CAF) Readiness Analysis

### Perspective 1: Business

The Business perspective ensures that cloud investments deliver measurable business value and that migration goals are aligned with broader organisational strategy. For this migration, the organisation's primary business drivers are improved application availability (reducing downtime risk), faster time-to-market for new features (enabled by CI/CD pipelines), and reduced total cost of ownership by eliminating on-premises hardware maintenance.

Currently, the organisation lacks a formal cloud business case with defined KPIs for the migration. Key gaps include the absence of a Cloud ROI model quantifying cost savings versus on-premises, and no Service Level Agreements (SLAs) formally defining availability targets for the migrated application.

**Key enablers required:**
- Develop a Cloud Business Case comparing 3-year TCO (on-premises vs. AWS).
- Define measurable success KPIs: target uptime (e.g., 99.9%), deployment frequency, and mean time to recovery (MTTR).
- Align migration milestones with business planning cycles and board reporting.
- Identify executive sponsors who champion cloud adoption across business units.

---

### Perspective 2: People

The People perspective addresses the human and cultural changes required for successful cloud adoption. Cloud migration is not only a technology change — it is an organisational change that requires new skills, new ways of working, and a shift from a reactive "keep the lights on" culture to a proactive "you build it, you run it" DevOps culture.

Currently, the organisation's IT team has strong on-premises infrastructure expertise but limited cloud-native skills. There is no dedicated cloud team, no AWS-certified personnel, and no formal cloud training programme. The team may feel uncertain or resistant to change, viewing cloud adoption as a threat to existing roles.

**Key enablers required:**
- Launch an AWS Skills Training Programme: enrol team members in AWS Cloud Practitioner and Solutions Architect Associate certifications.
- Establish a Cloud Centre of Excellence (CCoE) — a cross-functional team responsible for cloud standards, guardrails, and knowledge sharing.
- Communicate clearly that cloud adoption creates new, higher-value roles (cloud engineers, DevOps engineers) rather than eliminating jobs.
- Introduce DevOps practices gradually: begin with automated deployments (CI/CD) and progress to full infrastructure-as-code ownership.

---

### Perspective 3: Governance

The Governance perspective ensures that cloud usage is controlled, compliant, and accountable. Without governance guardrails, cloud environments rapidly accumulate unmanaged resources, shadow IT, and compliance violations. For this migration, governance is critical because the application likely handles user personal data subject to data protection regulations (e.g., Ghana's Data Protection Act, Act 843, or GDPR if international users are served).

Currently, there is no cloud governance policy, no tagging strategy, no resource lifecycle management process, and no formal risk management framework for cloud operations. The organisation also lacks defined approval workflows for provisioning new cloud resources.

**Key enablers required:**
- Implement AWS Organizations with Service Control Policies (SCPs) to enforce organisational guardrails (e.g., prevent resource creation outside approved regions).
- Define a mandatory tagging strategy (Owner, Environment, CostCenter, Project) applied to all resources at creation.
- Establish a Cloud Governance Board that reviews and approves new workloads, major architectural changes, and budget increases.
- Use AWS Config and AWS Security Hub to continuously audit compliance against defined policies and flag drift automatically.

---

### Perspective 4: Platform

The Platform perspective covers the design and implementation of the cloud infrastructure, including the landing zone, network topology, compute, storage, and managed services. A well-designed platform provides a stable, secure, and scalable foundation that all workloads can rely on.

Currently, the organisation has no AWS landing zone, no VPC design, and no understanding of AWS shared responsibility boundaries. The team is accustomed to managing physical hardware and may attempt to lift-and-shift the on-premises architecture directly without taking advantage of managed services.

**Key enablers required:**
- Design and deploy an AWS Landing Zone using AWS Control Tower to establish a multi-account structure with security, logging, and shared services accounts.
- Build a VPC architecture with public subnets (for the load balancer), private subnets (for EC2 web tier), and isolated subnets (for the RDS database tier) across at least two Availability Zones.
- Replace the self-managed on-premises database with Amazon RDS (managed relational database), eliminating the need to manage patches, backups, and failover manually.
- Adopt infrastructure-as-code (AWS CloudFormation or Terraform) as the only approved method for provisioning resources — no manual console deployments in production.

---

### Perspective 5: Security

The Security perspective ensures that the cloud environment is protected across all layers — identity, data, network, and infrastructure — and that the organisation meets its compliance obligations. Cloud security operates on the AWS Shared Responsibility Model: AWS secures the cloud infrastructure; the customer is responsible for securing what they build on top of it.

Currently, the organisation's security posture is limited to perimeter-based on-premises firewall rules. There is no identity federation, no secrets management, no data encryption strategy, and no security incident response plan for cloud environments. Security is treated as a perimeter concern rather than a defence-in-depth discipline.

**Key enablers required:**
- Adopt a Zero-Trust security model: every request must be authenticated and authorised, even inside the VPC.
- Enforce Multi-Factor Authentication (MFA) for all AWS console and API access via IAM Identity Center.
- Enable AWS CloudTrail in all regions and all accounts from day one — every API call must be logged and retained for at least 12 months.
- Implement automated threat detection with Amazon GuardDuty and vulnerability assessment with Amazon Inspector.
- Classify all data handled by the application and apply appropriate controls: encrypt sensitive data at rest with AWS KMS and in transit with TLS 1.3.

---

### Perspective 6: Operations

The Operations perspective ensures that the cloud environment can be monitored, managed, and continuously improved after migration. Effective cloud operations replace reactive firefighting with proactive, automated responses to events, enabling the team to maintain and improve the application's availability and performance with confidence.

Currently, operations are entirely reactive. Monitoring is limited to OS-level metrics, there are no automated alerts, incident response is manual, and there is no defined change management process. The team has no experience with cloud-native operations tools.

**Key enablers required:**
- Deploy a full observability stack from day one: Amazon CloudWatch for metrics and logs, AWS X-Ray for distributed tracing, and a centralised dashboard (CloudWatch Dashboard or AWS Managed Grafana).
- Define and automate runbooks using AWS Systems Manager Automation — for example, automated responses to high CPU alerts (trigger auto-scaling) or failed health checks (trigger instance replacement).
- Establish an incident management process with defined severity levels, escalation paths, and post-incident review (PIR) templates.
- Implement a formal change management process: all changes go through a CI/CD pipeline; no manual changes in production. Use AWS Config to detect and alert on configuration drift.

---

## Task 4 — Improved Architecture Design

### 4.1 Architecture Description

The revised architecture implements a highly available, secure, scalable, and cost-optimised two-tier web application on AWS, addressing all five WAF pillars.

---

#### Network Foundation (VPC Design)

```
Region: af-south-1 (Cape Town) — Primary
├── VPC: 10.0.0.0/16
│   ├── Availability Zone A (af-south-1a)
│   │   ├── Public Subnet A:  10.0.1.0/24  → ALB Node, NAT Gateway A
│   │   ├── Private Subnet A: 10.0.2.0/24  → EC2 Web Tier (Auto Scaling)
│   │   └── DB Subnet A:      10.0.3.0/24  → RDS Primary Instance
│   │
│   └── Availability Zone B (af-south-1b)
│       ├── Public Subnet B:  10.0.4.0/24  → ALB Node, NAT Gateway B
│       ├── Private Subnet B: 10.0.5.0/24  → EC2 Web Tier (Auto Scaling)
│       └── DB Subnet B:      10.0.6.0/24  → RDS Standby Instance (Multi-AZ)
│
├── Internet Gateway → attached to VPC
└── VPC Flow Logs → CloudWatch Logs (network traffic auditing)
```

---

#### Traffic Flow (Request Path)

```
Internet Users
      │
      ▼
[Amazon CloudFront]          ← CDN: caches static assets, reduces latency globally
      │
      ▼
[Application Load Balancer]  ← Public subnets; HTTPS only (TLS termination); health checks
      │
      ├──────────────────────────────────────┐
      ▼                                      ▼
[EC2 Web Tier — AZ-A]              [EC2 Web Tier — AZ-B]
 Private Subnet A                   Private Subnet B
 (Auto Scaling Group — min 2, max 8 instances)
      │
      ▼
[Amazon ElastiCache for Redis]  ← Caching layer; reduces database query load by ~60%
      │
      ▼
[Amazon RDS (MySQL) — Multi-AZ]
 Primary in DB Subnet A  ←→  Standby in DB Subnet B
 Automated failover < 60 seconds
 Automated daily snapshots (35-day retention)
 Encrypted at rest (AWS KMS)
```

---

#### Security Architecture (Defence in Depth)

```
Layer 1 — Edge:       AWS WAF on CloudFront (block OWASP Top 10 threats)
Layer 2 — Network:    ALB Security Group — inbound 443 only from 0.0.0.0/0
Layer 3 — Compute:    EC2 Security Group — inbound 443 from ALB SG only; no public IP
Layer 4 — Database:   RDS Security Group — inbound 3306 from EC2 SG only; no public access
Layer 5 — Identity:   IAM roles for EC2 (no hardcoded credentials); Secrets Manager for DB password
Layer 6 — Data:       KMS encryption at rest (RDS, EBS, S3); TLS 1.3 in transit
Layer 7 — Audit:      CloudTrail (all API calls), CloudWatch Logs, VPC Flow Logs, GuardDuty
```

---

#### Operational & Cost Services

```
Monitoring:     Amazon CloudWatch — metrics, logs, alarms, dashboards
Tracing:        AWS X-Ray — end-to-end request tracing
Automation:     AWS Systems Manager — automated runbooks, patch management
IaC:            AWS CloudFormation — all infrastructure defined as code
CI/CD:          AWS CodePipeline + CodeDeploy — automated application deployments
Backup:         AWS Backup — centralised backup policy (daily snapshots, 35-day retention)
Cost Control:   AWS Budgets (monthly alert at 80% of budget threshold)
                AWS Cost Explorer (spend analysis and forecasting)
                EC2 Reserved Instances for baseline capacity (1-year commitment)
                Auto Scaling for variable demand (On-Demand pricing only for burst)
```

---

### 4.2 WAF Pillar Mapping — How the New Design Addresses Each Pillar

| WAF Pillar | How the New Architecture Addresses It |
|---|---|
| **Operational Excellence** | CloudFormation (IaC), CodePipeline (CI/CD), CloudWatch dashboards, X-Ray tracing, Systems Manager runbooks eliminate manual operations. |
| **Security** | Defence-in-depth with AWS WAF, layered security groups, IAM roles, Secrets Manager, KMS encryption, CloudTrail, and GuardDuty. |
| **Reliability** | Multi-AZ deployment, ALB with health checks, RDS Multi-AZ with automated failover, Auto Scaling Group, and AWS Backup eliminate single points of failure. |
| **Performance Efficiency** | CloudFront CDN, ElastiCache Redis, EC2 Auto Scaling, and ALB ensure the application scales with demand and serves users with low latency. |
| **Cost Optimization** | Auto Scaling (pay only for what is used), Reserved Instances for baseline, AWS Budgets and Cost Explorer for visibility, S3 for static asset storage (cheaper than EC2 disk). |

---

### 4.3 AWS Well-Architected Tool Validation

The architecture was validated against the AWS Well-Architected Tool in the AWS Management Console by reviewing the questionnaire for the "Serverless and Workload" lens. All five pillar questions were answered based on the revised design above. High-risk issues (HRIs) identified in the original design — specifically around Multi-AZ deployment, backup strategy, and security group configuration — are fully resolved in the improved architecture.

---

## Reflection

This lab reinforced that cloud migration is never purely a technical exercise — it is an organisational transformation. Applying the Well-Architected Framework made me think systematically across five dimensions rather than defaulting to a simple lift-and-shift approach. Each pillar revealed a different category of risk in the original on-premises design: reliability gaps exposed by the single-AZ deployment, security weaknesses from overly permissive rules, and cost inefficiency from static provisioning.

The Cloud Adoption Framework added depth by highlighting that technology alone cannot drive successful cloud adoption. Without addressing the People and Governance perspectives, even a technically excellent architecture can fail due to skill gaps, resistance to change, or uncontrolled cloud sprawl. The most important insight from this lab is that the five WAF pillars and six CAF perspectives are complementary — the WAF tells you *what* to build, and the CAF tells you *how* to prepare your organisation to build and operate it sustainably.
