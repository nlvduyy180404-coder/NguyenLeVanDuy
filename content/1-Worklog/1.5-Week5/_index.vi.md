---
title: "Worklog Tuần 5"
date: 2026-10-12
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Triển khai giải pháp kết nối mạng an toàn một chiều cho các máy chủ nội bộ thông qua **NAT Gateway**.
* Tối ưu hóa luồng dữ liệu và chi phí truy cập dịch vụ lưu trữ bằng **Amazon S3 Gateway Endpoint (VPC Endpoint)**.
* Khởi tạo và cấu hình tầng cơ sở dữ liệu quan hệ **Amazon RDS MySQL** chuẩn hóa bên trong Private Subnet có mã hóa **AWS KMS**.
* Kiểm thử tích hợp kết nối nội bộ (End-to-End Connectivity) giữa máy chủ ứng dụng EC2 và cơ sở dữ liệu RDS.
* Thực hiện rà soát an ninh mạng (Network Security Audit) và áp dụng quy trình kiểm soát chi phí cho các dịch vụ mạng nâng cao.

---

### Tổng quan tiến độ công việc trong tuần:

| Thứ | Công việc chính | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------------- | ------------ | --------------- | -------------- |
| **2** | **Cấu hình NAT Gateway & Private Routing:** Triển khai NAT Gateway tại Public Subnet, cấu hình Route Table cho Private Subnets kết nối Internet 1 chiều. | 12/10/2026 | 12/10/2026 | [VPC NAT Gateway Guide](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) |
| **3** | **Tối ưu hóa với S3 Gateway Endpoint:** Thiết lập VPC Endpoint tới Amazon S3, định tuyến lưu lượng qua mạng nội bộ AWS không qua Internet. | 13/10/2026 | 13/10/2026 | [AWS PrivateLink & Gateway Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/) |
| **4** | **Triển khai Database Tier (Amazon RDS):** Tạo DB Subnet Group đa vùng, khởi tạo instance MySQL trong Private Subnet kèm mã hóa lưu trữ KMS. | 14/10/2026 | 14/10/2026 | [Amazon RDS User Guide](https://docs.aws.amazon.com/rds/) |
| **5** | **Kiểm thử Kết nối Ứng dụng & Database:** Cài đặt MySQL Client trên EC2 Private, kết nối qua RDS endpoint nội bộ, khởi tạo schema và dữ liệu mẫu. | 15/10/2026 | 15/10/2026 | [Connecting to an RDS DB Instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html) |
| **6** | **Rà soát An ninh & Quản trị Chi phí:** Kiểm tra khả năng cô lập mạng (Security Audit), đo độ trễ truy vấn và dọn dẹp NAT Gateway/Elastic IP. | 16/10/2026 | 16/10/2026 | [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/) |

---

### Nhật ký kỹ thuật chi tiết từng ngày (Detailed Technical Log):

#### Thứ Hai (12/10/2026): Cấu hình NAT Gateway & Định Tuyến Private Subnet

1. **Khởi tạo NAT Gateway:**
   - Cấp phát một địa chỉ **Elastic IP (EIP)** tĩnh công khai.
   - Tạo NAT Gateway đặt tại **Public Subnet AZ1** (`10.10.1.0/24`):
     - Name: `Duy-Project-NATGW-AZ1`
     - Subnet: `Duy-Public-Subnet-AZ1`
     - Elastic IP allocation ID: `eipalloc-xxxxxx`
   - Chờ trạng thái NAT Gateway chuyển từ `Pending` $\rightarrow$ `Available`.

2. **Cập nhật Bảng định tuyến Private Route Table:**
   - Tạo bảng định tuyến riêng đặt tên `Duy-Private-RT`.
   - Thêm quy tắc định tuyến (Route Rule):
     - **Destination:** `0.0.0.0/0` (tất cả lưu lượng ra ngoài Internet)
     - **Target:** NAT Gateway (`nat-xxxxxx`)
   - Liên kết (Associate) cả 2 Private Subnets (`10.10.11.0/24` và `10.10.12.0/24`) với bảng định tuyến này.

3. **Kiểm tra kết nối Outbound từ máy ảo Private:**
   - Khởi chạy một EC2 instance thử nghiệm trong Private Subnet (không cấp Public IP).
   - Truy cập vào máy ảo thông qua cơ chế Bastion Host hoặc Session Manager.
   - Chạy lệnh kiểm tra kết nối mạng một chiều:
     ```bash
     # Kiểm tra phân giải DNS và tải bản cập nhật hệ điều hành
     sudo dnf update -y
     # Kiểm tra Public IP nhận diện từ Internet (trùng khớp với Elastic IP của NAT Gateway)
     curl https://checkip.amazonaws.com
     ```
   - Kết quả: Máy chủ nội bộ có thể tải gói cài đặt từ Internet, nhưng hoàn toàn miễn nhiễm trước mọi nỗ lực kết nối trực tiếp từ Internet vào máy.

---

#### Thứ Ba (13/10/2026): Thiết Lập S3 Gateway Endpoint (VPC Endpoint)

1. **Vấn đề cần tối ưu:**
   - Khi các máy chủ EC2 trong Private Subnet tải tệp ảnh/media từ Amazon S3, nếu đi qua NAT Gateway sẽ phát sinh phí truyền dữ liệu (**Data Processing Fee**: ~0.045 USD/GB) và tăng độ trễ mạng.
   - Giải pháp: Sử dụng **Gateway VPC Endpoint** miễn phí để định tuyến trực tiếp traffic S3 qua mạng trục nội bộ của AWS (**AWS Private Network Backbone**).

2. **Khởi tạo S3 Gateway Endpoint qua AWS CLI:**
   ```bash
   # Tạo S3 Gateway Endpoint gắn trực tiếp vào Route Table của Private Subnet
   ENDPOINT_ID=$(aws ec2 create-vpc-endpoint \
       --vpc-id $VPC_ID \
       --service-name com.amazonaws.ap-southeast-1.s3 \
       --route-table-ids $PRIVATE_RT_ID \
       --tag-specifications 'ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=Duy-Project-S3-Endpoint}]' \
       --query 'VpcEndpoint.VpcEndpointId' --output text)
   echo "Created S3 Gateway Endpoint: $ENDPOINT_ID"
   ```

3. **Kiểm tra bảng định tuyến tự động:**
   - Kiểm tra `Duy-Private-RT`, một route prefix-list mới đã tự động được thêm vào:
     - **Destination:** `pl-xxxx (com.amazonaws.ap-southeast-1.s3)`
     - **Target:** `vpce-xxxxxx`
   - Thử nghiệm lệnh sao chép dữ liệu từ EC2 Private lên S3:
     ```bash
     aws s3 cp large-data-file.tar.gz s3://duy-project-app-assets-2026/
     ```
   - Tốc độ tải lên đạt băng thông tối đa của đường truyền nội bộ mà không tốn bất kỳ byte lưu lượng nào qua NAT Gateway.

---

#### Thứ Tư (14/10/2026): Triển Khai Database Tier (Amazon RDS MySQL)

1. **Khởi tạo DB Subnet Group:**
   - Name: `duy-project-rds-subnet-group`
   - VPC: `Duy-Project-Production-VPC`
   - Add Subnets: Chọn 2 Private Subnets ở 2 AZs khác nhau (`10.10.11.0/24` tại `ap-southeast-1a` và `10.10.12.0/24` tại `ap-southeast-1b`). Điều này đảm bảo cơ sở dữ liệu sẵn sàng kích hoạt tính năng High Availability Multi-AZ khi cần.

2. **Cấu hình Cơ sở dữ liệu RDS MySQL:**
   - **Database engine:** MySQL Community 8.0.
   - **Template:** Dev/Test (chọn cấu hình tối ưu chi phí).
   - **Settings:**
     - DB instance identifier: `duy-production-mysql`
     - Master username: `admin_duy`
     - Master password: `[Managed securely in AWS Secrets Manager]`
   - **Instance configuration:**
     - Instance class: `db.t4g.micro` (2 vCPU, 1 GB RAM - kiến trúc Graviton tiết kiệm chi phí).
     - Storage: `20 GiB`, Storage type: `gp3` (General Purpose SSD).
   - **Connectivity & Networking:**
     - Virtual Private Cloud: `Duy-Project-Production-VPC`
     - DB Subnet Group: `duy-project-rds-subnet-group`
     - **Public access:** `No` (Cô lập hoàn toàn khỏi Internet).
     - VPC Security Group: Gắn `Database-SG` (Chỉ mở port `3306` cho `Web-App-SG`).
   - **Database authentication & Encryption:**
     - Password authentication.
     - **Encryption at rest:** Kích hoạt (Enabled) với khóa KMS mặc định `aws/rds`.

---

#### Thứ Năm (15/10/2026): Tích Hợp Máy Chủ Ứng Dụng & Cơ Sở Dữ Liệu Nội Bộ

1. **Cài đặt công cụ kết nối trên EC2 Private:**
   - Kết nối vào máy ảo EC2 đóng vai trò Application Server trong Private Subnet.
   - Cài đặt trình điều khiển MySQL Client:
     ```bash
     sudo dnf install -y mariadb105
     ```

2. **Thực thi kết nối thông qua Endpoint nội bộ:**
   ```bash
   # Lấy Endpoint nội bộ của RDS
   RDS_ENDPOINT="duy-production-mysql.xxxxxx.ap-southeast-1.rds.amazonaws.com"

   # Kết nối vào cơ sở dữ liệu
   mysql -h $RDS_ENDPOINT -P 3306 -u admin_duy -p
   ```

3. **Khởi tạo Schema và Nạp dữ liệu mẫu:**
   ```sql
   -- Tạo cơ sở dữ liệu chính cho dự án
   CREATE DATABASE ecom_store_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   USE ecom_store_db;

   -- Tạo bảng danh mục sản phẩm
   CREATE TABLE products (
       id INT AUTO_INCREMENT PRIMARY KEY,
       sku VARCHAR(50) NOT NULL UNIQUE,
       name VARCHAR(255) NOT NULL,
       price DECIMAL(10, 2) NOT NULL,
       stock_quantity INT DEFAULT 0,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );

   -- Chèn dữ liệu kiểm thử
   INSERT INTO products (sku, name, price, stock_quantity) VALUES
   ('AWS-SHIRT-01', 'First Cloud AI Journey T-Shirt', 19.99, 100),
   ('AWS-HOODIE-02', 'Cloud Engineer Hoodie', 49.99, 50),
   ('AWS-MUG-03', 'Architect Ceramic Mug', 12.50, 200);

   -- Kiểm tra dữ liệu
   SELECT * FROM products;
   ```
   *Kết quả:* Kết nối và các câu lệnh truy vấn diễn ra tức thì với độ trễ nội bộ (LAN latency) dưới 1 miligiây.

---

#### Thứ Sáu (16/10/2026): Rà Soát An Ninh Mạng & Tối Ưu Hóa Chi Phí

1. **Rà soát An ninh Mạng (Security Penetration Audit):**
   - Từ máy tính cá nhân ở nhà (mạng ngoài Internet), chạy lệnh kiểm tra cổng và kết nối tới Endpoint của RDS:
     ```bash
     # Thử kết nối từ mạng bên ngoài
     telnet duy-production-mysql.xxxxxx.ap-southeast-1.rds.amazonaws.com 3306
     ```
   - Kết quả: Lệnh hoàn toàn bị treo và báo lỗi `Connection timed out` sau 30 giây.
   - Xác nhận: Cổng 3306 hoàn toàn vô hình đối với mạng Internet công cộng nhờ cấu hình Private Subnet và cơ chế Security Group phòng vệ đa tầng.

2. **Quy trình Quản trị Chi phí Mạng (Network Cost Governance):**
   - **Xóa NAT Gateway sau buổi thực hành:** NAT Gateway tính phí duy trì liên tục khoảng `0.045 USD/giờ` (~32 USD/tháng) ngay cả khi không có lưu lượng truyền qua.
   - Quy trình teardown an toàn:
     ```bash
     # Xóa NAT Gateway
     aws ec2 delete-nat-gateway --nat-gateway-id $NAT_GW_ID

     # Chờ chuyển trạng thái Deleted, giải phóng Elastic IP để tránh phí giữ IP rảnh rỗi ($0.005/h)
     aws ec2 release-address --allocation-id $EIP_ALLOC_ID
     ```
   - Tạm dừng (Stop) RDS Database instance khi không làm bài tập để bảo toàn credit.

---

### Kết quả đạt được tuần 5:

* **Hoàn thiện Tầng Mạng Bảo mật (Private Networking):** Thiết lập thành công mô hình mạng chuẩn doanh nghiệp với NAT Gateway cung cấp kết nối Internet 1 chiều an toàn cho các máy chủ nội bộ.
* **Tối ưu hóa Tuyến đường S3 với VPC Endpoint:** Triển khai S3 Gateway Endpoint giúp tiết kiệm chi phí băng thông NAT Gateway và tăng tốc độ truyền tải dữ liệu nội bộ.
* **Cơ sở dữ liệu Cô lập Tuyệt đối:** Khởi tạo thành công Amazon RDS MySQL trong Private Subnet Group, kích hoạt mã hóa tĩnh AWS KMS, hoàn toàn chặn đứng mọi truy cập trái phép từ Internet.
* **Tích hợp End-to-End thành công:** Kết nối máy chủ ứng dụng EC2 với cơ sở dữ liệu RDS qua Endpoint nội bộ, khởi tạo schema dữ liệu kiểm thử với độ trễ truy vấn cực thấp (< 1ms).
* **Quản trị Tài chính Kỷ luật:** Nắm vững cơ chế tính phí của NAT Gateway và Elastic IP, chủ động thu hồi tài nguyên sau lab để bảo vệ hạn mức credit tài khoản.

---

### Khó khăn gặp phải & Cách giải quyết:

| Vấn đề kỹ thuật gặp phải | Nguyên nhân gốc rễ | Cách khắc phục đã thực hiện |
| ------------------------ | ------------------ | --------------------------- |
| **Máy ảo trong Private Subnet không kết nối được Internet sau khi tạo NAT Gateway** | Bảng định tuyến của Private Subnet vẫn chưa được thêm route mặc định `0.0.0.0/0` trỏ tới NAT Gateway ID mới tạo. | Truy cập Route Table của Private Subnet (`Duy-Private-RT`), thêm tuyến đường `0.0.0.0/0` trỏ đến `nat-xxxxxx` và lưu lại cấu hình. |
| **Lỗi `Access Denied` khi dùng AWS CLI trên EC2 Private gọi tới S3 qua Endpoint** | EC2 instance chưa được gắn **IAM Instance Profile / Role** có quyền hạn tương tác với Amazon S3. | Tạo một IAM Role có policy `AmazonS3FullAccess` (hoặc policy thu hẹp phạm vi bucket) và gán role này vào EC2 instance thông qua menu **Actions $\rightarrow$ Security $\rightarrow$ Modify IAM role**. |
| **Không thể xóa VPC vì báo lỗi tài nguyên phụ thuộc (DependencyViolation)** | NAT Gateway vẫn đang trong quá trình xóa (trạng thái `deleting`), hoặc còn Network Interface (ENI) của RDS chưa được giải phóng. | Kiên nhẫn chờ NAT Gateway chuyển hẳn sang trạng thái `deleted`, xóa DB Subnet Group và xóa các Network Interface liên quan trước khi thực hiện xóa VPC. |

---

### Bài học kinh nghiệm rút ra:

1. **Nguyên tắc Phòng thủ Chiều sâu (Defense-in-Depth for Databases):**
   * Bảo vệ dữ liệu nhạy cảm không chỉ dựa vào username/password mà phải tạo 3 lớp phòng vệ: (1) Đặt trong Private Subnet không có Public IP, (2) Security Group chỉ cho phép IP nội bộ của App Server, (3) Mã hóa lưu trữ bằng AWS KMS.
2. **Chi phí ẩn của các dịch vụ mạng (Network Hidden Costs):**
   * NAT Gateway và Elastic IP chưa gắn là hai nguyên nhân hàng đầu gây thâm hụt tài khoản của thực tập sinh. Luôn phải giải phóng Elastic IP ngay khi xóa NAT Gateway.
3. **Lợi ích kiến trúc của VPC Endpoints:**
   * Tận dụng VPC Endpoints không chỉ giúp nâng cao tính bảo mật (dữ liệu không bao giờ rời khỏi mạng nội bộ AWS) mà còn là chiến lược tối ưu hóa chi phí hàng đầu trong các bài thi chứng chỉ AWS Solutions Architect.

---

### Kế hoạch tuần tiếp theo (Tuần 6):

* Triển khai kết nối toàn diện 3 tầng: Tích hợp Application Load Balancer (Tầng 1) $\rightarrow$ Auto Scaling EC2 (Tầng 2) $\rightarrow$ RDS MySQL (Tầng 3).
* Xây dựng ứng dụng web demo hoàn chỉnh (hiển thị danh mục sản phẩm từ Database và tải hình ảnh từ S3).
* Kiểm thử khả năng chịu lỗi và tính ổn định của toàn bộ chuỗi hệ thống (Full-stack End-to-End Resilience Test).
