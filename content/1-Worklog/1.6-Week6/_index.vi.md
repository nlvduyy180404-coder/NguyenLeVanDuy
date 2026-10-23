---
title: "Worklog Tuần 6"
date: 2026-10-19
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

* Tích hợp toàn diện chuỗi kiến trúc ứng dụng Web 3 lớp (**Full-stack 3-Tier Integration**): Ingress (ALB) $\rightarrow$ Compute (Auto Scaling EC2 Private) $\rightarrow$ Persistence (RDS MySQL & S3).
* Triển khai ứng dụng Web động (Dynamic Web Application) đọc ghi dữ liệu thời gian thực từ RDS MySQL và tải media từ S3.
* Tối ưu hóa phân phối nội dung toàn cầu với **Amazon CloudFront (CDN)** kết hợp cơ chế bảo vệ **Origin Access Control (OAC)**.
* Bảo mật thông tin xác thực cơ sở dữ liệu bằng **AWS Systems Manager Parameter Store** (SecureString mã hóa KMS).
* Thực hiện bài kiểm thử khả năng chịu lỗi toàn diện (**Full-stack Resilience & RDS Failover Test**).
* Đánh giá tổng kết mốc giữa kỳ thực tập (Mid-term Evaluation), cập nhật toàn bộ báo cáo lên GitHub Pages và hoạch định lộ trình cho giai đoạn 2.

---

### Tổng quan tiến độ công việc trong tuần:

| Thứ | Công việc chính | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------------- | ------------ | --------------- | -------------- |
| **2** | **Tích hợp Chuỗi Kiến trúc 3 Lớp:** Kết nối liên hoàn ALB tại Public Subnet $\rightarrow$ EC2 trong Private Subnet $\rightarrow$ RDS MySQL qua Security Group Chaining. | 19/10/2026 | 19/10/2026 | [AWS 3-Tier Architecture Pattern](https://aws.amazon.com/architecture/) |
| **3** | **Triển khai Ứng dụng Web Động & SSM Parameter Store:** Cấu hình mã nguồn web truy vấn dữ liệu sản phẩm, lưu trữ mật khẩu DB an toàn trên Parameter Store. | 20/10/2026 | 20/10/2026 | [AWS Systems Manager Guide](https://docs.aws.amazon.com/systems-manager/) |
| **4** | **Cấu hình CDN Amazon CloudFront & OAC S3:** Thiết lập CloudFront Distribution với 2 Origins (ALB cho dynamic traffic, S3 cho static assets). | 21/10/2026 | 21/10/2026 | [CloudFront OAC Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html) |
| **5** | **Kiểm thử Chịu lỗi Toàn diện (Failover & Load Test):** Thực hiện `Reboot with Failover` trên RDS Multi-AZ, đánh giá khả năng tự phục hồi và theo dõi CloudWatch metrics. | 22/10/2026 | 22/10/2026 | [Amazon RDS Multi-AZ Failover](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html) |
| **6** | **Tổng kết Mốc Giữa Kỳ & Cập nhật Báo cáo:** Đánh giá tiến độ nửa kỳ thực tập, hoàn tất tài liệu Hugo Worklog trên GitHub Pages nộp link cho Mentor. | 23/10/2026 | 23/10/2026 | Báo cáo tiến độ FCAJ |

---

### Nhật ký kỹ thuật chi tiết từng ngày (Detailed Technical Log):

#### Thứ Hai (19/10/2026): Tích Hợp Chuỗi Kiến Trúc 3-Tier Liên Hoàn

1. **Sơ đồ luồng lưu lượng và ranh giới bảo mật:**
   - Người dùng Internet $\rightarrow$ **Application Load Balancer** (`10.10.1.0/24`, `10.10.2.0/24` - Public Subnets).
   - ALB $\rightarrow$ **Auto Scaling EC2 Cluster** (`10.10.11.0/24`, `10.10.12.0/24` - Private App Subnets).
   - EC2 Instances $\rightarrow$ **Amazon RDS MySQL** (`10.10.11.0/24`, `10.10.12.0/24` - Private DB Subnets).
   - EC2 Instances $\rightarrow$ **Amazon S3** (qua S3 Gateway Endpoint nội bộ).

2. **Cấu hình Security Group liên hoàn (Chaining Verification):**
   - Rà soát chính xác các Inbound Rules để đảm bảo nguyên tắc phân quyền tối thiểu (Least Privilege):
     - `ALB-SG`: Mở cổng `80` (HTTP) và `443` (HTTPS) cho nguồn `0.0.0.0/0`.
     - `Web-App-SG`: Chỉ mở cổng `80` với nguồn là ID của `ALB-SG` (Tuyệt đối chặn kết nối trực tiếp từ Internet vào máy chủ ứng dụng).
     - `Database-SG`: Chỉ mở cổng `3306` với nguồn là ID của `Web-App-SG` (Ngăn chặn mọi truy cập trái phép không xuất phát từ máy chủ ứng dụng).

---

#### Thứ Ba (20/10/2026): Triển Khai Ứng Dụng Web Động & Bảo Mật với SSM Parameter Store

1. **Lưu trữ thông tin xác thực an toàn trên AWS Systems Manager:**
   - Tuyệt đối không hardcode mật khẩu cơ sở dữ liệu vào mã nguồn ứng dụng hoặc file cấu hình máy chủ.
   - Tạo Parameter bảo mật trên **SSM Parameter Store**:
     ```bash
     # Lưu chuỗi kết nối và mật khẩu với mã hóa KMS (SecureString)
     aws ssm put-parameter \
         --name "/app/production/db_endpoint" \
         --value "duy-production-mysql.xxxxxx.ap-southeast-1.rds.amazonaws.com" \
         --type "String" --overwrite

     aws ssm put-parameter \
         --name "/app/production/db_password" \
         --value "SuperSecurePassword2026!" \
         --type "SecureString" --overwrite
     ```
   - Gắn quyền `ssm:GetParameter` vào **IAM Instance Role** của các máy ảo EC2 để ứng dụng tự động truy vấn thông số khi khởi động.

2. **Mã nguồn ứng dụng hiển thị sản phẩm động:**
   - Ứng dụng Web tự động lấy thông tin máy chủ đang phục vụ (Instance ID, AZ) và truy vấn danh sách sản phẩm từ bảng `products` trong RDS MySQL:
   ```php
   <?php
   // Lấy credentials từ AWS Systems Manager Parameter Store thông qua AWS SDK
   require 'vendor/autoload.php';
   use Aws\Ssm\SsmClient;

   $ssm = new SsmClient(['region' => 'ap-southeast-1', 'version' => 'latest']);
   $db_host = $ssm->getParameter(['Name' => '/app/production/db_endpoint'])['Parameter']['Value'];
   $db_pass = $ssm->getParameter(['Name' => '/app/production/db_password', 'WithDecryption' => true])['Parameter']['Value'];

   $conn = new mysqli($db_host, 'admin_duy', $db_pass, 'ecom_store_db', 3306);
   if ($conn->connect_error) { die("DB Connection Failed: " . $conn->connect_error); }

   $result = $conn->query("SELECT sku, name, price, stock_quantity FROM products");
   echo "<h2>🔥 Danh Mục Sản Phẩm (Dữ Liệu Đọc Trực Tiếp từ RDS MySQL)</h2>";
   echo "<table border='1' cellpadding='8' style='border-collapse: collapse; margin: auto;'>";
   echo "<tr><th>SKU</th><th>Tên sản phẩm</th><th>Đơn giá (USD)</th><th>Kho</th></tr>";
   while($row = $result->fetch_assoc()) {
       echo "<tr><td>{$row['sku']}</td><td>{$row['name']}</td><td>\${$row['price']}</td><td>{$row['stock_quantity']}</td></tr>";
   }
   echo "</table>";
   ?>
   ```

---

#### Thứ Tư (21/10/2026): Cấu Hình Amazon CloudFront CDN & Origin Access Control (OAC)

1. **Thiết lập CloudFront Multi-Origin Distribution:**
   - **Origin 1 (Dynamic Ingress):** Trỏ đến DNS của Application Load Balancer (`duy-public-alb-xxxx.elb.amazonaws.com`).
     - Behavior: Default path pattern (`*`).
     - Cache Policy: `CachingDisabled` (Luôn chuyển tiếp request về máy chủ để lấy dữ liệu sản phẩm mới nhất).
   - **Origin 2 (Static Assets):** Trỏ đến S3 Bucket (`duy-project-app-assets-2026.s3.ap-southeast-1.amazonaws.com`).
     - Behavior: Path pattern (`/images/*`).
     - Cache Policy: `CachingOptimized` (Cache ảnh tại các Edge Location trong 24 giờ để tăng tốc độ tải trang).

2. **Bảo vệ S3 Bucket với Origin Access Control (OAC):**
   - Khóa hoàn toàn quyền truy cập trực tiếp từ Internet vào S3 Bucket (`Block all public access: ON`).
   - Tạo **Origin Access Control (OAC)** gán vào CloudFront.
   - Cập nhật **S3 Bucket Policy** chỉ cho phép Service Principal của CloudFront đọc file thông qua OAC:
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

#### Thứ Năm (22/10/2026): Kiểm Thử Chịu Lỗi Toàn Diện (Failover & Load Testing)

1. **Bài kiểm thử chuyển đổi dự phòng cơ sở dữ liệu (RDS Multi-AZ Failover Drill):**
   - Khởi động một script liên tục gửi truy vấn HTTP đọc dữ liệu sản phẩm qua URL CloudFront mỗi 1 giây.
   - Trên AWS RDS Console, chọn instance `duy-production-mysql` $\rightarrow$ **Actions** $\rightarrow$ **Reboot** $\rightarrow$ Tích chọn **Reboot with Failover**.
   - **Hiện tượng quan sát được:**
     - Giây 0 - 25: Ứng dụng web vẫn phản hồi bình thường nhờ kết nối cache và các truy vấn tĩnh từ CloudFront.
     - Giây 26 - 38: Xuất hiện thông báo gián đoạn kết nối tạm thời khi RDS chuyển quyền kiểm soát từ node Primary sang Standby.
     - Giây 39 trở đi: DNS nội bộ của RDS tự động cập nhật trỏ sang IP của máy chủ dự phòng mới; ứng dụng Web tự động kết nối lại thành công hoàn toàn mà không cần khởi động lại máy ảo EC2.
     - **Kết quả:** Thời gian RTO thực tế chỉ khoảng **38 giây**, vượt xa mục tiêu cam kết ban đầu (< 2 phút).

2. **Giám sát hiệu năng hệ thống qua Amazon CloudWatch:**
   - Theo dõi bảng điều khiển CloudWatch Metrics:
     - `ALB RequestCount`: ~1,500 requests trong thời gian kiểm thử.
     - `TargetResponseTime`: Trung bình 18ms (rất nhanh).
     - `CloudFront CacheHitRate`: Đạt 86% đối với các tệp hình ảnh `/images/*`.

---

#### Thứ Sáu (23/10/2026): Tổng Kết Mốc Giữa Kỳ & Cập Nhật Báo Cáo Thực Tập

1. **Đánh giá tiến độ nửa chặng đường (Mid-term Evaluation):**
   - Đạt 100% mục tiêu giai đoạn 1 đề ra: Từ người mới làm quen với AWS Console/CLI đã xây dựng thành công kiến trúc ứng dụng Web 3 lớp hoàn chỉnh, tự động co giãn, có khả năng tự phục hồi và bảo mật nhiều tầng.
   - Nắm vững các khái niệm cốt lõi: VPC, Subnet, Route Table, Internet/NAT Gateway, ALB, ASG, RDS Multi-AZ, S3, CloudFront, IAM, KMS, SSM Parameter Store.
2. **Chuẩn hóa và đồng bộ hệ thống báo cáo:**
   - Cập nhật toàn bộ nội dung Worklog từ Tuần 1 đến Tuần 6 trên website cá nhân bằng Hugo.
   - Kiểm thử toàn diện đường dẫn, giao diện song ngữ Anh - Việt, đảm bảo không có link hỏng trước khi nộp link website chính thức cho Mentor FCAJ: `https://nlvduyy180404-coder.github.io/NguyenLeVanDuy/`.
3. **Hoạch định mục tiêu giai đoạn 2 (Tuần 7 – Tuần 12):**
   - Tự động hóa hạ tầng hoàn toàn bằng **Infrastructure as Code (IaC)** với Terraform hoặc AWS CloudFormation.
   - Xây dựng quy trình tích hợp và phân phối liên tục **CI/CD Pipeline** với GitHub Actions / AWS CodePipeline.
   - Triển khai đề tài chuyên sâu cho bài thuyết trình Workshop kết thúc kỳ thực tập.

---

### Kết quả đạt được tuần 6:

* **Hiện thực hóa 100% Kiến trúc 3-Tier:** Vận hành thành công toàn bộ chuỗi hệ thống hoàn chỉnh từ Ingress (ALB/CloudFront) $\rightarrow$ Compute (Auto Scaling EC2) $\rightarrow$ Database (RDS MySQL) trong môi trường mạng phân tầng bảo mật.
* **Tối ưu hóa Phân phối với CloudFront OAC:** Cấu hình thành công CDN đa nguồn (Multi-Origin), bảo vệ an toàn kho tệp S3 bằng Origin Access Control, đạt tỷ lệ Cache Hit Rate 86%.
* **Bảo mật Dữ liệu Đạt Chuẩn Doanh nghiệp:** Ứng dụng cơ chế mã hóa tĩnh AWS KMS, quản lý bí mật tập trung với SSM Parameter Store, loại bỏ hoàn toàn việc lưu trữ credentials trong mã nguồn.
* **Xác thực Khả năng Phục hồi Tự động:** Chứng minh thành công khả năng chuyển đổi dự phòng RDS Multi-AZ với thời gian gián đoạn chỉ ~38 giây (Zero Data Loss).
* **Hoàn thành Xuất sắc Giai đoạn 1:** Khép lại nửa đầu kỳ thực tập với hệ thống báo cáo trực quan, chuyên nghiệp được xuất bản đầy đủ trên GitHub Pages.

---

### Khó khăn gặp phải & Cách giải quyết:

| Vấn đề kỹ thuật gặp phải | Nguyên nhân gốc rễ | Cách khắc phục đã thực hiện |
| ------------------------ | ------------------ | --------------------------- |
| **Ứng dụng Web báo lỗi `Access Denied` khi gọi SSM Parameter Store** | IAM Role gắn vào EC2 Instance chỉ có quyền cơ bản, chưa được cấp quyền giải mã khóa KMS (`kms:Decrypt`) khi đọc tham số loại `SecureString`. | Bổ sung thêm quyền `kms:Decrypt` vào Inline Policy của IAM Role cho phép giải mã các secret thuộc khóa KMS mặc định của Systems Manager. |
| **Lỗi 403 Forbidden khi CloudFront tải ảnh từ S3 Bucket** | S3 Bucket Policy cấu hình điều kiện `AWS:SourceArn` bị sai ID Distribution hoặc chưa kích hoạt chính sách OAC trên CloudFront. | Kiểm tra lại chính xác ARN của CloudFront Distribution trong phần điều kiện `Condition` của S3 Bucket Policy và chọn đúng OAC profile trong Origin Settings của CloudFront. |
| **Kết nối database bị đứt quãng trong quá trình Failover** | Ứng dụng Web ban đầu không có cơ chế tự động thử lại kết nối (Retry Connection Logic) khi DNS của RDS đang thay đổi IP. | Bổ sung logic thử lại tự động (Exponential Backoff & Retry) trong code kết nối database để ứng dụng tự động thử lại kết nối sau mỗi 3 giây thay vì crash ngay lập tức. |

---

### Bài học kinh nghiệm rút ra:

1. **Quản trị Bí mật Tập trung (Centralized Secret Management):**
   * Tuyệt đối không bao giờ đưa mật khẩu, API key vào code hoặc biến môi trường tĩnh trên server. Luôn sử dụng SSM Parameter Store hoặc AWS Secrets Manager để dễ dàng xoay vòng mật khẩu (Secret Rotation).
2. **Sức mạnh kết hợp giữa ALB và CloudFront:**
   * CloudFront không chỉ dành cho file tĩnh; việc kết hợp CloudFront đứng trước ALB giúp tăng tốc độ mạng thông qua các điểm hiện diện Edge Location của AWS và cung cấp lớp bảo vệ chống tấn công DDoS miễn phí (AWS Shield Standard).
3. **Kiểm thử Thực tế là Thước đo Duy nhất của Độ Sẵn Sàng:**
   * Không thể khẳng định hệ thống có tính sẵn sàng cao nếu chưa thực hiện bài kiểm thử sập máy chủ thật (Chaos Engineering / Failover Drill). Quá trình kiểm thử giúp phát hiện ra các vấn đề nhỏ nhưng nguy hiểm như thiếu cơ chế retry kết nối.

---

### Kế hoạch giai đoạn 2 (Tuần 7 – Tuần 12):

* Nghiên cứu và đóng gói toàn bộ hạ tầng 3-Tier thành mã nguồn với **Terraform (Infrastructure as Code)**.
* Thiết lập quy trình **CI/CD Pipeline** tự động kiểm thử và triển khai ứng dụng web lên máy chủ EC2 mỗi khi đẩy code lên GitHub.
* Lựa chọn đề tài chuyên đề và bắt tay vào chuẩn bị bài thực hành chi tiết cho phần **Workshop**.
