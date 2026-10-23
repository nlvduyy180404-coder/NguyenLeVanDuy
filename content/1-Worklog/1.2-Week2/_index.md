---
title: "Week 2 Worklog"
date: 2026-09-21
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

* Master the AWS CLI command-line interface to interact with cloud infrastructure.
* Gain deep understanding of object storage (Amazon S3) and block storage (Amazon EBS).
* Construct a foundational networking architecture with Amazon VPC (Subnets, Internet Gateway, Route Tables).
* Automate Web Server deployment on EC2 using User Data bootstrap scripts.
* Complete all remaining AWS Free Tier challenges (Amazon RDS, AWS Budgets) to earn the maximum $200 promotional credit.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | ---- | ---------- | --------------- | ------------------ |
| 2   | - **AWS CLI Setup & Configuration:** <br>&emsp; + Generate secure Access Key & Secret Key for IAM User <br>&emsp; + Install AWS CLI locally and run `aws configure` (Region, Output format) <br>&emsp; + Verify credentials and caller identity via `aws sts get-caller-identity` | 21/09/2026 | 21/09/2026 | <https://docs.aws.amazon.com/cli/> |
| 3   | - **Explore & Practice Amazon S3:** <br>&emsp; + Create an S3 Bucket with a globally unique name <br>&emsp; + Upload, download, and manage objects via Console and CLI <br>&emsp; + Understand Block Public Access and configure Bucket Policies | 22/09/2026 | 22/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - **Build a Custom VPC:** <br>&emsp; + Create a VPC with an IPv4 CIDR block <br>&emsp; + Provision Public and Private Subnets across Availability Zones <br>&emsp; + Create an Internet Gateway (IGW) and attach it to the VPC <br>&emsp; + Configure a Route Table to route outbound traffic through the IGW | 23/09/2026 | 23/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - **Advanced EC2 Practice & EBS Storage:** <br>&emsp; + Launch an EC2 instance in the Public Subnet of the custom VPC <br>&emsp; + Apply a **User Data script** to automatically install and run a Web Server (Nginx/Apache) upon boot <br>&emsp; + Provision an additional EBS Volume, attach it to EC2, format filesystem, and mount to directory | 24/09/2026 | 24/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - **Complete Free Tier Milestone Activities:** <br>&emsp; + **Amazon RDS:** Launch a MySQL/PostgreSQL DB instance under Free Tier, test connectivity, and terminate/delete upon completion <br>&emsp; + **AWS Budgets:** Configure a Cost Budget with automatic email alerts when thresholds ($10, $20) are approached | 25/09/2026 | 25/09/2026 | <https://aws.amazon.com/free/> |


### Week 2 Achievements:

* Successfully installed and operated the **AWS CLI**, querying and managing resources directly from the local terminal without solely relying on the Console web UI.
* Mastered core **Amazon S3** mechanisms:
  * Understood the structure and organization of Buckets and Objects.
  * Differentiated between Public and Private buckets, grasping the safety mechanisms of *S3 Block Public Access*.
* Gained hands-on proficiency with **Amazon VPC** networking:
  * Designed and built a fully functional Custom VPC from scratch.
  * Clearly distinguished between **Public Subnets** (routed to Internet Gateway) and **Private Subnets** (isolated and internal).
* Mastered advanced **Amazon EC2** provisioning:
  * Used User Data scripts to automate bootstrap operations (installing and launching web server packages) during initial instance launch.
  * Understood the role of **EBS Volumes** as attached block storage, learning how to format, mount, and expand server storage.
* Fulfilled all 5 hands-on challenges of the AWS Free Tier 2025 program (EC2, RDS, Lambda, Bedrock, Budgets) and claimed the full **$200 Credit** package for uninterrupted learning.
* Successfully configured **AWS Budgets**, establishing proactive cost monitoring and preventing unexpected runaway cloud expenses.

---

### Challenges Encountered & Solutions:

| Problem Encountered | Root Cause | Solution |
| ------------------- | ---------- | -------- |
| **Access Denied (403) when attempting to access a public file on S3** | S3 Block Public Access is enabled by default at account and bucket levels to prevent unintentional data exposure. | Disabled Block Public Access settings for the specific public bucket and attached a well-scoped **Bucket Policy** (granting `s3:GetObject` action) to permit legitimate access. |
| **EBS Volume attached but storage not visible inside the EC2 operating system** | New block storage devices are unpartitioned raw blocks and must be initialized with a filesystem and mounted into the Linux hierarchy. | Executed `lsblk` to identify the device name (`/dev/xvdf` or `/dev/nvme1n1`), used `mkfs -t ext4` to format the partition, and used `mount` to link the volume to the target directory. |
| **Amazon RDS instance takes long to provision and fails external connection** | Cloud relational database provisioning typically takes 5–10 minutes; default Security Group does not accept external traffic. | Patiently waited for status transition to `Available`, updated DB Security Group Inbound Rules to open the corresponding port (3306 for MySQL / 5432 for PostgreSQL). Terminated the instance immediately after testing to prevent ongoing storage charges. |

---

### Lessons Learned & Best Practices:

1. **Absolute Security for Access Keys & Secret Keys:**
   * Never commit credentials or configuration files containing secrets to public version control repositories like GitHub.
   * Periodically audit or rotate keys that are no longer actively required.
2. **Networking is the Backbone of Cloud Architecture:**
   * Understanding packet flows from the Internet through IGWs, Route Tables, and down to Subnets makes security configuration and network troubleshooting significantly more intuitive.
3. **The Power of Infrastructure Automation:**
   * Leveraging User Data bootstrap scripts eliminates tedious manual configuration and ensures reproducibility when launching multiple server instances.

---

### Plan for Next Week (Week 3):

* Study load balancing and high availability on AWS: **Application Load Balancer (ALB)**.
* Explore dynamic scalability: **Auto Scaling Group (ASG)**.
* Practice setting up a basic fault-tolerant, multi-AZ web architecture.
