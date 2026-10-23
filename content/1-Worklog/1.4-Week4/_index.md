---
title: "Week 4 Worklog"
date: 2026-10-05
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Finalize the comprehensive **Project Proposal** report deliverable for the **3-Tier Resilient Web Architecture** capstone project.
* Author an in-depth system architecture blueprint highlighting data paths and multi-Availability-Zone security boundaries.
* Model and forecast monthly operational expenditures using the official **AWS Pricing Calculator**.
* Perform risk matrix evaluations and formulate backup and disaster recovery (DR) strategies.
* Commence provisioning foundational cloud infrastructure components on AWS in accordance with the ratified architectural specifications.

---

### Weekly Task Summary:

| Day | Primary Tasks | Start Date | Completion Date | Reference Material |
| --- | ------------- | ---------- | --------------- | ------------------ |
| **2** | **Proposal Authoring & Problem Statement:** Articulate the business problem, target availability metrics (99.9% uptime SLA), and non-functional requirements. | 05/10/2026 | 05/10/2026 | Internal FCAJ Proposal Guide |
| **3** | **Detailed Architecture Diagramming:** Draft the 3-tier model (Ingress, Compute, Data Tiers) and establish Security Group chaining policies. | 06/10/2026 | 06/10/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) |
| **4** | **Budgeting via AWS Pricing Calculator:** Break down granular costs across services (ALB, EC2, RDS Multi-AZ, S3, CloudFront). | 07/10/2026 | 07/10/2026 | [AWS Pricing Calculator](https://calculator.aws/) |
| **5** | **Risk Assessment & DR Planning:** Analyze risk exposure (RTO/RPO), finalize the formal deliverable, and conduct peer team review. | 08/10/2026 | 08/10/2026 | [AWS Well-Architected Reliability](https://docs.aws.amazon.com/wellarchitected/) |
| **6** | **Foundational Infrastructure Provisioning:** Stand up the Production VPC (4 Subnets across 2 AZs), attach an Internet Gateway, and create the S3 assets bucket. | 09/10/2026 | 09/10/2026 | [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/) |

---

### Detailed Technical Log:

#### Monday (05/10/2026): Proposal Documentation & Business Problem Definition

1. **Context and Problem Statement:**
   - Small-to-medium enterprise applications and e-commerce platforms often encounter crippling server outages when traffic surges during flash sales or marketing campaigns.
   - Traditional monolithic single-server deployments suffer from Single Points of Failure (SPOF): When an underlying physical host or datacenter network degrades, business operations halt.
2. **Defining Service Level Indicators & Targets (SLO / SLA):**
   - **Availability:** Target **99.9% uptime** (under 8.76 hours of permissible unscheduled downtime annually).
   - **Elastic Scalability:** Automated horizontal scaling from 2 up to 6 compute instances within 3 minutes when average CPU exceeds 60%.
   - **Disaster Recovery Targets:**
     - *Recovery Point Objective (RPO):* Under 5 minutes (transactional integrity preserved via RDS automated multi-AZ replication).
     - *Recovery Time Objective (RTO):* Under 2 minutes (automatic health-check-driven failover without human intervention).

---

#### Tuesday (06/10/2026): Detailed Architectural System Blueprinting

1. **3-Tier Architectural Hierarchy:**
   - **Tier 1 - Ingress & Edge Tier:**
     - *Amazon Route 53:* Manages authoritative DNS resolution and intelligent health-checked routing.
     - *Amazon CloudFront:* Global CDN caching static assets (HTML, CSS, JS, images) at worldwide edge locations, absorbing up to 70% of origin traffic.
     - *Application Load Balancer (ALB):* Placed in Public Subnets across 2 Availability Zones, distributing HTTP/HTTPS traffic to application servers.
   - **Tier 2 - Application & Compute Tier:**
     - *Auto Scaling Group (ASG):* Manages fleets of Amazon EC2 nodes situated inside isolated **Private App Subnets** (devoid of public IPv4 addresses).
     - *NAT Gateway:* Provides outbound-only Internet connectivity enabling private compute nodes to download security patches and updates.
   - **Tier 3 - Database & Storage Tier:**
     - *Amazon RDS for MySQL (Multi-AZ Deployment):* Primary DB host residing in Private Subnet AZ-1 with synchronous replication to an active Standby instance in AZ-2.
     - *Amazon S3 Bucket:* Durable object storage for application media assets backed by 99.999999999% (11 9s) data durability.

2. **Security Group Chaining Matrix:**

| Security Group | Permitted Inbound Traffic | Source Rule | Security Objective |
| -------------- | ------------------------- | ----------- | ------------------ |
| `ALB-SG` | Port `80` (HTTP) / Port `443` (HTTPS) | `0.0.0.0/0` (Internet) | Ingress entrypoint for public client traffic |
| `Web-App-SG` | Port `80` (HTTP) | `ALB-SG` ID only | Eliminates unauthorized direct access bypassing the load balancer |
| `Database-SG` | Port `3306` (MySQL) | `Web-App-SG` ID only | Restricts database access strictly to authenticated compute instances |

---

#### Wednesday (07/10/2026): Quantitative Cost Modeling via AWS Pricing Calculator

1. **Detailed Monthly Expense Breakdown:**
   - Simulated realistic production workloads within the Singapore Region (`ap-southeast-1`):

| AWS Service | Configuration Specifications | Estimated Cost (USD/month) |
| ----------- | ---------------------------- | -------------------------- |
| **Amazon EC2** | 2x `t3.micro` instances (2 vCPU, 1GB RAM) running 24/7 baseline | ~15.18 USD |
| **Amazon EBS** | 2x `gp3` root volumes at 20GB each (40GB aggregate) | ~3.20 USD |
| **Application Load Balancer** | 1 ALB processing ~10GB throughput and 1 LCU | ~16.50 USD |
| **Amazon RDS (Multi-AZ)** | `db.t4g.micro` (2 vCPU, 1GB RAM) with 20GB gp3 storage | ~26.40 USD |
| **Amazon S3 Standard** | 10GB object storage, 50,000 GET requests, 5,000 PUT requests | ~0.45 USD |
| **Amazon CloudFront** | 50GB Data Transfer Out to Internet | ~6.00 USD |
| **Data Transfer & Route 53** | 1 Public Hosted Zone + inter-AZ internal transfer | ~1.50 USD |
| **Total Monthly Baseline** | *(Prior to applying Free Tier promotions/credits)* | **~69.23 USD / month** |

2. **Development Sandbox Strategy:**
   - Leveraging the **$200 Promotional Credit** earned during previous onboarding milestones allows the team to sustain sandbox operations for over 2.5 months with zero out-of-pocket costs.
   - For day-to-day development, deploying RDS in Single-AZ mode and pausing ALBs during inactive periods reduces operating spend by over 60%.

---

#### Thursday (08/10/2026): Risk Assessment & Disaster Recovery Strategy

1. **Risk Evaluation Matrix:**

| Identified Threat | Impact Level | Likelihood | Mitigation Strategy |
| ----------------- | ------------ | ---------- | ------------------- |
| **Availability Zone (AZ) Outage** | Critical | Low | Multi-AZ architecture: ALB and ASG redirect traffic to surviving AZ; RDS executes automated failover within 60 seconds. |
| **Traffic Surges (Resource Starvation)** | High | Medium | ASG dynamic scale-out triggers when CPU > 60%; CloudFront CDN caches and serves static traffic from edge nodes. |
| **Database Corruption / Data Loss** | Severe | Very Low | Automated RDS point-in-time snapshots retained for 7 days; S3 object versioning enabled against accidental deletion. |
| **Unexpected Cost Overrun** | Moderate | Low | Configured **AWS Budgets** alerts dispatching notifications at 50%, 80%, and 100% project thresholds. |

---

#### Friday (09/10/2026): Foundational Infrastructure Provisioning

1. **Automated Production VPC Creation via AWS CLI:**
   ```bash
   # 1. Create project VPC with a 10.10.0.0/16 CIDR block
   VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/16 \
       --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Duy-Project-Production-VPC}]' \
       --query 'Vpc.VpcId' --output text)
   echo "Created Production VPC: $VPC_ID"

   # 2. Enable DNS Hostnames inside the VPC
   aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames "{\"Value\":true}"

   # 3. Create Internet Gateway and attach to VPC
   IGW_ID=$(aws ec2 create-internet-gateway \
       --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Duy-Project-IGW}]' \
       --query 'InternetGateway.InternetGatewayId' --output text)
   aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID
   ```

2. **Network Subnet Allocation across 2 AZs (`ap-southeast-1a` & `ap-southeast-1b`):**
   - **Public Subnet AZ1:** `10.10.1.0/24` (ALB Node A)
   - **Public Subnet AZ2:** `10.10.2.0/24` (ALB Node B)
   - **Private Subnet AZ1:** `10.10.11.0/24` (Compute Node A & RDS Primary)
   - **Private Subnet AZ2:** `10.10.12.0/24` (Compute Node B & RDS Standby)

3. **Secure S3 Storage Provisioning:**
   ```bash
   # Provision the application asset repository bucket
   aws s3 mb s3://duy-project-app-assets-2026 --region ap-southeast-1

   # Enable object versioning to safeguard against accidental overwrites
   aws s3api put-bucket-versioning \
       --bucket duy-project-app-assets-2026 \
       --versioning-configuration Status=Enabled
   ```

---

### Week 4 Achievements:

* **Proposal Document Completed:** Successfully authored the comprehensive Project Proposal deliverable detailing requirements, technical design, roadmap, and pricing.
* **Professional Architecture Blueprint:** Produced an industry-grade diagram illustrating multi-tier boundary separation, multi-AZ high availability, and Security Group chaining.
* **Rigorous Cost Modeling:** Mastered the **AWS Pricing Calculator**, accurately forecasting monthly operating expenses at ~$69 USD/month while demonstrating optimal utilization of promotional credits.
* **Foundational Cloud Network Provisioned:** Successfully stood up the production VPC spanning 4 subnets across two AZs with enabled DNS resolution and an attached Internet Gateway.
* **Phase Readiness:** Seamlessly transitioned from planning to execution, establishing a solid baseline for service provisioning in Week 5.

---

### Challenges Encountered & Solutions:

| Problem Encountered | Root Cause | Solution Implemented |
| ------------------- | ---------- | -------------------- |
| **Multi-AZ RDS pricing exceeded initial estimates in Calculator** | Multi-AZ doubles compute hours and storage allocation to maintain the synchronous standby node. | Adjusted development strategy: Use Single-AZ RDS (`db.t4g.micro`) during initial integration weeks; only activate Multi-AZ replication during final failover validation drills. |
| **CIDR address overlap when planning database subnets** | Subnet CIDR masks were initially too small or collided with legacy lab VPCs. | Standardized on `/24` subnet masks (251 usable host IPs per subnet) and created a dedicated address plan strictly separating public and private domains. |
| **CORS errors when CloudFront interacts with S3 assets** | S3 buckets restrict Cross-Origin Resource Sharing (CORS) by default for external domain origins. | Attached a tailored JSON CORS policy to the S3 bucket allowing `GET` and `HEAD` methods from the CloudFront distribution domain. |

---

### Lessons Learned & Best Practices:

1. **Architecture Planning Precedes Implementation:**
   * Spending adequate time designing network topology and modeling pricing mitigates costly tear-down and rebuild cycles later.
2. **Balancing Cost vs Availability:**
   * High availability requires deliberate architectural tradeoffs. Knowing when to run cost-effective dev configurations versus high-resilience production environments is an essential cloud engineering skill.
3. **Infrastructure as Code (IaC) Readiness:**
   * Scripting the VPC creation via AWS CLI builds the exact foundation needed for future automation with AWS CloudFormation or Terraform.

---

### Plan for Next Week (Week 5):

* Deploy the Compute Tier inside Private Subnets: Configure outbound Internet connectivity via **NAT Gateway** or **VPC Endpoints**.
* Launch the **Amazon RDS MySQL** instance inside the isolated private database subnet with KMS encryption at rest.
* Test internal application-to-database communication within the private network boundary.
