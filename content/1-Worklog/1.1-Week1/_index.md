---
title: "Week 1 Worklog"
date: 2026-09-14
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:

* Get acquainted with team members and understand internship regulations at First Cloud AI Journey (FCAJ).
* Explore the revamped **AWS Free Tier 2025** program and credit redemption pathways.
* Create a personal AWS account and establish core security baselines (MFA, IAM User).
* Provision and clean up a first virtual server with **Amazon EC2**.
* Experiment with serverless computing on **AWS Lambda** and generative AI on **Amazon Bedrock**.
* Complete onboarding milestone activities to claim initial promotional credits.

---

### Weekly Task Summary:

| Day | Primary Tasks | Start Date | Completion Date | Reference Material |
| --- | ------------- | ---------- | --------------- | ------------------ |
| **2** | **Onboarding & Internship Orientation:** Meet FCAJ team members, review program rules, expectations, and roadmap. | 14/09/2026 | 14/09/2026 | Internal FCAJ Docs |
| **3** | **Explore AWS Free Tier 2025:** Study *"AWS Free Tier 2025: Revolution!"*, understand credit allocation and Free vs Paid account plans. | 15/09/2026 | 15/09/2026 | [AWS Free Tier 2025](https://aws.amazon.com/free/) |
| **4** | **AWS Account Setup & Hardening:** Register account, activate MFA for the root user, create daily IAM User. | 16/09/2026 | 16/09/2026 | [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| **5** | **Hands-on with Amazon EC2:** Launch an EC2 instance (Amazon Linux 2023), configure Security Groups, and verify connectivity. | 17/09/2026 | 17/09/2026 | [EC2 User Guide](https://docs.aws.amazon.com/ec2/) |
| **6** | **Serverless & GenAI Exploration:** Build a Node.js Lambda function with a Function URL; request Model Access and test prompts on Amazon Bedrock. | 18/09/2026 | 18/09/2026 | [AWS Lambda & Bedrock](https://docs.aws.amazon.com/bedrock/) |

---

### Detailed Technical Log:

#### Monday (14/09/2026): Onboarding & Internship Orientation

1. **Team Introduction & Alignment:**
   - Attended the kickoff session for the **Workforce Bootcamp - First Cloud AI Journey** program.
   - Connected with assigned mentors and peer intern teammates.
2. **Reviewing Guidelines & Tooling:**
   - Studied guidelines on maintaining the Hugo internship documentation repository.
   - Reviewed key milestones: Weekly Worklogs, Project Proposal, Thematic Workshop, and Final Evaluation.
   - Initialized personal development workstation (Git, Visual Studio Code).

---

#### Tuesday (15/09/2026): Deep Dive into AWS Free Tier 2025 Architecture

1. **Core Innovations of the 2025 Free Tier Program:**
   - Transitioned away from the legacy service-hour model (750 hours/month over 12 months) toward a **direct promotional credit model**:
     - **Initial $100 Credit:** Automatically activated upon successful account registration.
     - **Up to $100 Bonus Credit:** Earned by completing 5 practical hands-on activities ($20 each: EC2, RDS, Lambda, Bedrock, AWS Budgets).
   - Dual-tier plan structure:
     - *Free Account Plan:* 6-month validity window, ideal for educational bootcamp tracks.
     - *Paid Account Plan:* Uncapped operational lifetime for production workloads.
   - Curated catalog of 30+ **Always Free** tier services with perpetual baseline quotas.

2. **Cost Avoidance Strategy:**
   - Identified high-risk cost drivers to avoid during early learning (NAT Gateways, unattached Elastic IPs, provisioned high-IOPS storage).

---

#### Wednesday (16/09/2026): AWS Account Provisioning & IAM Hardening

1. **Account Registration and Root MFA Activation:**
   - Completed registration and global billing identity verification.
   - Immediately accessed **IAM Console** $\rightarrow$ **Security credentials**:
     - Configured **Multi-Factor Authentication (MFA)** on the Root user using Google Authenticator.
     - Synchronized consecutive TOTP token codes to secure the account perimeter.

2. **IAM User Provisioning (Least Privilege Principle):**
   - Navigated to **IAM** $\rightarrow$ **Users** $\rightarrow$ **Create user**:
     - Username: `duy-admin`
     - Granted AWS Management Console access.
     - Attached administrative policy: `AdministratorAccess` (managed via an Admin Group).
   - Logged out of the Root account permanently; routed all future engineering workflows through the dedicated IAM sign-in portal (`https://<account-id>.signin.aws.amazon.com/console`).

---

#### Thursday (17/09/2026): Amazon EC2 Provisioning & Lifecycle Management

1. **Launching the First Virtual Machine:**
   - Selected working region: **Singapore (`ap-southeast-1`)**.
   - Navigated to **EC2 Dashboard** $\rightarrow$ **Launch Instance**:
     - **Name:** `Duy-Demo-EC2-Instance`
     - **AMI:** `Amazon Linux 2023 AMI` (Free tier eligible, x86_64).
     - **Instance Type:** `t2.micro` (or `t3.micro`).
     - **Key pair:** Generated and securely archived `duy-ec2-key.pem`.
   - Security Group Inbound Rules:
     - Rule 1: **SSH (Port 22)** restricted to source `My IP`.
     - Rule 2: **HTTP (Port 80)** open to `Anywhere (0.0.0.0/0)`.

2. **Connection & Teardown Verification:**
   - Connected via **EC2 Instance Connect** in the browser console.
   - Executed baseline diagnostic commands:
     ```bash
     uname -a
     cat /etc/os-release
     free -m
     ```
   - Following milestone confirmation and credit attribution, selected **Instance state $\rightarrow$ Terminate instance** to permanently release resources and prevent idle consumption.

---

#### Friday (18/09/2026): Serverless with AWS Lambda & Generative AI on Amazon Bedrock

1. **Serverless Function Deployment (AWS Lambda):**
   - Navigated to **AWS Lambda Console** $\rightarrow$ **Create function**:
     - Name: `duy-hello-serverless`
     - Runtime: **Node.js 20.x**
     - Architecture: `x86_64`
   - Configured function handler logic:
     ```javascript
     export const handler = async (event) => {
         const response = {
             statusCode: 200,
             headers: {
                 "Content-Type": "application/json"
             },
             body: JSON.stringify({
                 message: "Hello from Nguyen Le Van Duy's Lambda Serverless Function!",
                 timestamp: new Date().toISOString(),
                 status: "Success"
             }),
         };
         return response;
     };
     ```
   - Under **Configuration $\rightarrow$ Function URL**, created an unauthenticated public HTTPS endpoint. Verified instantaneous JSON responses in the web browser without managing any server runtime.

2. **Generative AI Playground Testing (Amazon Bedrock):**
   - Switched to **US East (N. Virginia - `us-east-1`)** for maximum foundation model coverage.
   - Under **Amazon Bedrock $\rightarrow$ Model access**, submitted access requests for foundation models (Anthropic Claude, Amazon Titan).
   - Opened **Bedrock Playgrounds $\rightarrow$ Text / Chat**:
     - Submitted technical evaluation prompts: *"Explain the core difference between Amazon EC2 and AWS Lambda in 3 concise bullet points."*
     - Evaluated latency, coherence, and accuracy across model variants.
   - Successfully collected promotional credit allocations for Lambda and Bedrock milestone achievements.

---

### Week 1 Achievements:

* **Security Baselines:** Applied the Principle of Least Privilege, enabled Root MFA, and migrated all interactive sessions to dedicated IAM User credentials.
* **Mastered Free Tier Dynamics:** Understood credit redemption pathways under the AWS Free Tier 2025 framework to safely optimize sandbox expenditures.
* **Virtual Server Competence:** Hands-on experience configuring AMIs, instance types, key pairs, and ingress Security Group rules.
* **Serverless Foundations:** Grasped event-driven compute architecture using AWS Lambda and Function URLs.
* **GenAI Exposure:** Successfully requested Model Access and interacted with leading Foundation Models on Amazon Bedrock.
* **Account Progress:** Successfully registered onboarding milestones and confirmed bonus credit balances.

---

### Challenges Encountered & Solutions:

| Problem Encountered | Root Cause | Solution Implemented |
| ------------------- | ---------- | -------------------- |
| **Cannot use foundation models on Amazon Bedrock immediately** | Foundation models are locked by default and require access requests; some models are unavailable in certain regions. | Navigated to **Model access** in the Bedrock Console, submitted the use case request form, and switched to a region with broad model availability (`us-east-1` - N. Virginia). |
| **SSH connection error to EC2 (Connection timed out / Permission denied)** | The Security Group did not allow port 22 from the current public IP, or key permissions were misconfigured. | Updated the Inbound Rules of the Security Group to allow port 22 with the source set to `My IP`. Also utilized **EC2 Instance Connect** directly from the browser console for quick and seamless browser-based SSH access. |
| **Concern regarding unexpected cloud costs** | Unfamiliarity with hourly pay-as-you-go pricing models on cloud platforms. | Established a routine of verifying resource status and **Terminating** EC2 instances immediately after hands-on sessions; monitored the **Billing & Cost Management Dashboard** regularly. |

---

### Lessons Learned & Best Practices:

1. **Principle of Least Privilege & Security First:**
   * Never use the AWS Root account for everyday development and administrative tasks.
   * Enforce Multi-Factor Authentication (MFA) from day one to protect cloud assets.
2. **Cost Awareness on Cloud:**
   * Even with Free Tier credits, cleaning up resources after every lab session is a fundamental discipline for a Cloud Engineer to prevent credit exhaustion or unexpected credit card billing.
3. **Serverless Architecture vs Traditional Virtual Servers:**
   * Experiencing AWS Lambda demonstrated the distinct advantage of serverless computing: rapid deployment without the burden of OS provisioning, network plumbing, or worrying about shutting down idle servers.

---

### Plan for Next Week (Week 2):

* Install and configure **AWS CLI** locally with IAM Access Keys to operate directly from the terminal.
* Study fundamental cloud networking on AWS: **Amazon VPC**, Public/Private Subnets, Internet Gateways, and Route Tables.
* Practice hands-on with object storage (**Amazon S3**) and block storage (**Amazon EBS**).
* Complete remaining Free Tier bonus challenges (**Amazon RDS** database and **AWS Budgets** alerts).
