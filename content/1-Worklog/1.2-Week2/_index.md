---
title: "Week 2 Worklog"
date: 2026-09-21
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

* Master the **AWS CLI** command-line interface to inspect, configure, and manage cloud resources directly from the terminal.
* Gain comprehensive understanding of object storage (**Amazon S3**) and block storage (**Amazon EBS**).
* Architect and implement a standalone foundational cloud network with **Amazon VPC** (Custom VPC, Public/Private Subnets, Internet Gateway, Route Tables).
* Advance **Amazon EC2** administration skills: automate Web Server deployment via **User Data bootstrap scripts** and mount secondary EBS volumes.
* Fulfill all Free Tier milestone activities (Amazon RDS, AWS Budgets) to secure the maximum **$200 Promotional Credit** for the entire internship journey.

---

### Weekly Task Summary:

| Day | Primary Tasks | Start Date | Completion Date | Reference Material |
| --- | ------------- | ---------- | --------------- | ------------------ |
| **2** | **AWS CLI Setup & Identity Management:** Generate IAM Access Keys, configure CLI profile, and test permissions via the terminal. | 21/09/2026 | 21/09/2026 | [AWS CLI User Guide](https://docs.aws.amazon.com/cli/) |
| **3** | **Mastering Amazon S3 Storage:** Create Globally Unique Buckets, manage objects via CLI, configure Block Public Access & Bucket Policies. | 22/09/2026 | 22/09/2026 | [S3 Documentation](https://docs.aws.amazon.com/s3/) |
| **4** | **Amazon VPC Network Design:** Plan CIDR address blocks, configure Public/Private Subnets, Internet Gateway, and Route Tables. | 23/09/2026 | 23/09/2026 | [VPC User Guide](https://docs.aws.amazon.com/vpc/) |
| **5** | **EC2 Bootstrap & EBS Storage:** Launch EC2 with User Data to automate Web Server provisioning; attach, format, and mount new EBS volumes. | 24/09/2026 | 24/09/2026 | [EC2 User Guide](https://docs.aws.amazon.com/ec2/) |
| **6** | **Free Tier Milestones ($200 Credit):** Provision an Amazon RDS relational database instance and establish budget alerts using AWS Budgets. | 25/09/2026 | 25/09/2026 | [AWS Free Tier 2025](https://aws.amazon.com/free/) |

---

### Detailed Technical Log:

#### Monday (21/09/2026): AWS CLI Installation & Configuration

1. **Create Access Key for IAM User:**
   - Log in to the IAM Console using the IAM User credentials established in Week 1.
   - Navigate to **Security credentials** $\rightarrow$ select **Create access key** $\rightarrow$ select *Command Line Interface (CLI)* use case.
   - Securely save the `Access Key ID` and `Secret Access Key` pair without exposing them publicly.

2. **Terminal Configuration:**
   ```bash
   # Launch the AWS CLI configuration wizard
   aws configure
   ```
   - Provide the requested parameters:
     - `AWS Access Key ID`: `AKIA...`
     - `AWS Secret Access Key`: `wJalrXUtnFEMI...`
     - `Default region name`: `ap-southeast-1` (Singapore)
     - `Default output format`: `json`

3. **Verify Identity & Connectivity:**
   ```bash
   aws sts get-caller-identity
   ```
   *Expected JSON response:*
   ```json
   {
       "UserId": "AIDASAMPLEUSERID2026",
       "Account": "123456789012",
       "Arn": "arn:aws:iam::123456789012:user/duy-iam-user"
   }
   ```

---

#### Tuesday (22/09/2026): Object Storage Management with Amazon S3

1. **Bucket Creation and Object Operations via CLI:**
   ```bash
   # Create a new bucket with a globally unique name
   aws s3 mb s3://fcj-duy-storage-bucket-2026 --region ap-southeast-1

   # Upload a sample file to the bucket
   aws s3 cp sample-file.txt s3://fcj-duy-storage-bucket-2026/

   # List objects inside the bucket
   aws s3 ls s3://fcj-duy-storage-bucket-2026/
   ```

2. **Access Control (S3 Block Public Access & Bucket Policies):**
   - By default, Amazon S3 enforces **Block all public access** at the account and bucket levels to safeguard against accidental data leaks.
   - For specific report artifacts requiring public viewing, disabled Block Public Access and attached a well-scoped **Bucket Policy**:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicReadGetObject",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::fcj-duy-storage-bucket-2026/*"
           }
       ]
   }
   ```

---

#### Wednesday (23/09/2026): Building a Custom Amazon VPC Architecture from Scratch

1. **Network Addressing Plan:**

| Component | Resource Name | CIDR Block | Description |
| --------- | ------------- | ---------- | ----------- |
| **VPC** | `Duy-Custom-VPC` | `10.0.0.0/16` | Accommodates up to 65,536 private IP addresses |
| **Public Subnet** | `Duy-Public-Subnet-AZ1` | `10.0.1.0/24` | Associated with Route Table directing out to Internet Gateway |
| **Private Subnet** | `Duy-Private-Subnet-AZ1` | `10.0.2.0/24` | Isolated internal subnet designated for database instances |
| **Internet Gateway** | `Duy-VPC-IGW` | - | Bridge connecting the VPC to the public Internet |

2. **Route Table Configuration for Public Subnet:**
   - Provisioned a custom Route Table named `Duy-Public-RT`.
   - Added outbound routing rule:
     - **Destination:** `0.0.0.0/0` (all outbound IPv4 traffic)
     - **Target:** Internet Gateway (`Duy-VPC-IGW`)
   - Associated Subnet `10.0.1.0/24` with this route table.

---

#### Thursday (24/09/2026): EC2 Automation with User Data & EBS Volume Mounting

1. **Launch Virtual Machine with User Data Script:**
   - Under **Advanced details $\rightarrow$ User data** during EC2 launch (Amazon Linux 2023), provided an automated bootstrap script:
   ```bash
   #!/bin/bash
   # Update software packages
   dnf update -y

   # Install Apache HTTP Server
   dnf install -y httpd

   # Enable and start the service
   systemctl start httpd
   systemctl enable httpd

   # Generate default test landing page
   echo "<h1>Welcome to Nguyen Le Van Duy's Web Server on AWS!</h1>" > /var/www/html/index.html
   ```
   - As soon as the instance transitioned to `Running`, loading the Public IP in a web browser instantly rendered the test page without requiring manual SSH intervention.

2. **Attach and Mount a Secondary EBS Volume:**
   - Created a 5GB General Purpose SSD (`gp3`) volume in the same Availability Zone as the EC2 instance.
   - Selected **Attach Volume** to associate the block device with the EC2 instance.
   - Connected via SSH to partition, format, and mount the storage:
   ```bash
   # Inspect block device tree
   lsblk

   # Create an ext4 filesystem on the new volume
   sudo mkfs -t ext4 /dev/xvdf

   # Create mount point directory and mount device
   sudo mkdir /data
   sudo mount /dev/xvdf /data

   # Verify filesystem disk space usage
   df -h /data
   ```

---

#### Friday (25/09/2026): Completing Free Tier Milestones ($200 Credit)

1. **Deploy Relational Database with Amazon RDS:**
   - Service: **Amazon RDS** $\rightarrow$ Create database.
   - Engine: **MySQL** (Community Edition).
   - Template: **Free Tier** (`db.t3.micro` or `db.t4g.micro`, 20GB gp2 storage).
   - Placed the instance inside the VPC Private Subnet for security isolation, permitting inbound connections only from the EC2 Web Server Security Group over port 3306.
   - Completed connection tests and immediately initiated **Delete Database** (without final snapshot) to conserve allocated Free Tier quotas.

2. **Configure Budget Tracking via AWS Budgets:**
   - Navigated to **AWS Billing and Cost Management** $\rightarrow$ **Budgets**.
   - Created a **Cost Budget**:
     - Budget amount: **$10 USD / month**.
     - Configured notification threshold: Automatically sends alert emails to `nlvduyy180404@gmail.com` when actual spending reaches 80% ($8 USD) or forecasted spending hits 100%.
   - Successfully achieved all 5 hands-on milestones, crediting the full **$200 promotional allowance** to the account.

---

### Week 2 Achievements:

* **CLI Competence:** Gained proficiency using AWS CLI commands to query resources, configure multi-region profiles, and execute actions directly from the terminal without graphical console dependency.
* **Storage Distinction:** Mastered the architectural differences between Object Storage (S3 - files, static assets, backups) and Block Storage (EBS - raw virtual disks for operating systems and databases).
* **Network Independence:** Successfully designed and stood up an isolated Custom VPC network with separate Public/Private subnets and Internet Gateway routing.
* **Infrastructure Automation:** Demonstrated automated server bootstrapping using User Data scripts, drastically accelerating infrastructure provisioning.
* **Financial Governance & Security:** Completed all AWS Free Tier 2025 milestone challenges, earned the full $200 promotional credit, and established proactive budget alarms to avoid unexpected billing.

---

### Challenges Encountered & Solutions:

| Problem Encountered | Root Cause | Solution Implemented |
| ------------------- | ---------- | -------------------- |
| **403 Forbidden error when downloading objects via public S3 URLs** | *S3 Block Public Access* safeguards were enabled at the account and bucket levels to prevent data exposure. | Disabled Block Public Access on the specific bucket and applied a JSON **Bucket Policy** granting `s3:GetObject` permissions to public users (`*`). |
| **Attached EBS volume not visible in Linux filesystem** | Raw block devices lack a filesystem and must be explicitly formatted and mounted into the Linux hierarchy. | Ran `lsblk` to identify the device path, executed `sudo mkfs -t ext4` to format the partition, and used `sudo mount` to link the volume to `/data`. |
| **Amazon RDS instance takes prolonged time to provision and fails connection** | Managed database provisioning inherently requires 5–10 minutes; the default security group blocks all incoming network traffic. | Waited patiently until status displayed `Available`. Adjusted the DB Security Group to permit port 3306 inbound traffic from the EC2 Security Group. Promptly deleted the database after validation to prevent ongoing storage charges. |

---

### Lessons Learned & Best Practices:

1. **Credential Hygiene:**
   * Never hardcode Access Keys or Secret Keys into application codebases or commit them to public GitHub repositories.
   * Favor IAM Roles over long-lived access keys whenever AWS services communicate with one another (e.g., EC2 instance roles calling S3).
2. **Network Isolation Principles:**
   * Only position public-facing ingress components (e.g., Web Servers, Load Balancers) in Public Subnets.
   * Keep databases (RDS) and internal processing services strictly inside Private Subnets to minimize potential attack surfaces.
3. **The Value of Automated Provisioning:**
   * Utilizing User Data bootstrap scripts eliminates human error associated with repetitive manual configuration and accelerates server readiness.

---

### Plan for Next Week (Week 3):

* Explore high-availability load balancing with **Application Load Balancer (ALB)**.
* Study automated horizontal scaling with **Auto Scaling Groups (ASG)**.
* Practice orchestrating VPC + ALB + EC2 Auto Scaling to construct a resilient, fault-tolerant web architecture.
