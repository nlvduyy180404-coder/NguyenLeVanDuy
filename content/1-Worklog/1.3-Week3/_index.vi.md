---
title: "Worklog Tuần 3"
date: 2026-09-28
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Họp nhóm và làm việc với Mentor để thống nhất ý tưởng, phạm vi và kiến trúc cho đề tài đồ án thực tập (**Project Proposal**).
* Làm chủ cơ chế cân bằng tải ứng dụng với **Application Load Balancer (ALB)**, hiểu sâu về Target Group và Health Check.
* Triển khai hệ thống tự động co giãn **Auto Scaling Group (ASG)** kết hợp với **Launch Template**.
* Kiểm thử khả năng chịu lỗi và tính sẵn sàng cao (**High Availability & Self-healing**) trên môi trường đa vùng (**Multi-AZ**).
* Nghiên cứu giải pháp phân phối nội dung toàn cầu với **Amazon CloudFront (CDN)** và định tuyến tên miền **Amazon Route 53**.

---

### Tổng quan tiến độ công việc trong tuần:

| Thứ | Công việc chính | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------------- | ------------ | --------------- | -------------- |
| **2** | **Thảo luận Đồ án Proposal & Họp nhóm:** Thảo luận cùng Mentor và nhóm thực tập, phân tích bài toán kiến trúc 3 lớp (3-tier Web Architecture). | 28/09/2026 | 28/09/2026 | Tài liệu hướng dẫn Proposal FCAJ |
| **3** | **Cấu hình Cân bằng tải Application Load Balancer (ALB):** Tạo Target Group, cấu hình Health Check HTTP và triển khai ALB trải trên 2 Availability Zones. | 29/09/2026 | 29/09/2026 | [AWS ELB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/) |
| **4** | **Triển khai Tự động Co giãn Auto Scaling Group (ASG):** Tạo Launch Template chuẩn hóa, cấu hình chính sách mở rộng động (Target Tracking Scaling Policy theo CPU). | 30/09/2026 | 30/09/2026 | [AWS Auto Scaling Guide](https://docs.aws.amazon.com/autoscaling/) |
| **5** | **Kiểm thử Chịu lỗi Multi-AZ & Tìm hiểu CDN/DNS:** Giả lập tải cao bằng công cụ `stress`, kiểm tra tính năng tự phục hồi (Self-healing); tìm hiểu CloudFront & Route 53. | 01/10/2026 | 01/10/2026 | [Amazon CloudFront](https://docs.aws.amazon.com/cloudfront/) |
| **6** | **Tổng kết Thực nghiệm & Khung Đề tài Proposal:** Chốt sơ đồ kiến trúc hệ thống đề tài nhóm, lên dàn ý chi tiết báo cáo Proposal và dọn dẹp tài nguyên lab. | 02/10/2026 | 02/10/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) |

---

### Nhật ký kỹ thuật chi tiết từng ngày (Detailed Technical Log):

#### Thứ Hai (28/09/2026): Họp Nhóm Định Hướng & Thảo Luận Đề Tài Proposal

1. **Họp nhóm và trao đổi cùng Mentor:**
   - Cùng các thành viên trong nhóm thảo luận về định hướng bài tập lớn / đồ án thực tập (**Project Proposal**).
   - Nhận định hướng từ Mentor: Đề tài cần đảm bảo các tiêu chuẩn kiến trúc điện toán đám mây theo khung **AWS Well-Architected Framework**:
     - *Độ tin cậy & Sẵn sàng cao (Reliability & High Availability):* Hệ thống không có điểm lỗi đơn lẻ (Single Point of Failure - SPOF).
     - *Khả năng co giãn (Scalability):* Tự động điều chỉnh số lượng máy chủ theo lượng người dùng truy cập.
     - *Bảo mật nhiều lớp (Security in Depth):* Tách biệt Web Tier (Public) và Database Tier (Private).
2. **Thống nhất mô hình đồ án:**
   - Quyết định lựa chọn mô hình kiến trúc ứng dụng Web 3 lớp (**3-Tier Resilient Web Architecture**):
     - **Tầng 1 (Presentation / Ingress):** Amazon Route 53 + CloudFront + Application Load Balancer.
     - **Tầng 2 (Application / Compute):** Auto Scaling Group chạy Web/App Servers trên 2 Availability Zones.
     - **Tầng 3 (Database / Storage):** Amazon RDS Multi-AZ kết hợp Amazon S3 lưu trữ media.

---

#### Thứ Ba (29/09/2026): Cấu hình Cân Bằng Tải Application Load Balancer (ALB)

1. **Khởi tạo Target Group:**
   - Target type: `Instances`.
   - Protocol: `HTTP`, Port: `80`, VPC: `Duy-Custom-VPC`.
   - Cấu hình **Health checks**:
     - Protocol: `HTTP`, Path: `/index.html`
     - Healthy threshold: `3` lần liên tiếp.
     - Unhealthy threshold: `2` lần liên tiếp.
     - Timeout: `5` giây, Interval: `15` giây.

2. **Triển khai Application Load Balancer:**
   - Name: `Duy-Public-ALB`.
   - Scheme: **Internet-facing** (truy cập từ Internet).
   - IP address type: `IPv4`.
   - Network mapping: Chọn `Duy-Custom-VPC` và tích chọn **2 Availability Zones** khác nhau:
     - `ap-southeast-1a` $\rightarrow$ gán `Duy-Public-Subnet-AZ1` (`10.0.1.0/24`)
     - `ap-southeast-1b` $\rightarrow$ gán `Duy-Public-Subnet-AZ2` (`10.0.3.0/24`)
   - Security Group: Cho phép Inbound port `80` (HTTP) từ `0.0.0.0/0`.
   - Listeners and routing: HTTP port 80 forward tới `Duy-Target-Group`.

3. **Kiểm tra trạng thái điều phối traffic:**
   - Đăng ký thủ công 2 máy ảo EC2 vào Target Group.
   - Quan sát trạng thái chuyển từ `Initial` $\rightarrow$ `Healthy`.
   - Truy cập vào DNS Name của ALB: `http://duy-public-alb-12345678.ap-southeast-1.elb.amazonaws.com`.
   - Refresh liên tục trình duyệt, thấy nội dung phản hồi luân phiên hiển thị giữa 2 máy ảo, chứng minh ALB phân phối tải (Round Robin) thành công.

---

#### Thứ Tư (30/09/2026): Tự Động Co Giãn với Auto Scaling Group (ASG)

1. **Tạo Launch Template chuẩn hóa (`Duy-Web-LT`):**
   - AMI: `Amazon Linux 2023`.
   - Instance type: `t2.micro` (Free tier).
   - Security Group: Chỉ cho phép port 80 từ nguồn là **Security Group của ALB** (Bảo vệ máy chủ không bị truy cập trực tiếp từ bên ngoài).
   - Đoạn **User Data script** động giúp nhận biết máy ảo và Availability Zone đang xử lý request:
   ```bash
   #!/bin/bash
   dnf update -y
   dnf install -y httpd
   systemctl start httpd
   systemctl enable httpd

   # Lấy Instance ID và Availability Zone thông qua IMDSv2
   TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
   INSTANCE_ID=`curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/instance-id`
   AZ=`curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/placement/availability-zone`

   # Xuất trang HTML hiển thị thông tin máy ảo
   echo "<html><body style='font-family: Arial; text-align: center; margin-top: 50px;'>" > /var/www/html/index.html
   echo "<h1>🚀 Nguyen Le Van Duy - High Availability Web App</h1>" >> /var/www/html/index.html
   echo "<p>Served by Instance: <b>$INSTANCE_ID</b></p>" >> /var/www/html/index.html
   echo "<p>Running in Availability Zone: <span style='color: green;'><b>$AZ</b></span></p>" >> /var/www/html/index.html
   echo "</body></html>" >> /var/www/html/index.html
   ```

2. **Cấu hình Auto Scaling Group (`Duy-Web-ASG`):**
   - Gắn với Launch Template `Duy-Web-LT`.
   - Phân bổ mạng: Trải trên cả 2 Public Subnet ở 2 Availability Zones.
   - Tích hợp Load Balancer: Gắn trực tiếp vào `Duy-Target-Group`.
   - Cấu hình dung lượng nhóm (Group size):
     - **Desired capacity:** `2`
     - **Minimum capacity:** `2`
     - **Maximum capacity:** `4`
   - Cấu hình **Target Tracking Scaling Policy**:
     - Metric: `Average CPU utilization`
     - Target value: `50%` (Tự động scale out thêm instance khi tải CPU trung bình vượt quá 50%).

---

#### Thứ Năm (01/10/2026): Kiểm Thử Khả Năng Chịu Lỗi Multi-AZ & Nghiên Cứu CDN

1. **Thực nghiệm giả lập tải cao (Stress Testing & Scale Out):**
   - Kết nối vào một EC2 instance do ASG tạo ra.
   - Cài đặt công cụ tạo tải CPU `stress`:
   ```bash
   sudo dnf install -y stress
   # Chạy 2 tiến trình ép CPU hoạt động 100% trong 300 giây
   stress --cpu 2 --timeout 300
   ```
   - Quan sát trên CloudWatch & ASG: Sau vài phút, metric CPU trung bình vượt ngưỡng 50%.
   - ASG lập tức kích hoạt hoạt động mở rộng (**Scaling activity**): Tự động provision thêm instance thứ 3 và tự động gắn vào ALB Target Group mà không cần can thiệp thủ công.
   - Khi tiến trình `stress` kết thúc, tải CPU giảm về bình thường, ASG thực hiện thu hẹp (**Scale in**) đưa số lượng máy về 2.

2. **Thực nghiệm khả năng tự phục hồi (Self-healing & Fault Tolerance):**
   - Chủ động terminate đột ngột 1 máy ảo trên Console.
   - Target Group lập tức phát hiện instance bị `Unhealthy` và ngừng điều hướng traffic đến máy đó.
   - Trong vòng chưa đầy 1 phút, ASG phát hiện số lượng instance hiện tại (`1`) thấp hơn Desired capacity (`2`) $\rightarrow$ Tự động bật ngay một máy ảo thay thế mới toanh để duy trì cam kết sẵn sàng cao.

3. **Nghiên cứu Amazon CloudFront & Amazon Route 53:**
   - Tìm hiểu cơ chế mạng lưới máy chủ biên (Edge Locations) của **CloudFront CDN** giúp cache các tệp tĩnh (hình ảnh, CSS, video), giảm tải trực tiếp cho ALB và Web Server.
   - Nghiên cứu chính sách định tuyến của **Route 53** (Simple, Weighted, Latency, Failover routing).

---

#### Thứ Sáu (02/10/2026): Tổng Hợp Kết Quả & Hoàn Thiện Khung Proposal

1. **Chốt khung đề tài Proposal cùng nhóm:**
   - Hoàn thiện sơ đồ kiến trúc tổng thể cho đề tài đồ án tốt nghiệp bootcamp.
   - Chuẩn bị danh mục các dịch vụ sẽ sử dụng thực tế: Amazon VPC, ALB, Auto Scaling EC2, Amazon RDS MySQL, Amazon S3, CloudFront.
   - Phân chia nhiệm vụ viết tài liệu chi tiết cho bài nộp Proposal ở tuần tiếp theo.
2. **Dọn dẹp tài nguyên Lab an toàn (Cost Optimization):**
   - Xóa Auto Scaling Group (đặt Desired = 0 và xóa ASG).
   - Xóa Application Load Balancer và Target Group.
   - Xóa các Security Group và Subnet phụ để đảm bảo không phát sinh chi phí duy trì ngầm.

---

### Kết quả đạt được tuần 3:

* **Làm chủ hệ thống Cân bằng tải (ALB):** Hiểu rõ cách định tuyến traffic thông minh qua Target Group, cơ chế Health Check định kỳ giúp phát hiện sự cố máy chủ tức thì.
* **Tự động hóa co giãn đàn hồi (ASG):** Tự tay cấu hình Launch Template với IMDSv2 User Data và xây dựng chính sách Target Tracking Scaling Policy theo tải CPU.
* **Kiểm chứng thành công Kiến trúc Sẵn sàng cao (High Availability):**
  * Hệ thống vượt qua bài test giả lập sự cố máy chủ: Tự phát hiện và tự động bù đắp instance mới (**Self-healing**) hoàn toàn tự động.
  * Đảm bảo dịch vụ web vẫn hoạt động liên tục không gián đoạn (Zero Downtime) dù một vùng sẵn sàng (AZ) gặp sự cố.
* **Định hình vững chắc Đề tài Proposal:** Nhóm đã thống nhất được toàn bộ khung kiến trúc, mục tiêu kỹ thuật và lộ trình triển khai đồ án thực tập.
* **Quản trị chi phí chủ động:** Nắm vững quy trình dọn dẹp sạch sẽ các tài nguyên phức tạp (ALB, ASG) ngay sau khi hoàn thành lab.

---

### Khó khăn gặp phải & Cách giải quyết:

| Vấn đề kỹ thuật gặp phải | Nguyên nhân gốc rễ | Cách khắc phục đã thực hiện |
| ------------------------ | ------------------ | --------------------------- |
| **ALB báo trạng thái instance `Unhealthy` liên tục** | Health Check cấu hình kiểm tra đường dẫn `/index.html` nhưng Web Server chưa kịp khởi động xong hoặc Security Group của EC2 chưa mở port 80 cho ALB. | Kiểm tra lại User Data script để đảm bảo Apache đã được `systemctl start`, đồng thời cập nhật Security Group của EC2: Chỉ mở Inbound port 80 với nguồn là **Security Group của chính ALB**. |
| **Không lấy được Instance ID trong script User Data (IMDSv2)** | Hệ điều hành Amazon Linux 2023 mặc định yêu cầu chuẩn bảo mật IMDSv2 (buộc phải tạo token trước) thay vì gọi thẳng URL như IMDSv1. | Thêm bước tạo session token với `curl -X PUT` kèm header `X-aws-ec2-metadata-token-ttl-seconds` trước khi truy vấn metadata `instance-id` và `placement/availability-zone`. |
| **Auto Scaling co giãn quá chậm khi test tải CPU** | Mặc định CloudWatch Metric thu thập mỗi 5 phút hoặc khoảng thời gian Warm-up period của ASG đang để quá dài. | Điều chỉnh thông số Evaluation Periods và Warm-up period trong cấu hình Scaling Policy xuống mức phù hợp để hệ thống phản hồi nhanh nhạy hơn trong bài thử nghiệm. |

---

### Bài học kinh nghiệm rút ra:

1. **Bảo mật hạ tầng nhiều lớp (Security Group Chaining):**
   * Không bao giờ mở port 80/443 của máy chủ EC2 ra toàn thế giới (`0.0.0.0/0`). Chỉ cho phép traffic đi qua Load Balancer (ALB) bằng cách gán Source là ID của Security Group ALB.
2. **Nguyên tắc Thiết kế Hướng Sẵn sàng (Design for Failure):**
   * Trong điện toán đám mây, mọi phần cứng đều có thể gặp trục trặc bất cứ lúc nào. Việc phân bổ ứng dụng tối thiểu trên **2 Availability Zones** là yêu cầu tiên quyết cho các hệ thống môi trường production.
3. **Sức mạnh của IMDSv2 trong quản trị máy ảo:**
   * Việc tự động hóa đọc thông tin metadata của chính máy chủ giúp ứng dụng tự nhận thức được ngữ cảnh chạy (Region, Zone, Instance ID), cực kỳ hữu ích cho việc ghi log và chẩn đoán sự cố.

---

### Kế hoạch tuần tiếp theo (Tuần 4):

* Hoàn thiện toàn diện tài liệu **Báo cáo Đề tài Đồ án Thực tập (Project Proposal)** bao gồm: Tổng quan bài toán, sơ đồ kiến trúc chi tiết, bảng ước tính chi phí AWS Pricing Calculator.
* Bắt tay vào viết mã nguồn và triển khai các thành phần hạ tầng đầu tiên theo thiết kế đã được phê duyệt.
* Chuẩn bị bài thuyết trình bảo vệ ý tưởng Proposal trước Mentor và các thành viên FCAJ.
