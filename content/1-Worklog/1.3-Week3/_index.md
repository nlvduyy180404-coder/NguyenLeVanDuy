---
title: "Week 3 Worklog"
date: 2026-09-28
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

* Collaborate with teammates and consult with Mentors to align on problem scope, requirements, and architecture for the **Project Proposal**.
* Master application-layer traffic distribution with **Application Load Balancer (ALB)**, diving deep into Target Groups and Health Checks.
* Implement dynamic fleet elasticity using **Auto Scaling Groups (ASG)** and standardized **Launch Templates**.
* Validate fault tolerance and self-healing capabilities across a **Multi-AZ** (Multiple Availability Zones) architecture.
* Research global content delivery optimization using **Amazon CloudFront (CDN)** and domain routing with **Amazon Route 53**.

---

### Weekly Task Summary:

| Day | Primary Tasks | Start Date | Completion Date | Reference Material |
| --- | ------------- | ---------- | --------------- | ------------------ |
| **2** | **Project Proposal Ideation & Team Sync:** Meet with Mentors and peer interns to analyze requirements for a 3-Tier Resilient Web Architecture. | 28/09/2026 | 28/09/2026 | Internal FCAJ Proposal Guide |
| **3** | **Application Load Balancer (ALB) Setup:** Create Target Groups, configure HTTP Health Checks, and provision an Internet-facing ALB across 2 AZs. | 29/09/2026 | 29/09/2026 | [AWS ELB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/) |
| **4** | **Auto Scaling Group (ASG) Deployment:** Author a standardized Launch Template and define Target Tracking Scaling Policies based on CPU utilization. | 30/09/2026 | 30/09/2026 | [AWS Auto Scaling Guide](https://docs.aws.amazon.com/autoscaling/) |
| **5** | **Multi-AZ Fault-tolerance Testing & CDN/DNS:** Conduct CPU stress tests and simulate AZ outages to verify self-healing; study CloudFront & Route 53. | 01/10/2026 | 01/10/2026 | [Amazon CloudFront](https://docs.aws.amazon.com/cloudfront/) |
| **6** | **Lab Synthesis & Proposal Framework Finalization:** Finalize architecture diagrams, outline the Proposal report, and safely tear down lab resources. | 02/10/2026 | 02/10/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) |

---

### Detailed Technical Log:

#### Monday (28/09/2026): Team Alignment & Proposal Project Definition

1. **Team Coordination & Mentor Consultation:**
   - Participated in an extensive sync meeting with assigned Mentors and teammates to select the semester-long capstone topic (**Project Proposal**).
   - Reviewed guidance on adhering to the **AWS Well-Architected Framework**:
     - *Reliability & High Availability:* Eliminating Single Points of Failure (SPOF) across every layer.
     - *Elastic Scalability:* Ensuring server fleet size dynamically adjusts to unpredictable traffic spikes.
     - *Defense in Depth:* Maintaining strict logical boundary isolation between public web ingress and private database stores.
2. **Architecture Blueprint Selection:**
   - Agreed to architect a **3-Tier Highly Available & Fault-Tolerant Web Architecture**:
     - **Tier 1 (Presentation / Ingress):** Amazon Route 53 + Amazon CloudFront + Application Load Balancer.
     - **Tier 2 (Application / Compute):** Auto Scaling Group running EC2 Web/App Servers across multiple Availability Zones.
     - **Tier 3 (Database / Persistence):** Amazon RDS Multi-AZ relational store combined with Amazon S3 for media assets.

---

#### Tuesday (29/09/2026): Application Load Balancer (ALB) Configuration

1. **Target Group Provisioning:**
   - Target type: `Instances`.
   - Protocol: `HTTP`, Port: `80`, VPC: `Duy-Custom-VPC`.
   - Configured **Health checks**:
     - Protocol: `HTTP`, Path: `/index.html`
     - Healthy threshold: `3` consecutive successes.
     - Unhealthy threshold: `2` consecutive failures.
     - Timeout: `5` seconds, Interval: `15` seconds.

2. **Deploying the Application Load Balancer:**
   - Name: `Duy-Public-ALB`.
   - Scheme: **Internet-facing**.
   - IP address type: `IPv4`.
   - Network mapping: Selected `Duy-Custom-VPC` spanning **2 distinct Availability Zones**:
     - `ap-southeast-1a` $\rightarrow$ associated with `Duy-Public-Subnet-AZ1` (`10.0.1.0/24`)
     - `ap-southeast-1b` $\rightarrow$ associated with `Duy-Public-Subnet-AZ2` (`10.0.3.0/24`)
   - Security Group: Allowed inbound port `80` (HTTP) from `0.0.0.0/0`.
   - Listeners & routing: Forward inbound HTTP port 80 requests to `Duy-Target-Group`.

3. **Verifying Traffic Distribution:**
   - Registered 2 EC2 instances into the Target Group.
   - Observed transition from `Initial` $\rightarrow$ `Healthy`.
   - Visited the ALB DNS endpoint: `http://duy-public-alb-12345678.ap-southeast-1.elb.amazonaws.com`.
   - Refreshing the browser confirmed alternating responses between backend instances, proving functional Round-Robin traffic distribution.

---

#### Wednesday (30/09/2026): Dynamic Fleet Elasticity with Auto Scaling Group (ASG)

1. **Standardized Launch Template (`Duy-Web-LT`):**
   - AMI: `Amazon Linux 2023`.
   - Instance type: `t2.micro` (Free tier eligible).
   - Security Group: Restricted inbound port 80 exclusively to the **ALB's Security Group ID** (preventing direct bypass of the load balancer).
   - Dynamic **User Data bootstrap script** leveraging IMDSv2 to output instance identity and zone location:
   ```bash
   #!/bin/bash
   dnf update -y
   dnf install -y httpd
   systemctl start httpd
   systemctl enable httpd

   # Retrieve Instance ID and Availability Zone via IMDSv2 session token
   TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
   INSTANCE_ID=`curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/instance-id`
   AZ=`curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/placement/availability-zone`

   # Generate dynamic response webpage
   echo "<html><body style='font-family: Arial; text-align: center; margin-top: 50px;'>" > /var/www/html/index.html
   echo "<h1>🚀 Nguyen Le Van Duy - High Availability Web App</h1>" >> /var/www/html/index.html
   echo "<p>Served by Instance: <b>$INSTANCE_ID</b></p>" >> /var/www/html/index.html
   echo "<p>Running in Availability Zone: <span style='color: green;'><b>$AZ</b></span></p>" >> /var/www/html/index.html
   echo "</body></html>" >> /var/www/html/index.html
   ```

2. **Auto Scaling Group Configuration (`Duy-Web-ASG`):**
   - Associated with Launch Template `Duy-Web-LT`.
   - Subnet distribution: Multi-AZ across both Public Subnets.
   - Load Balancer integration: Attached to `Duy-Target-Group`.
   - Capacity quotas:
     - **Desired capacity:** `2`
     - **Minimum capacity:** `2`
     - **Maximum capacity:** `4`
   - Configured **Target Tracking Scaling Policy**:
     - Metric: `Average CPU utilization`
     - Target value: `50%` (Automatically triggers scale-out actions when average CPU crosses 50%).

---

#### Thursday (01/10/2026): Multi-AZ Fault Tolerance & Edge CDN Exploration

1. **Simulated High Load Stress Test (Scale-Out Verification):**
   - Logged into an ASG-spawned EC2 instance via EC2 Instance Connect.
   - Installed the Linux `stress` utility:
   ```bash
   sudo dnf install -y stress
   # Spawn 2 CPU-bound worker processes targeting 100% load for 300 seconds
   stress --cpu 2 --timeout 300
   ```
   - CloudWatch & ASG observation: Average CPU exceeded the 50% threshold within minutes.
   - ASG initiated a **Scaling Activity**, automatically provisioning a 3rd instance and seamlessly registering it with the ALB without manual administrative intervention.
   - Once the workload ceased and CPU normalized, the policy executed a **Scale-in** action, gracefully terminating excess nodes back to 2.

2. **Validating Self-Healing and Zero Downtime:**
   - Manually terminated one EC2 instance directly in the AWS Console.
   - Target Group health checks immediately flagged the instance as `Unhealthy` and ceased sending requests to it.
   - Within seconds, ASG detected capacity drop (`1 < Desired 2`) and automatically provisioned a fresh replacement instance (**Self-healing**), preserving uptime without service interruption.

3. **Exploration of Amazon CloudFront & Amazon Route 53:**
   - Studied how **CloudFront's** worldwide Edge Location network caches static web assets (images, CSS, media), reducing round-trip latency and offloading origin load from the ALB.
   - Reviewed **Route 53** routing policies (Simple, Weighted, Latency, and Failover routing).

---

#### Friday (02/10/2026): Proposal Finalization & Cost-conscious Teardown

1. **Consolidating Proposal Specifications:**
   - Completed the high-level architecture diagram and component definitions for the upcoming proposal milestone.
   - Prepared service inventory: Amazon VPC, ALB, Auto Scaling EC2, Amazon RDS MySQL Multi-AZ, and Amazon S3.
   - Allocated documentation sections across team members for the formal proposal deliverable.
2. **Safe Resource Teardown (Cost Governance):**
   - Set ASG Desired and Minimum capacity to 0, then deleted the Auto Scaling Group.
   - Deleted the Application Load Balancer and Target Group.
   - Removed temporary Security Groups to maintain a clean cloud environment and prevent accidental budget leakage.

---

### Week 3 Achievements:

* **Load Balancing Mastery:** Acquired concrete expertise configuring ALBs, Target Groups, and granular HTTP health check thresholds for early fault detection.
* **Elastic Infrastructure Automation:** Designed Launch Templates utilizing IMDSv2 metadata retrieval and orchestrated CPU-driven Target Tracking scaling policies.
* **Validated High Availability & Resilience:**
  * Successfully verified the **Self-healing** capability of AWS Auto Scaling during simulated instance crashes.
  * Ensured uninterrupted Zero Downtime availability across multiple geographic Availability Zones (Multi-AZ).
* **Clear Capstone Project Direction:** Unified team vision on a robust 3-Tier architecture proposal deliverable.
* **Proactive Cost Discipline:** Maintained strict hygiene by promptly decommissioning billable load balancers and auto scaling groups immediately following verification.

---

### Challenges Encountered & Solutions:

| Problem Encountered | Root Cause | Solution Implemented |
| ------------------- | ---------- | -------------------- |
| **ALB reported instances as permanently `Unhealthy`** | Health checks expected `/index.html`, but Apache was slow to initialize, or the EC2 Security Group did not permit ALB ingress traffic. | Verified the User Data script ensured `systemctl start httpd` executed successfully, and updated EC2 Inbound Rules to explicitly permit port 80 only from the **ALB's Security Group ID**. |
| **User Data failed to retrieve Instance ID (IMDSv2 requirement)** | Amazon Linux 2023 defaults to IMDSv2, requiring a PUT session token before allowing metadata queries. | Added a `curl -X PUT` token generation step with header `X-aws-ec2-metadata-token-ttl-seconds` before querying `instance-id` and `placement/availability-zone`. |
| **Auto Scaling reaction time felt delayed during stress tests** | Default CloudWatch metric collection intervals and cooldown/warm-up periods were set conservatively. | Tuned evaluation periods and warmup duration in the Scaling Policy configuration to make responsiveness suitable for experimental validation. |

---

### Lessons Learned & Best Practices:

1. **Security Group Chaining:**
   * Never expose compute instances directly to the public Internet (`0.0.0.0/0`). Restrict inbound traffic strictly to the security group of the front-facing Load Balancer.
2. **Design for Failure across Multi-AZ:**
   * Hardware failures are inevitable at cloud scale. Spanning compute clusters across at least **two Availability Zones** is mandatory for production-grade reliability.
3. **Operational Power of Instance Metadata (IMDSv2):**
   * Dynamically querying metadata empowers instances to inspect their own operating context (Zone, Region, ID), streamlining troubleshooting and distributed logging.

---

### Plan for Next Week (Week 4):

* Finalize and submit the formal **Project Proposal Deliverable**: Problem statement, detailed 3-tier architecture, and AWS Pricing Calculator budget analysis.
* Begin initial baseline infrastructure provisioning for the approved project design.
* Prepare presentation slides for the Proposal Defense before Mentors and FCAJ peers.
