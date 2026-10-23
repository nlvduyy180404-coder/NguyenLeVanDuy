---
title: "Worklog Tuần 2"
date: 2026-09-21
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Thành thạo công cụ dòng lệnh **AWS CLI** để truy vấn và quản lý tài nguyên đám mây trực tiếp từ terminal.
* Hiểu sâu về dịch vụ lưu trữ đối tượng **Amazon S3** (Object Storage) và lưu trữ khối **Amazon EBS** (Block Storage).
* Tự tay thiết kế và triển khai kiến trúc mạng nền tảng **Amazon VPC** (Custom VPC, Public/Private Subnets, Internet Gateway, Route Tables).
* Nâng cao kỹ năng quản trị máy ảo **Amazon EC2**: tự động hóa triển khai Web Server bằng **User Data script** và mount ổ đĩa cứng EBS phụ.
* Hoàn thành trọn vẹn các mốc thử thách Free Tier (Amazon RDS, AWS Budgets) để nhận tối đa gói **$200 Credit** phục vụ toàn bộ kỳ thực tập.

---

### Tổng quan tiến độ công việc trong tuần:

| Thứ | Công việc chính | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------------- | ------------ | --------------- | -------------- |
| **2** | **Cài đặt & Quản trị danh tính AWS CLI:** Tạo Access Key IAM, cấu hình CLI profile, kiểm tra quyền hạn qua terminal. | 21/09/2026 | 21/09/2026 | [AWS CLI User Guide](https://docs.aws.amazon.com/cli/) |
| **3** | **Làm chủ lưu trữ Amazon S3:** Tạo Globally Unique Bucket, quản lý object qua CLI, tìm hiểu Block Public Access & Bucket Policy. | 22/09/2026 | 22/09/2026 | [S3 Documentation](https://docs.aws.amazon.com/s3/) |
| **4** | **Thiết kế mạng Amazon VPC:** Hoạch định dải mạng CIDR, cấu hình Public/Private Subnet, Internet Gateway và Route Table. | 23/09/2026 | 23/09/2026 | [VPC User Guide](https://docs.aws.amazon.com/vpc/) |
| **5** | **EC2 Bootstrap & Lưu trữ EBS:** Khởi chạy EC2 với User Data tự động dựng Web Server; tạo và mount phân vùng ổ cứng EBS mới. | 24/09/2026 | 24/09/2026 | [EC2 User Guide](https://docs.aws.amazon.com/ec2/) |
| **6** | **Hoàn thành mốc Free Tier ($200 Credit):** Khởi tạo database quan hệ Amazon RDS và thiết lập cảnh báo ngân sách với AWS Budgets. | 25/09/2026 | 25/09/2026 | [AWS Free Tier 2025](https://aws.amazon.com/free/) |

---

### Nhật ký kỹ thuật chi tiết từng ngày (Detailed Technical Log):

#### Thứ Hai (21/09/2026): Cài đặt & Cấu hình AWS CLI

1. **Tạo Access Key cho IAM User:**
   - Đăng nhập vào IAM Console bằng tài khoản IAM User đã tạo ở Tuần 1.
   - Truy cập **Security credentials** $\rightarrow$ chọn **Create access key** $\rightarrow$ chọn mục đích sử dụng là *Command Line Interface (CLI)*.
   - Lưu trữ an toàn cặp khóa `Access Key ID` và `Secret Access Key` (không chia sẻ công khai).

2. **Cấu hình trên Terminal:**
   ```bash
   # Khởi chạy wizard cấu hình của AWS CLI
   aws configure
   ```
   - Nhập thông số:
     - `AWS Access Key ID`: `AKIA...`
     - `AWS Secret Access Key`: `wJalrXUtnFEMI...`
     - `Default region name`: `ap-southeast-1` (Singapore)
     - `Default output format`: `json`

3. **Kiểm tra danh tính và kết nối:**
   ```bash
   aws sts get-caller-identity
   ```
   *Kết quả trả về định dạng JSON:*
   ```json
   {
       "UserId": "AIDASAMPLEUSERID2026",
       "Account": "123456789012",
       "Arn": "arn:aws:iam::123456789012:user/duy-iam-user"
   }
   ```

---

#### Thứ Ba (22/09/2026): Quản trị Lưu trữ Đối tượng với Amazon S3

1. **Khởi tạo và quản lý S3 Bucket qua dòng lệnh:**
   ```bash
   # Tạo một bucket mới với tên duy nhất toàn cầu
   aws s3 mb s3://fcj-duy-storage-bucket-2026 --region ap-southeast-1

   # Tải file lên bucket
   aws s3 cp sample-file.txt s3://fcj-duy-storage-bucket-2026/

   # Liệt kê các object bên trong bucket
   aws s3 ls s3://fcj-duy-storage-bucket-2026/
   ```

2. **Cấu hình Quyền riêng tư (S3 Block Public Access & Bucket Policy):**
   - Mặc định, Amazon S3 kích hoạt cơ chế phòng vệ **Block all public access** để ngăn chặn vô tình lộ dữ liệu nhạy cảm ra ngoài Internet.
   - Khi cần public một số file tài liệu cho bài báo cáo, thực hiện tắt Block Public Access và gắn **Bucket Policy** cấp quyền đọc cụ thể:
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

#### Thứ Tư (23/09/2026): Xây dựng Kiến trúc Mạng Amazon VPC từ đầu

1. **Bảng quy hoạch dải mạng (Network Addressing Plan):**

| Thành phần | Tên tài nguyên | Dải CIDR Block | Ghi chú |
| ---------- | -------------- | -------------- | ------- |
| **VPC** | `Duy-Custom-VPC` | `10.0.0.0/16` | Hỗ trợ tối đa 65,536 địa chỉ IP |
| **Public Subnet** | `Duy-Public-Subnet-AZ1` | `10.0.1.0/24` | Gắn Route Table ra Internet Gateway (IGW) |
| **Private Subnet** | `Duy-Private-Subnet-AZ1` | `10.0.2.0/24` | Không có route ra IGW, dành cho Database |
| **Internet Gateway** | `Duy-VPC-IGW` | - | Cổng kết nối giữa VPC và Internet |

2. **Cấu hình Route Table cho Public Subnet:**
   - Tạo Route Table đặt tên `Duy-Public-RT`.
   - Thêm quy tắc định tuyến (Route Rule):
     - **Destination:** `0.0.0.0/0` (tất cả lưu lượng ra ngoài)
     - **Target:** Internet Gateway (`Duy-VPC-IGW`)
   - Liên kết (Associate) Subnet `10.0.1.0/24` với bảng định tuyến này.

---

#### Thứ Năm (24/09/2026): Tự động hóa EC2 với User Data & Mount ổ cứng EBS

1. **Khởi chạy máy chủ ảo kèm User Data script:**
   - Trong quá trình cấu hình Launch Instance (Amazon Linux 2023), tại mục **Advanced details $\rightarrow$ User data**, thêm script tự động hóa:
   ```bash
   #!/bin/bash
   # Cập nhật các gói phần mềm
   dnf update -y

   # Cài đặt Apache Web Server
   dnf install -y httpd

   # Kích hoạt và khởi chạy service
   systemctl start httpd
   systemctl enable httpd

   # Tạo trang web kiểm tra mặc định
   echo "<h1>Welcome to Nguyen Le Van Duy's Web Server on AWS!</h1>" > /var/www/html/index.html
   ```
   - Nhờ User Data, ngay khi máy chủ chuyển sang trạng thái `Running`, truy cập vào Public IP của máy qua trình duyệt sẽ thấy ngay trang Web hiển thị mà không cần phải SSH vào gõ lệnh thủ công.

2. **Đính kèm và Mount ổ cứng EBS Volume:**
   - Tạo thêm một EBS Volume dung lượng 5GB (loại `gp3`) trong cùng Availability Zone với EC2.
   - Chọn Action **Attach Volume** vào máy ảo (nhận diện dưới tên `/dev/sdf` hoặc `/dev/nvme1n1`).
   - SSH vào máy ảo để định dạng và gắn ổ:
   ```bash
   # Kiểm tra danh sách ổ đĩa
   lsblk

   # Định dạng hệ thống tệp ext4 cho ổ đĩa mới
   sudo mkfs -t ext4 /dev/xvdf

   # Tạo thư mục mount và gắn ổ đĩa
   sudo mkdir /data
   sudo mount /dev/xvdf /data

   # Kiểm tra dung lượng sau khi mount thành công
   df -h /data
   ```

---

#### Thứ Sáu (25/09/2026): Hoàn thành thử thách Free Tier ($200 Credit)

1. **Triển khai Cơ sở dữ liệu Amazon RDS:**
   - Dịch vụ: **Amazon RDS** $\rightarrow$ Create database.
   - Engine: **MySQL** (Community Edition).
   - Template: **Free Tier** (chọn `db.t3.micro` hoặc `db.t4g.micro`, 20GB gp2 Storage).
   - Đặt instance trong Private Subnet của VPC để đảm bảo an toàn, chỉ cho phép EC2 Web Server kết nối qua cổng 3306.
   - Sau khi hoàn thành kiểm tra kết nối, thực hiện **Delete Database** (tắt snapshot tự động) để tránh hao phí tài nguyên.

2. **Thiết lập Cảnh báo Ngân sách AWS Budgets:**
   - Truy cập dịch vụ **AWS Billing and Cost Management** $\rightarrow$ **Budgets**.
   - Tạo một **Cost Budget**:
     - Định mức: **10 USD/tháng**.
     - Cấu hình cảnh báo: Gửi email cảnh báo tự động về `nlvduyy180404@gmail.com` khi chi phí thực tế đạt 80% (8 USD) hoặc chi phí dự báo (forecasted) chạm 100%.
   - Hoàn thành đầy đủ 5 nhiệm vụ để tài khoản nhận đủ **$200 promotional credit**.

---

### Kết quả đạt được tuần 2:

* **Kỹ năng CLI:** Sử dụng thành thạo các câu lệnh cơ bản của AWS CLI để truy vấn thông tin, cấu hình profile và thao tác với dịch vụ mà không cần dùng chuột trên giao diện web.
* **Lưu trữ Cloud:** Nắm vững sự khác nhau giữa Object Storage (S3 - lưu trữ file, static assets, backup) và Block Storage (EBS - ổ đĩa ảo cho hệ điều hành và cơ sở dữ liệu).
* **Kiến trúc Mạng độc lập:** Tự tay thiết kế và cấu hình thành công một hệ thống mạng Custom VPC độc lập với đầy đủ Public/Private Subnet và định tuyến Internet Gateway.
* **Tự động hóa Bootstrap:** Triển khai thành công Web Server tự động thông qua User Data script, tiết kiệm thời gian vận hành hạ tầng.
* **Tối ưu Chi phí & Bảo mật:** Hoàn thành trọn vẹn thử thách Free Tier 2025, nhận đủ $200 credit và thiết lập hàng rào cảnh báo ngân sách tự động bảo vệ tài chính an toàn.

---

### Khó khăn gặp phải & Cách giải quyết:

| Vấn đề kỹ thuật gặp phải | Nguyên nhân gốc rễ | Cách khắc phục đã thực hiện |
| ------------------------ | ------------------ | --------------------------- |
| **Lỗi 403 Forbidden khi tải object từ link S3 công khai** | Tính năng *S3 Block Public Access* đang được kích hoạt ở cấp độ tài khoản và bucket để bảo vệ dữ liệu. | Tắt tính năng Block Public Access đối với bucket cần chia sẻ, đồng thời cấu hình thêm **Bucket Policy** với quyền `s3:GetObject` cho công chúng (`*`). |
| **Ổ đĩa EBS Volume đã đính kèm (attached) nhưng không hiển thị dung lượng** | Ổ cứng dạng khối (block device) mới chưa có hệ thống tệp (filesystem) và chưa được mount vào cây thư mục của Linux. | Chạy lệnh `lsblk` để xác định đúng tên thiết bị, thực thi lệnh `sudo mkfs -t ext4` để format phân vùng và dùng `sudo mount` đưa ổ đĩa vào thư mục `/data`. |
| **Quá trình khởi tạo Amazon RDS diễn ra rất lâu và không kết nối được** | Việc provision hệ thống cơ sở dữ liệu managed tốn từ 5 đến 10 phút; Security Group mặc định của RDS chặn toàn bộ inbound traffic. | Kiên nhẫn chờ trạng thái chuyển sang `Available`. Cấu hình Security Group của RDS cho phép port 3306 với Source là Security Group của EC2 Instance. Sau khi kiểm tra xong, tiến hành xóa ngay instance để giải phóng tài nguyên. |

---

### Bài học kinh nghiệm rút ra:

1. **Quản trị Access Key an toàn:**
   * Không bao giờ lưu cứng thông tin Access Key / Secret Key vào source code hoặc đẩy lên GitHub công khai.
   * Ưu tiên sử dụng IAM Roles thay cho Access Key khi các dịch vụ AWS cần giao tiếp với nhau (ví dụ: EC2 gọi S3).
2. **Nguyên tắc phân tách mạng Public & Private:**
   * Chỉ đưa những thành phần cần nhận truy cập trực tiếp từ người dùng (như Web Server, Load Balancer) vào Public Subnet.
   * Luôn đặt cơ sở dữ liệu (RDS) và các hệ thống xử lý nội bộ trong Private Subnet để ngăn chặn tối đa nguy cơ tấn công mạng.
3. **Tư duy Tự động hóa Hạ tầng (Infrastructure as Code & Automation):**
   * Sử dụng User Data script chứng minh việc tự động hóa giúp loại bỏ sai sót do thao tác thủ công của con người và tăng tốc độ triển khai máy chủ lên gấp nhiều lần.

---

### Kế hoạch tuần tiếp theo (Tuần 3):

* Tìm hiểu về kiến trúc sẵn sàng cao (High Availability): Cơ chế cân bằng tải với **Application Load Balancer (ALB)**.
* Tìm hiểu cơ chế tự động co giãn theo tải thực tế: **Auto Scaling Group (ASG)**.
* Thực hành kết hợp VPC + ALB + EC2 Auto Scaling để xây dựng một hệ thống web có khả năng tự phục hồi khi có sự cố.
