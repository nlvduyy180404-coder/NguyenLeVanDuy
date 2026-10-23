---
title: "Worklog Tuần 4"
date: 2026-10-05
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Hoàn thiện toàn diện tài liệu Báo cáo Đề xuất Đồ án (**Project Proposal**) cho đề tài xây dựng kiến trúc ứng dụng Web 3 lớp có khả năng chịu tải và độ sẵn sàng cao (**3-Tier Resilient Web Architecture**).
* Xây dựng sơ đồ kiến trúc hệ thống chi tiết (High-Level Architecture Diagram) thể hiện rõ luồng dữ liệu và ranh giới bảo mật đa vùng (Multi-AZ).
* Sử dụng công cụ **AWS Pricing Calculator** để hoạch định, bóc tách và ước tính chi phí vận hành hàng tháng của toàn bộ hệ thống.
* Đánh giá ma trận rủi ro và xây dựng chiến lược sao lưu, phục hồi sự cố (Disaster Recovery).
* Bắt đầu triển khai các khối tài nguyên hạ tầng nền tảng (Foundation Infrastructure) đầu tiên trên AWS theo đúng thiết kế đã được phê duyệt.

---

### Tổng quan tiến độ công việc trong tuần:

| Thứ | Công việc chính | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------------- | ------------ | --------------- | -------------- |
| **2** | **Soạn thảo Văn bản Proposal:** Mô tả bài toán thực tế, xác định mục tiêu độ sẵn sàng (SLA 99.9%) và các yêu cầu phi chức năng. | 05/10/2026 | 05/10/2026 | Tài liệu hướng dẫn Proposal FCAJ |
| **3** | **Thiết kế Sơ đồ Kiến trúc Chi tiết:** Vẽ mô hình phân tầng 3-Tier (Ingress Tier, Compute Tier, Data Tier) và hoạch định ranh giới Security Group. | 06/10/2026 | 06/10/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) |
| **4** | **Tính toán Ngân sách (AWS Pricing Calculator):** Bóc tách chi phí định lượng từng dịch vụ (ALB, EC2, RDS Multi-AZ, S3, CloudFront). | 07/10/2026 | 07/10/2026 | [AWS Pricing Calculator](https://calculator.aws/) |
| **5** | **Đánh giá Rủi ro & Chiến lược Phục hồi:** Phân tích ma trận rủi ro (RPO/RTO), hoàn thiện tài liệu đồ án và rà soát cùng nhóm. | 08/10/2026 | 08/10/2026 | [AWS Well-Architected Reliability](https://docs.aws.amazon.com/wellarchitected/) |
| **6** | **Khởi tạo Hạ tầng Nền tảng (Foundation Setup):** Triển khai Production VPC (4 Subnets qua 2 AZs), gắn Internet Gateway và tạo S3 Assets Bucket. | 09/10/2026 | 09/10/2026 | [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/) |

---

### Nhật ký kỹ thuật chi tiết từng ngày (Detailed Technical Log):

#### Thứ Hai (05/10/2026): Soạn Thảo Văn Bản & Xác Định Bài Toán Đề Tài Proposal

1. **Bối cảnh và bài toán thực tế của đề tài:**
   - Các ứng dụng thương mại điện tử hoặc cổng thông tin doanh nghiệp vừa và nhỏ thường đối mặt với nguy cơ sập máy chủ khi lượng người dùng tăng đột biến trong các chiến dịch khuyến mãi hoặc sự kiện truyền thông.
   - Mô hình máy chủ đơn lẻ (Single Instance) tiềm ẩn rủi ro lỗi điểm đơn (SPOF): Khi trung tâm dữ liệu gặp sự cố mất điện/mạng, toàn bộ hoạt động kinh doanh bị tê liệt.
2. **Xác định các chỉ số mục tiêu kỹ thuật (SLO / SLA Targets):**
   - **Tính sẵn sàng (Availability):** Hướng đến mức **99.9% uptime** (thời gian ngừng hoạt động tối đa cho phép dưới 8.76 giờ/năm).
   - **Khả năng co giãn đàn hồi (Elastic Scalability):** Hệ thống có khả năng tự động tăng số lượng máy chủ từ 2 lên tối đa 6 máy trong vòng 3 phút khi CPU trung bình vượt quá 60%.
   - **Mục tiêu phục hồi:**
     - *Recovery Point Objective (RPO):* Dưới 5 phút (dữ liệu giao dịch được bảo vệ liên tục thông qua sao lưu tự động của RDS Multi-AZ).
     - *Recovery Time Objective (RTO):* Dưới 2 phút (chuyển đổi dự phòng tự động giữa các Availability Zones mà không cần can thiệp thủ công).

---

#### Thứ Ba (06/10/2026): Thiết Kế Sơ Đồ Kiến Trúc Hệ Thống Chi Tiết

1. **Phân tầng kiến trúc giải pháp 3 lớp (3-Tier Architecture):**
   - **Tầng 1 - Ingress & Edge Tier:**
     - *Amazon Route 53:* Tiếp nhận phân giải DNS tên miền và chuyển hướng người dùng thông minh.
     - *Amazon CloudFront:* Đóng vai trò CDN toàn cầu, lưu trữ đệm (caching) các tệp tĩnh (HTML, CSS, JavaScript, hình ảnh sản phẩm) tại các Edge Location, giảm tải 70% truy vấn trực tiếp về máy chủ gốc.
     - *Application Load Balancer (ALB):* Đặt tại Public Subnets qua 2 Availability Zones, phân phối tải HTTP/HTTPS đồng đều đến các máy chủ ứng dụng.
   - **Tầng 2 - Application & Compute Tier:**
     - *Auto Scaling Group (ASG):* Quản lý các máy ảo Amazon EC2 (Amazon Linux 2023) đặt trong các **Private App Subnets** (cô lập an toàn, không có Public IP), tự động co giãn theo tải thực tế.
     - *NAT Gateway:* Cung cấp kết nối 1 chiều ra Internet để các máy ảo trong Private Subnet có thể tải bản vá bảo mật và cập nhật phần mềm.
   - **Tầng 3 - Database & Storage Tier:**
     - *Amazon RDS for MySQL (Multi-AZ Deployment):* Máy chủ Database chính (Primary) đặt tại Private Subnet của AZ-1, tự động sao chép đồng bộ (Synchronous Replication) sang máy chủ dự phòng (Standby) tại AZ-2.
     - *Amazon S3 Bucket:* Lưu trữ dữ liệu tệp đính kèm và media với độ bền 99.999999999% (11 số 9).

2. **Ranh giới bảo mật phân tầng (Security Group Chaining Matrix):**

| Security Group | Inbound Rules cho phép | Nguồn truy cập (Source) | Mục đích bảo vệ |
| -------------- | ---------------------- | ----------------------- | --------------- |
| `ALB-SG` | Port `80` (HTTP) / Port `443` (HTTPS) | `0.0.0.0/0` (Internet) | Tiếp nhận lưu lượng người dùng công khai |
| `Web-App-SG` | Port `80` (HTTP) | Chỉ từ `ALB-SG` ID | Ngăn chặn hoàn toàn việc bypass Load Balancer |
| `Database-SG` | Port `3306` (MySQL) | Chỉ từ `Web-App-SG` ID | Chỉ cho phép tầng ứng dụng truy vấn cơ sở dữ liệu |

---

#### Thứ Tư (07/10/2026): Bóc Tách Chi Phí với AWS Pricing Calculator

1. **Lập bảng tính toán chi phí vận hành ước tính:**
   - Sử dụng công cụ chính thức **AWS Pricing Calculator** để mô phỏng chi phí hoạt động cho một tháng tại Region Singapore (`ap-southeast-1`):

| Dịch vụ AWS | Cấu hình định lượng chi tiết | Chi phí ước tính (USD/tháng) |
| ----------- | ---------------------------- | ---------------------------- |
| **Amazon EC2** | 2 instance `t3.micro` (2 vCPU, 1GB RAM) chạy 24/7 (khi không scale out) | ~15.18 USD |
| **Amazon EBS** | 2 ổ đĩa `gp3` dung lượng 20GB/máy (tổng 40GB) | ~3.20 USD |
| **Application Load Balancer** | 1 ALB xử lý ~10GB dữ liệu và 1 LCU | ~16.50 USD |
| **Amazon RDS (Multi-AZ)** | `db.t4g.micro` (2 vCPU, 1GB RAM), 20GB gp3 General Purpose Storage | ~26.40 USD |
| **Amazon S3 Standard** | 10GB lưu trữ đối tượng, 50,000 lượt GET, 5,000 lượt PUT | ~0.45 USD |
| **Amazon CloudFront** | 50GB truyền dữ liệu ra Internet (Data Transfer Out) | ~6.00 USD |
| **Data Transfer & Route 53** | 1 Hosted Zone + Lưu lượng mạng liên vùng nội bộ | ~1.50 USD |
| **Tổng cộng hàng tháng** | *(Chưa áp dụng ưu đãi Free Tier / Credit)* | **~69.23 USD / tháng** |

2. **Chiến lược tối ưu hóa cho môi trường thử nghiệm:**
   - Nhờ có gói **$200 Promotional Credit** đã đạt được từ các tuần trước, hệ thống hoàn toàn có thể chạy thực nghiệm liên tục trong hơn 2.5 tháng mà không phát sinh bất kỳ khoản phí thực tế nào vào thẻ ngân hàng.
   - Với môi trường lab hằng ngày: Chỉ bật RDS Single-AZ và tắt ALB khi không kiểm thử để giảm 60% chi phí.

---

#### Thứ Năm (08/10/2026): Phân Tích Ma Trận Rủi Ro & Kế Hoạch Dự Phòng

1. **Bảng đánh giá ma trận rủi ro (Risk Assessment):**

| Rủi ro tiềm ẩn | Mức độ ảnh hưởng | Xác suất xảy ra | Giải pháp kiểm soát & Giảm thiểu |
| -------------- | ---------------- | --------------- | -------------------------------- |
| **Sự cố ngừng hoạt động tại một Availability Zone (AZ Outage)** | Rất cao | Thấp | Thiết kế Multi-AZ: ALB và ASG tự động phân bổ tải sang AZ còn lại; RDS tự động failover sang Standby instance trong vòng 60 giây. |
| **Lưu lượng truy cập tăng đột biến làm quá tải máy chủ (Traffic Spike)** | Cao | Trung bình | Auto Scaling Group tự động kích hoạt scale-out thêm máy ảo khi ngưỡng CPU > 60%; CloudFront CDN hấp thụ phần lớn tải các tệp tĩnh. |
| **Rò rỉ hoặc mất mát dữ liệu cơ sở dữ liệu (Data Loss)** | Nghiêm trọng | Rất thấp | Kích hoạt tính năng **Automated Backups** của Amazon RDS với chu kỳ lưu giữ snapshot hàng ngày; kích hoạt **S3 Versioning** chống xóa nhầm. |
| **Vượt ngân sách ngoài dự kiến (Cost Overrun)** | Trung bình | Thấp | Cấu hình **AWS Budgets** cảnh báo qua email ở các mốc 50%, 80% và 100% ngân sách đề tài. |

---

#### Thứ Sáu (09/10/2026): Bắt Đầu Khởi Tạo Hạ Tầng Nền Tảng (Foundation Setup)

1. **Khởi tạo Production VPC thông qua AWS CLI:**
   ```bash
   # 1. Tạo VPC chính cho dự án với dải mạng 10.10.0.0/16
   VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/16 \
       --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Duy-Project-Production-VPC}]' \
       --query 'Vpc.VpcId' --output text)
   echo "Created Production VPC: $VPC_ID"

   # 2. Bật DNS Hostnames cho VPC
   aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames "{\"Value\":true}"

   # 3. Tạo Internet Gateway và đính kèm vào VPC
   IGW_ID=$(aws ec2 create-internet-gateway \
       --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Duy-Project-IGW}]' \
       --query 'InternetGateway.InternetGatewayId' --output text)
   aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID
   ```

2. **Quy hoạch 4 Subnets phân bố trên 2 Availability Zones (`ap-southeast-1a` & `ap-southeast-1b`):**
   - **Public Subnet AZ1:** `10.10.1.0/24` (Dành cho ALB Node A)
   - **Public Subnet AZ2:** `10.10.2.0/24` (Dành cho ALB Node B)
   - **Private Subnet AZ1:** `10.10.11.0/24` (Dành cho Web-App EC2 & RDS Primary)
   - **Private Subnet AZ2:** `10.10.12.0/24` (Dành cho Web-App EC2 & RDS Standby)

3. **Tạo S3 Assets Bucket với mã hóa an toàn:**
   ```bash
   # Tạo S3 bucket lưu trữ media của dự án
   aws s3 mb s3://duy-project-app-assets-2026 --region ap-southeast-1

   # Bật tính năng quản lý phiên bản (Versioning) để phòng ngừa lỗi ghi đè/xóa nhầm
   aws s3api put-bucket-versioning \
       --bucket duy-project-app-assets-2026 \
       --versioning-configuration Status=Enabled
   ```

---

### Kết quả đạt được tuần 4:

* **Tài liệu Proposal hoàn chỉnh:** Hoàn thành trọn vẹn văn bản Báo cáo Đề xuất Đồ án với đầy đủ cấu trúc: Đặt vấn đề, Yêu cầu kỹ thuật, Thiết kế giải pháp 3-Tier, Kế hoạch triển khai và Dự toán ngân sách.
* **Bản vẽ Kiến trúc chuẩn AWS:** Thể hiện trực quan mô hình bảo mật nhiều lớp, cơ chế phân bổ tải đa vùng (Multi-AZ) và ranh giới Security Group Chaining.
* **Bóc tách Chi phí Định lượng:** Nắm vững cách sử dụng **AWS Pricing Calculator**, dự toán chi phí vận hành ở mức ~69 USD/tháng và lên phương án tận dụng triệt để gói $200 Credit.
* **Khởi tạo thành công Hạ tầng Nền móng:** Triển khai hoàn tất Production VPC với 4 Subnets phân chia khoa học, gắn Internet Gateway và cấu hình S3 Bucket có bật Versioning.
* **Sẵn sàng triển khai giai đoạn 2:** Nhóm đã thống nhất được toàn bộ kế hoạch để bước vào giai đoạn cài đặt dịch vụ và cấu hình chi tiết ở Tuần 5.

---

### Khó khăn gặp phải & Cách giải quyết:

| Vấn đề kỹ thuật gặp phải | Nguyên nhân gốc rễ | Cách khắc phục đã thực hiện |
| ------------------------ | ------------------ | --------------------------- |
| **Chi phí RDS Multi-AZ trong Pricing Calculator cao hơn dự kiến ban đầu** | Tùy chọn Multi-AZ tự động nhân đôi chi phí tài nguyên máy tính (Compute) và nhân đôi dung lượng ổ đĩa lưu trữ cho node Standby dự phòng. | Điều chỉnh kế hoạch triển khai: Trong giai đoạn phát triển và kiểm thử ở các tuần đầu, nhóm sẽ sử dụng RDS Single-AZ (`db.t4g.micro`) để tiết kiệm chi phí; chỉ kích hoạt Multi-AZ khi tiến hành bài kiểm thử chuyển đổi dự phòng (Failover Test) ở giai đoạn nghiệm thu. |
| **Xung đột dải mạng CIDR khi kết nối các thành phần VPC** | Phân bổ số lượng IP cho Subnet chưa tối ưu hoặc thiếu subnet chuyên biệt cho Database. | Thiết kế lại bảng phân bổ mạng: Sử dụng subnet mask `/24` (251 IP khả dụng cho mỗi subnet), tách biệt hoàn toàn Public Subnet (cho Load Balancer) và Private Subnet (cho máy chủ ứng dụng và cơ sở dữ liệu). |
| **Độ trễ và cấu hình CORS khi CloudFront kết nối tới S3 Bucket** | S3 Bucket mới tạo mặc định chặn các truy vấn Cross-Origin Resource Sharing (CORS) từ domain khác. | Thêm cấu hình CORS Policy bằng JSON cho S3 Bucket cho phép các method `GET`, `HEAD` từ nguồn tên miền của CloudFront. |

---

### Bài học kinh nghiệm rút ra:

1. **Giá trị của việc Hoạch định Kiến trúc trước khi viết Code:**
   * Việc dành thời gian xây dựng sơ đồ phân tầng và tính toán chi phí trước giúp tránh được các sai lầm tốn kém (như việc phải đập đi tạo lại toàn bộ mạng VPC vì sai dải CIDR).
2. **Kỹ thuật tối ưu hóa chi phí (Cost-Performance Balance):**
   * Trong thực tế, không nhất thiết phải bật cấu hình cao cấp nhất ngay từ đầu. Cần hiểu rõ sự đánh đổi giữa chi phí (Cost) và mức độ sẵn sàng (Availability) ở từng môi trường phát triển (Dev) vs thực tế (Production).
3. **Quản lý hạ tầng bằng mã lệnh (Infrastructure as Code Preparation):**
   * Việc sử dụng AWS CLI để tạo VPC và Subnet giúp ghi lại toàn bộ thông số cấu hình một cách rõ ràng, tạo tiền đề thuận lợi cho việc chuyển đổi sang CloudFormation/Terraform sau này.

---

### Kế hoạch tuần tiếp theo (Tuần 5):

* Triển khai cấu hình sâu cho tầng ứng dụng (Compute Tier): Cấu hình bảo mật nâng cao cho EC2 trong Private Subnets thông qua **NAT Gateway** hoặc **VPC Endpoints**.
* Khởi tạo cơ sở dữ liệu **Amazon RDS MySQL** bên trong Private Data Subnet và cấu hình cơ chế mã hóa dữ liệu tĩnh (Encryption at Rest với AWS KMS).
* Tiến hành kết nối thử nghiệm giữa máy chủ ứng dụng và cơ sở dữ liệu trong môi trường mạng cô lập nội bộ.
