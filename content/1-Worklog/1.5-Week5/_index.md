---
title: "Week 5 Worklog"
date: 2026-10-12
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Implement secure one-way outbound Internet egress for private workloads utilizing **AWS NAT Gateways**.
* Optimize internal data traffic paths and reduce bandwidth expenses with **Amazon S3 Gateway Endpoints (VPC Endpoints)**.
* Provision and secure a production-ready **Amazon RDS MySQL** relational database inside private subnets featuring **AWS KMS** encryption at rest.
* Validate end-to-end private network connectivity between private EC2 compute instances and the internal database endpoint.
* Conduct a comprehensive network security audit and establish strict cost-governance routines for fee-accruing networking services.

---

### Weekly Task Summary:

| Day | Primary Tasks | Start Date | Completion Date | Reference Material |
| --- | ------------- | ---------- | --------------- | ------------------ |
| **2** | **NAT Gateway & Private Egress Routing:** Provision a NAT Gateway in the Public Subnet and update Private Route Tables for secure outbound patch access. | 12/10/2026 | 12/10/2026 | [VPC NAT Gateway Guide](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) |
| **3** | **S3 Gateway Endpoint Optimization:** Configure a VPC Gateway Endpoint to Amazon S3, routing traffic through the AWS private network backbone without NAT charges. | 13/10/2026 | 13/10/2026 | [AWS PrivateLink & Gateway Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/) |
| **4** | **Database Tier Provisioning (Amazon RDS):** Create a multi-AZ DB Subnet Group and deploy a MySQL instance in private subnets with KMS storage encryption. | 14/10/2026 | 14/10/2026 | [Amazon RDS User Guide](https://docs.aws.amazon.com/rds/) |
| **5** | **Application-to-Database Integration Testing:** Install MySQL Client on private EC2, connect via internal DNS, initialize database schemas and mock records. | 15/10/2026 | 15/10/2026 | [Connecting to an RDS Instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html) |
| **6** | **Security Auditing & Cost Decommissioning:** Perform external port penetration tests, evaluate internal query latencies, and tear down NAT Gateways and Elastic IPs. | 16/10/2026 | 16/10/2026 | [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/) |

---

### Detailed Technical Log:

#### Monday (12/10/2026): NAT Gateway Deployment & Private Subnet Egress

1. **Provisioning the NAT Gateway:**
   - Allocated a dedicated static public **Elastic IP (EIP)**.
   - Deployed the NAT Gateway inside **Public Subnet AZ1** (`10.10.1.0/24`):
     - Name: `Duy-Project-NATGW-AZ1`
     - Subnet: `Duy-Public-Subnet-AZ1`
     - Elastic IP allocation ID: `eipalloc-xxxxxx`
   - Monitored status transition from `Pending` $\rightarrow$ `Available`.

2. **Configuring the Private Route Table:**
   - Created a dedicated custom route table named `Duy-Private-RT`.
   - Added outbound default route:
     - **Destination:** `0.0.0.0/0` (all outbound Internet traffic)
     - **Target:** NAT Gateway (`nat-xxxxxx`)
   - Associated both Private Subnets (`10.10.11.0/24` and `10.10.12.0/24`) with this route table.

3. **Verifying Outbound-Only Connectivity:**
   - Launched a test EC2 instance strictly inside the Private Subnet (without assigning public IPv4).
   - Accessed the instance via Bastion Host or AWS Systems Manager Session Manager.
   - Verified one-way network egress:
     ```bash
     # Verify external DNS resolution and package manager downloads
     sudo dnf update -y
     # Verify outbound public IP matches the NAT Gateway Elastic IP
     curl https://checkip.amazonaws.com
     ```
   - Validation confirmed: The internal host downloads packages freely while remaining completely shielded from unsolicited inbound Internet probes.

---

#### Tuesday (13/10/2026): Architectural Optimization with S3 Gateway Endpoints

1. **Architectural Motivation:**
   - Directing high-volume object transfers between private EC2 instances and Amazon S3 through a NAT Gateway incurs data processing fees (~$0.045/GB) and introduces network hops.
   - Deploying a complimentary **Gateway VPC Endpoint** routes S3 traffic directly over the high-speed **AWS Internal Network Backbone**.

2. **Automating S3 Gateway Endpoint Creation via AWS CLI:**
   ```bash
   # Provision S3 Gateway Endpoint and associate directly with Private Route Table
   ENDPOINT_ID=$(aws ec2 create-vpc-endpoint \
       --vpc-id $VPC_ID \
       --service-name com.amazonaws.ap-southeast-1.s3 \
       --route-table-ids $PRIVATE_RT_ID \
       --tag-specifications 'ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=Duy-Project-S3-Endpoint}]' \
       --query 'VpcEndpoint.VpcEndpointId' --output text)
   echo "Created S3 Gateway Endpoint: $ENDPOINT_ID"
   ```

3. **Routing Verification:**
   - Inspected `Duy-Private-RT`; a new managed prefix list route was automatically injected:
     - **Destination:** `pl-xxxx (com.amazonaws.ap-southeast-1.s3)`
     - **Target:** `vpce-xxxxxx`
   - Tested high-throughput object upload from private EC2 to S3:
     ```bash
     aws s3 cp large-data-file.tar.gz s3://duy-project-app-assets-2026/
     ```
   - Observed wire-speed transfer performance without incurring any NAT Gateway data-processing fees.

---

#### Wednesday (14/10/2026): Deploying the Database Tier (Amazon RDS MySQL)

1. **DB Subnet Group Configuration:**
   - Name: `duy-project-rds-subnet-group`
   - VPC: `Duy-Project-Production-VPC`
   - Attached Subnets: Selected 2 Private Subnets spanning distinct AZs (`10.10.11.0/24` in `ap-southeast-1a` and `10.10.12.0/24` in `ap-southeast-1b`), ensuring full multi-AZ readiness.

2. **Hardening the RDS MySQL Instance:**
   - **Engine:** MySQL Community 8.0.
   - **Template:** Dev/Test tier.
   - **Identifiers:**
     - DB instance identifier: `duy-production-mysql`
     - Master username: `admin_duy`
     - Master password: `[Securely managed in AWS Secrets Manager]`
   - **Instance Specifications:**
     - Instance class: `db.t4g.micro` (Graviton2 ARM, 2 vCPU, 1 GB RAM).
     - Storage: `20 GiB`, Storage type: `gp3`.
   - **Network Isolation:**
     - VPC: `Duy-Project-Production-VPC`
     - DB Subnet Group: `duy-project-rds-subnet-group`
     - **Publicly Accessible:** `No` (Strictly internal).
     - Security Group: Bound to `Database-SG` (Port 3306 restricted solely to `Web-App-SG`).
   - **Encryption:**
     - Storage encryption enabled via AWS Key Management Service (**AWS KMS** key `aws/rds`).

---

#### Thursday (15/10/2026): Private Tier Integration & Data Seeding

1. **Installing Database Client on Private Compute Node:**
   - Connected to the private application EC2 instance.
   - Installed the native client binary:
     ```bash
     sudo dnf install -y mariadb105
     ```

2. **Establishing Connection via Private DNS Endpoint:**
   ```bash
   RDS_ENDPOINT="duy-production-mysql.xxxxxx.ap-southeast-1.rds.amazonaws.com"
   mysql -h $RDS_ENDPOINT -P 3306 -u admin_duy -p
   ```

3. **Database Schema Initialization & Data Population:**
   ```sql
   CREATE DATABASE ecom_store_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   USE ecom_store_db;

   CREATE TABLE products (
       id INT AUTO_INCREMENT PRIMARY KEY,
       sku VARCHAR(50) NOT NULL UNIQUE,
       name VARCHAR(255) NOT NULL,
       price DECIMAL(10, 2) NOT NULL,
       stock_quantity INT DEFAULT 0,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );

   INSERT INTO products (sku, name, price, stock_quantity) VALUES
   ('AWS-SHIRT-01', 'First Cloud AI Journey T-Shirt', 19.99, 100),
   ('AWS-HOODIE-02', 'Cloud Engineer Hoodie', 49.99, 50),
   ('AWS-MUG-03', 'Architect Ceramic Mug', 12.50, 200);

   SELECT * FROM products;
   ```
   *Observations:* Query operations completed instantaneously with local area network (LAN) round-trip latencies below 1 millisecond.

---

#### Friday (16/10/2026): Network Security Audit & Cost Hygiene

1. **Security Penetration Audit:**
   - Attempted external port probing from an unauthenticated home workstation:
     ```bash
     telnet duy-production-mysql.xxxxxx.ap-southeast-1.rds.amazonaws.com 3306
     ```
   - Connection attempts timed out after 30 seconds with zero acknowledgement.
   - Confirmed complete perimeter stealth: Port 3306 is utterly unreachable from the public Internet.

2. **Disciplined Decommissioning Routine:**
   - **NAT Gateway Teardown:** Avoided the perpetual idle cost of `$0.045/hour` (~$32/month):
     ```bash
     # Delete NAT Gateway
     aws ec2 delete-nat-gateway --nat-gateway-id $NAT_GW_ID

     # Release unattached Elastic IP to avoid idle holding penalties ($0.005/hr)
     aws ec2 release-address --allocation-id $EIP_ALLOC_ID
     ```
   - Stopped the RDS database instance during non-laboratory hours to preserve promotional credits.

---

### Week 5 Achievements:

* **Production-Grade Private Networking:** Successfully engineered one-way Internet access for private tier instances utilizing NAT Gateways and isolated route tables.
* **Cost & Route Optimization via Endpoints:** Configured Amazon S3 Gateway Endpoints, eliminating NAT data transfer costs while improving private throughput over the AWS backbone.
* **Impenetrable Database Isolation:** Deployed Amazon RDS MySQL within private subnet groups with KMS storage encryption, completely barring unauthorized public access.
* **Verified Internal Tier Integration:** Connected application compute instances to database endpoints over private DNS with sub-millisecond query latency.
* **Proactive Cost Discipline:** Decommissioned billable NAT Gateways and released unattached Elastic IPs immediately following validation.

---

### Challenges Encountered & Solutions:

| Problem Encountered | Root Cause | Solution Implemented |
| ------------------- | ---------- | -------------------- |
| **Private instances unable to reach Internet after NAT Gateway creation** | The Private Route Table was missing the default `0.0.0.0/0` route directing outbound traffic to the newly created NAT Gateway. | Edited the Private Route Table (`Duy-Private-RT`), added destination `0.0.0.0/0` targeting the NAT Gateway ID, and saved changes. |
| **Access Denied when private EC2 executed S3 commands via Endpoint** | The EC2 instance lacked an attached **IAM Instance Profile** granting programmatic S3 permissions. | Created an IAM Role with `AmazonS3FullAccess` and associated it with the instance via **Actions $\rightarrow$ Security $\rightarrow$ Modify IAM role**. |
| **VPC deletion blocked due to `DependencyViolation` error** | NAT Gateway was still transitioning through `deleting` state, or active RDS Network Interfaces (ENIs) remained bound. | Allowed the NAT Gateway to transition fully to `deleted`, dismantled the DB Subnet Group, and verified ENI detachment before deleting the VPC. |

---

### Lessons Learned & Best Practices:

1. **Defense-in-Depth for Data Stores:**
   * Protecting relational databases demands 3 distinct defensive layers: (1) Placement in Private Subnets without Public IPs, (2) Security Groups restricted solely to app tier callers, (3) Data encryption at rest with AWS KMS.
2. **Beware Hidden Network Charges:**
   * Idle NAT Gateways and unattached Elastic IPs represent the most common sources of unexpected cloud spend. Always release Elastic IPs immediately upon NAT deletion.
3. **Architectural Value of VPC Endpoints:**
   * Gateway and Interface VPC Endpoints enhance both security (keeping data entirely on AWS wires) and cost efficiency (avoiding NAT data fees).

---

### Plan for Next Week (Week 6):

* Orchestrate the complete 3-Tier Integration: Application Load Balancer (Tier 1) $\rightarrow$ Auto Scaling EC2 (Tier 2) $\rightarrow$ Amazon RDS MySQL (Tier 3).
* Deploy a functional dynamic web application demo reading live data from RDS and rendering assets from S3.
* Execute full-stack end-to-end resilience and failover testing.
