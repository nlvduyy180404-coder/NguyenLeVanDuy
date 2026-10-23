---
title: "Week 6 Worklog"
date: 2026-10-19
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Complete full-stack integration across the **3-Tier Resilient Web Architecture**: Ingress (ALB & CloudFront) $\rightarrow$ Compute (Private Auto Scaling EC2) $\rightarrow$ Persistence (Amazon RDS MySQL & S3).
* Deploy an active dynamic web application reading product catalogs directly from RDS MySQL and delivering media assets through **Amazon CloudFront**.
* Enforce secret management governance using **AWS Systems Manager Parameter Store** (SecureString with KMS encryption).
* Conduct comprehensive resilience drills (**Full-stack Fault Tolerance & Multi-AZ RDS Failover Test**).
* Perform the mid-term milestone evaluation, publish the synchronized Hugo worklog portal on GitHub Pages, and outline phase 2 objectives.

---

### Weekly Task Summary:

| Day | Primary Tasks | Start Date | Completion Date | Reference Material |
| --- | ------------- | ---------- | --------------- | ------------------ |
| **2** | **End-to-End 3-Tier Integration:** Stitch together ALB in Public Subnets $\rightarrow$ Private EC2 Compute $\rightarrow$ Private RDS MySQL through chained Security Groups. | 19/10/2026 | 19/10/2026 | [AWS 3-Tier Architecture Pattern](https://aws.amazon.com/architecture/) |
| **3** | **Dynamic Web Deployment & SSM Parameters:** Deploy product catalog application code and retrieve database credentials securely via SSM Parameter Store. | 20/10/2026 | 20/10/2026 | [AWS Systems Manager Guide](https://docs.aws.amazon.com/systems-manager/) |
| **4** | **Amazon CloudFront CDN & S3 OAC Setup:** Configure multi-origin CloudFront distributions (ALB for dynamic routes, S3 with Origin Access Control for static media). | 21/10/2026 | 21/10/2026 | [CloudFront OAC Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html) |
| **5** | **Comprehensive Resilience & Failover Testing:** Execute an automated `Reboot with Failover` on RDS Multi-AZ, verify zero data loss, and monitor CloudWatch metrics. | 22/10/2026 | 22/10/2026 | [Amazon RDS Multi-AZ Failover](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html) |
| **6** | **Mid-Term Milestone Review & Documentation Sync:** Evaluate halfway milestone goals, finalize Hugo documentation on GitHub Pages, and map out Phase 2 plans. | 23/10/2026 | 23/10/2026 | FCAJ Progress Review |

---

### Detailed Technical Log:

#### Monday (19/10/2026): End-to-End 3-Tier Architectural Integration

1. **Traffic Flow and Multi-Tier Topology:**
   - Public Client Traffic $\rightarrow$ **Application Load Balancer** (`10.10.1.0/24`, `10.10.2.0/24` - Public Subnets).
   - Ingress ALB $\rightarrow$ **Auto Scaling EC2 Fleet** (`10.10.11.0/24`, `10.10.12.0/24` - Isolated Private App Subnets).
   - Compute Nodes $\rightarrow$ **Amazon RDS MySQL** (`10.10.11.0/24`, `10.10.12.0/24` - Isolated Private DB Subnets).
   - Compute Nodes $\rightarrow$ **Amazon S3** (routed securely via internal S3 Gateway Endpoints).

2. **Security Group Chaining Verification:**
   - Audited ingress rules to enforce strict boundary defense:
     - `ALB-SG`: Permits inbound port `80` (HTTP) and `443` (HTTPS) from `0.0.0.0/0`.
     - `Web-App-SG`: Restricts inbound port `80` exclusively to `ALB-SG` ID (rendering the compute layer immune to direct outside probes).
     - `Database-SG`: Restricts inbound port `3306` exclusively to `Web-App-SG` ID (ensuring no traffic can touch database ports from outside the application tier).

---

#### Tuesday (20/10/2026): Dynamic Web App Deployment & Parameter Store Hardening

1. **Secret Decoupling with AWS Systems Manager:**
   - Completely eliminated hardcoded database credentials from source files and environment blocks.
   - Initialized encrypted parameters in **SSM Parameter Store**:
     ```bash
     # Store connection endpoints and KMS-encrypted credentials
     aws ssm put-parameter \
         --name "/app/production/db_endpoint" \
         --value "duy-production-mysql.xxxxxx.ap-southeast-1.rds.amazonaws.com" \
         --type "String" --overwrite

     aws ssm put-parameter \
         --name "/app/production/db_password" \
         --value "SuperSecurePassword2026!" \
         --type "SecureString" --overwrite
     ```
   - Bound IAM permissions (`ssm:GetParameter` and `kms:Decrypt`) to the **EC2 IAM Instance Profile** so compute nodes retrieve configuration upon application startup.

2. **Dynamic Application Code Implementation:**
   - Built an interactive catalog view querying records dynamically from the RDS MySQL persistence tier:
   ```php
   <?php
   require 'vendor/autoload.php';
   use Aws\Ssm\SsmClient;

   $ssm = new SsmClient(['region' => 'ap-southeast-1', 'version' => 'latest']);
   $db_host = $ssm->getParameter(['Name' => '/app/production/db_endpoint'])['Parameter']['Value'];
   $db_pass = $ssm->getParameter(['Name' => '/app/production/db_password', 'WithDecryption' => true])['Parameter']['Value'];

   $conn = new mysqli($db_host, 'admin_duy', $db_pass, 'ecom_store_db', 3306);
   if ($conn->connect_error) { die("DB Connection Failed: " . $conn->connect_error); }

   $result = $conn->query("SELECT sku, name, price, stock_quantity FROM products");
   echo "<h2>🔥 Live Product Catalog (Queried from Amazon RDS MySQL)</h2>";
   echo "<table border='1' cellpadding='8' style='border-collapse: collapse; margin: auto;'>";
   echo "<tr><th>SKU</th><th>Product Name</th><th>Price (USD)</th><th>Stock</th></tr>";
   while($row = $result->fetch_assoc()) {
       echo "<tr><td>{$row['sku']}</td><td>{$row['name']}</td><td>\${$row['price']}</td><td>{$row['stock_quantity']}</td></tr>";
   }
   echo "</table>";
   ?>
   ```

---

#### Wednesday (21/10/2026): Amazon CloudFront Multi-Origin & S3 OAC Hardening

1. **Multi-Origin Distribution Architecture:**
   - **Origin 1 (Dynamic Ingress):** Points to the Application Load Balancer DNS (`duy-public-alb-xxxx.elb.amazonaws.com`).
     - Behavior: Default wildcard pattern (`*`).
     - Cache Policy: `CachingDisabled` (Passes all live catalog requests straight to compute nodes).
   - **Origin 2 (Static Media Assets):** Points to the private S3 Bucket (`duy-project-app-assets-2026.s3.ap-southeast-1.amazonaws.com`).
     - Behavior: Dedicated media path (`/images/*`).
     - Cache Policy: `CachingOptimized` (Caches static images across global Edge Locations for 24 hours).

2. **Securing Storage via Origin Access Control (OAC):**
   - Enabled S3 Block Public Access completely on the bucket.
   - Configured CloudFront **Origin Access Control (OAC)** and attached the restricted S3 policy:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": {
           "Sid": "AllowCloudFrontServicePrincipalReadOnly",
           "Effect": "Allow",
           "Principal": { "Service": "cloudfront.amazonaws.com" },
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::duy-project-app-assets-2026/*",
           "Condition": {
               "StringEquals": {
                   "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/E12345EXAMPLE"
               }
           }
       }
   }
   ```

---

#### Thursday (22/10/2026): Full-Stack Resilience & Multi-AZ RDS Failover Drill

1. **RDS Multi-AZ Failover Validation Drill:**
   - Launched continuous HTTP polling against the application endpoint at 1-second intervals.
   - Initiated a disruptive event via the RDS Console: Selected instance `duy-production-mysql` $\rightarrow$ **Reboot with Failover**.
   - **Diagnostic Observations:**
     - Seconds 0 - 25: Application continued serving cached catalog assets seamlessly.
     - Seconds 26 - 38: Transient connection failure observed while the active database promoted the standby node in AZ-2.
     - Seconds 39+: RDS internal DNS record propagated to the promoted node's IP; application connections re-established automatically without manual intervention.
     - **Result:** Actual observed RTO registered at **~38 seconds**, comfortably beating the 2-minute SLA target with Zero Data Loss.

2. **CloudWatch Metrics Analysis:**
   - Inspected operational telemetry:
     - `ALB RequestCount`: ~1,500 test requests recorded.
     - `TargetResponseTime`: Sustained average of 18ms.
     - `CloudFront CacheHitRate`: Maintained 86% efficiency on static `/images/*` paths.

---

#### Friday (23/10/2026): Mid-Term Milestone Review & Documentation Publication

1. **Mid-Term Evaluation:**
   - Reached 100% of Phase 1 learning milestones: Transitioned from basic AWS CLI onboarding to orchestrating an enterprise-grade 3-Tier Resilient Web Architecture.
   - Established complete working mastery over VPCs, Subnets, Gateways, ALBs, ASGs, Multi-AZ RDS, S3, CloudFront OAC, KMS, and SSM Parameter Store.
2. **Synchronizing Documentation:**
   - Standardized and published all Weekly Worklogs (Weeks 1 through 6) on the Hugo-powered internship portal.
   - Validated site links and multi-language routing on GitHub Pages: `https://nlvduyy180404-coder.github.io/NguyenLeVanDuy/`.
3. **Roadmap for Phase 2 (Weeks 7 through 12):**
   - Translate manual architecture into **Infrastructure as Code (IaC)** with Terraform / CloudFormation.
   - Architect automated **CI/CD Pipelines** using GitHub Actions to deploy application updates automatically.
   - Finalize the comprehensive, publication-grade **Workshop** module.

---

### Week 6 Achievements:

* **Realized Complete 3-Tier Architecture:** Successfully tied together Ingress (CloudFront/ALB), Compute (Multi-AZ Private Auto Scaling EC2), and Database (Multi-AZ RDS MySQL).
* **High-Performance Edge Caching:** Configured multi-origin CDN caching with Origin Access Control, protecting S3 assets while realizing an 86% cache hit ratio.
* **Enterprise Security Standards:** Eliminated credential leaks via KMS-encrypted SSM Parameter Store integration; restricted database access purely to internal application tiers.
* **Empirically Proven High Availability:** Conducted live RDS Multi-AZ failover drills achieving recovery within ~38 seconds without data loss.
* **Exemplary Mid-Term Documentation:** Closed Phase 1 with a thoroughly validated, bilingual internship logbook published on GitHub Pages.

---

### Challenges Encountered & Solutions:

| Problem Encountered | Root Cause | Solution Implemented |
| ------------------- | ---------- | -------------------- |
| **Web application thrown `Access Denied` when reading SSM parameters** | The EC2 IAM Role had `ssm:GetParameter` permission but lacked `kms:Decrypt` access to decrypt `SecureString` types. | Appended `kms:Decrypt` permissions to the instance role's policy document targeting the default Systems Manager KMS key. |
| **403 Forbidden errors when CloudFront pulled media from S3** | S3 Bucket Policy had a mismatched CloudFront Distribution ARN in the condition statement, or OAC was misbound. | Audited the exact Distribution ARN inside the S3 policy's `Condition` block and verified OAC profile assignment within CloudFront Origin Settings. |
| **Application crashed permanently during RDS failover** | Initial application code lacked retry connection logic, terminating the script upon the first socket disconnect. | Implemented exponential backoff and retry handling in the database connection layer, allowing the app to poll and reconnect cleanly once DNS updated. |

---

### Lessons Learned & Best Practices:

1. **Centralized Secret Governance:**
   * Never commit database passwords or API keys to source repositories. Utilize SSM Parameter Store or Secrets Manager with KMS encryption to simplify credential rotation.
2. **Synergy of CloudFront + ALB:**
   * CloudFront is not solely for static sites; pairing it in front of an ALB accelerates dynamic API traffic across AWS backbone edge networks and provides baseline DDoS mitigation (AWS Shield Standard).
3. **Resilience Requires Real Testing:**
   * High availability cannot be assumed without real chaos drills. Performing live failover simulations uncovers subtle edge cases like connection timeouts and DNS caching policies.

---

### Roadmap for Phase 2 (Weeks 7 through 12):

* Codify the entire 3-Tier infrastructure into **Terraform (Infrastructure as Code)**.
* Build automated **CI/CD Pipelines** with GitHub Actions for zero-downtime application deployments.
* Author and publish the detailed, step-by-step **Workshop** capstone module.
