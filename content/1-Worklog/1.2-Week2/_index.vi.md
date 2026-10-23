---
title: "Worklog Tuần 2"
date: 2026-09-21
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Thành thạo công cụ dòng lệnh AWS CLI để tương tác với tài nguyên đám mây.
* Hiểu sâu về dịch vụ lưu trữ đối tượng Amazon S3 và lưu trữ khối Amazon EBS.
* Xây dựng kiến trúc mạng nền tảng với Amazon VPC (Subnet, Internet Gateway, Route Table).
* Tự động hóa khởi chạy Web Server trên EC2 bằng User Data script.
* Hoàn thành trọn vẹn các nhiệm vụ Free Tier (Amazon RDS, AWS Budgets) để nhận tối đa $200 Credit thưởng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | - **Cài đặt & Cấu hình AWS CLI:** <br>&emsp; + Tạo Access Key & Secret Key an toàn cho IAM User <br>&emsp; + Cài AWS CLI trên máy và chạy `aws configure` (Region, Output format) <br>&emsp; + Kiểm tra danh tính và quyền bằng lệnh `aws sts get-caller-identity` | 21/09/2026 | 21/09/2026 | <https://docs.aws.amazon.com/cli/> |
| 3   | - **Tìm hiểu & Thực hành Amazon S3:** <br>&emsp; + Tạo S3 Bucket với tên duy nhất toàn cầu (Globally Unique) <br>&emsp; + Upload, download và quản lý file/object qua Console & CLI <br>&emsp; + Tìm hiểu tính năng Block Public Access và cấu hình Bucket Policy | 22/09/2026 | 22/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - **Xây dựng Custom VPC:** <br>&emsp; + Tạo VPC với dải mạng IPv4 CIDR block <br>&emsp; + Tạo Public Subnet và Private Subnet <br>&emsp; + Tạo Internet Gateway (IGW) và đính kèm (attach) vào VPC <br>&emsp; + Cấu hình Route Table định tuyến traffic ra Internet qua IGW | 23/09/2026 | 23/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - **Thực hành EC2 nâng cao & Lưu trữ EBS:** <br>&emsp; + Khởi chạy EC2 trong Public Subnet của VPC vừa tạo <br>&emsp; + Dùng **User Data script** để tự động cài đặt và chạy Web Server (Nginx/Apache) khi khởi động <br>&emsp; + Tạo thêm một EBS Volume mới, attach vào EC2, định dạng phân vùng và mount vào thư mục hệ thống | 24/09/2026 | 24/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - **Hoàn thành các nhiệm vụ Free Tier còn lại:** <br>&emsp; + **Amazon RDS:** Khởi tạo database MySQL/PostgreSQL theo gói Free Tier, kiểm tra kết nối và xóa database sau khi hoàn thành <br>&emsp; + **AWS Budgets:** Thiết lập ngân sách kiểm soát chi phí (Cost Budget) kèm cảnh báo tự động gửi qua email khi chạm các ngưỡng 10$, 20$ | 25/09/2026 | 25/09/2026 | <https://aws.amazon.com/free/> |


### Kết quả đạt được tuần 2:

* Cài đặt và sử dụng thành thạo **AWS CLI**, có thể truy vấn và quản lý tài nguyên trực tiếp từ terminal máy cá nhân mà không cần phụ thuộc 100% vào giao diện Console.
* Nắm vững cơ chế hoạt động của **Amazon S3**:
  * Hiểu cách tổ chức Bucket và Object.
  * Phân biệt giữa Public Bucket và Private Bucket, hiểu cơ chế phòng hộ bảo mật của tính năng *S3 Block Public Access*.
* Nắm vững kiến trúc mạng cơ bản **Amazon VPC**:
  * Tự tay xây dựng được một Custom VPC hoàn chỉnh từ đầu.
  * Phân biệt rõ sự khác nhau giữa **Public Subnet** (có route ra Internet Gateway) và **Private Subnet** (cô lập an toàn nội bộ).
* Thành thạo kỹ thuật cấu hình máy ảo **Amazon EC2**:
  * Sử dụng User Data để tự động hóa hoàn toàn bước bootstrap (cài đặt gói phần mềm web server) ngay khi instance vừa bật lên.
  * Hiểu rõ vai trò của **EBS Volume** như một ổ đĩa cứng cắm rời (block storage), biết cách mount và mở rộng dung lượng cho máy ảo.
* Hoàn thành đầy đủ 5 nhiệm vụ thực hành của AWS Free Tier 2025 (EC2, RDS, Lambda, Bedrock, Budgets) và nhận tối đa gói **$200 Credit** để an tâm tiếp tục thực hành.
* Thiết lập thành công **AWS Budgets**, đảm bảo theo dõi chi phí chặt chẽ và không lo bị vượt hạn mức ngoài tầm kiểm soát.

---

### Khó khăn gặp phải & Cách giải quyết:

| Vấn đề gặp phải | Nguyên nhân | Cách khắc phục |
| --------------- | ----------- | -------------- |
| **Lỗi Access Denied (403) khi truy cập file công khai trên S3** | Mặc định tính năng *Block Public Access* của S3 được bật ở mức account và bucket để ngăn ngừa rò rỉ dữ liệu. | Tắt tính năng Block Public Access cho bucket cần công khai, đồng thời viết thêm **Bucket Policy** (cho phép hành động `s3:GetObject`) để cấp quyền truy cập hợp lệ. |
| **EBS Volume đã đính kèm (attach) nhưng không thấy dung lượng trên EC2** | Ổ cứng mới chưa được tạo phân vùng (file system) và chưa được gắn (mount) vào cây thư mục của Linux. | Dùng lệnh `lsblk` để xác định tên ổ cứng mới (`/dev/xvdf` hoặc `/dev/nvme1n1`), dùng lệnh `mkfs -t ext4` để định dạng phân vùng và lệnh `mount` để đưa ổ đĩa vào thư mục sử dụng. |
| **Amazon RDS khởi tạo lâu và không thể kết nối từ máy ngoài** | Quá trình provision cơ sở dữ liệu trên cloud mất 5–10 phút; Security Group mặc định của DB không mở port cho bên ngoài. | Kiên nhẫn chờ trạng thái chuyển sang `Available`, kiểm tra Inbound Rules của DB Security Group để mở đúng port tương ứng (3306 cho MySQL hoặc 5432 cho PostgreSQL). Sau khi kiểm tra xong, xóa ngay instance để tránh chi phí lưu trữ phát sinh. |

---

### Bài học kinh nghiệm rút ra:

1. **Bảo mật tuyệt đối thông tin Access Key & Secret Key:**
   * Tuyệt đối không commit file chứa credentials lên các kho mã nguồn công khai như GitHub.
   * Định kỳ kiểm tra hoặc thu hồi (revoke) các key không còn sử dụng.
2. **Kiến trúc mạng là xương sống của hệ thống Cloud:**
   * Việc hiểu rõ luồng đi của gói tin từ Internet qua IGW, Route Table rồi tới Subnet giúp việc cấu hình bảo mật và khắc phục sự cố (troubleshooting) trở nên rõ ràng và dễ dàng hơn.
3. **Sức mạnh của tự động hóa (Automation):**
   * Sử dụng User Data script giúp tiết kiệm đáng kể thời gian cài đặt thủ công và đảm bảo tính đồng nhất khi khởi tạo hàng loạt instance.

---

### Kế hoạch tuần tiếp theo (Tuần 3):

* Tìm hiểu về cơ chế cân bằng tải và tính sẵn sàng cao (High Availability): **Application Load Balancer (ALB)**.
* Tìm hiểu cơ chế tự động mở rộng theo nhu cầu: **Auto Scaling Group (ASG)**.
* Luyện tập dựng mô hình kiến trúc có khả năng chịu lỗi (Fault-tolerant Architecture) cơ bản trên AWS.
