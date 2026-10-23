---
title: "Worklog Tuần 1"
date: 2026-09-14
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:

* Làm quen với các thành viên và nắm vững nội quy thực tập tại First Cloud AI Journey (FCAJ).
* Tìm hiểu chương trình **AWS Free Tier 2025** hoàn toàn mới và quy trình nhận credit.
* Khởi tạo tài khoản AWS cá nhân và thiết lập lớp bảo mật nền tảng (MFA, IAM User).
* Thực hành khởi chạy và dọn dẹp máy ảo **Amazon EC2** đầu tiên trên Cloud.
* Triển khai thử nghiệm ứng dụng Serverless với **AWS Lambda** và mô hình AI tạo sinh trên **Amazon Bedrock**.
* Hoàn thành các nhiệm vụ khởi đầu để nhận credit thưởng vào tài khoản.

---

### Tổng quan tiến độ công việc trong tuần:

| Thứ | Công việc chính | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------------- | ------------ | --------------- | -------------- |
| **2** | **Onboarding & Định hướng thực tập:** Gặp gỡ thành viên FCAJ, phổ biến quy chế, mục tiêu và lộ trình thực tập. | 14/09/2026 | 14/09/2026 | Tài liệu nội bộ FCAJ |
| **3** | **Nghiên cứu AWS Free Tier 2025:** Đọc bài *"AWS Free Tier 2025: Revolution!"*, tìm hiểu chính sách cấp credit và 2 gói Free/Paid plan. | 15/09/2026 | 15/09/2026 | [AWS Free Tier 2025](https://aws.amazon.com/free/) |
| **4** | **Khởi tạo & Bảo mật tài khoản AWS:** Đăng ký tài khoản, bật MFA cho root user, khởi tạo IAM User quản trị hàng ngày. | 16/09/2026 | 16/09/2026 | [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| **5** | **Thực hành máy chủ ảo Amazon EC2:** Khởi chạy EC2 instance (Amazon Linux 2023), cấu hình Security Group và kiểm tra kết nối. | 17/09/2026 | 17/09/2026 | [EC2 User Guide](https://docs.aws.amazon.com/ec2/) |
| **6** | **Khám phá Serverless & GenAI:** Viết Lambda Function bằng Node.js có Function URL; xin cấp quyền Model Access và test prompt trên Amazon Bedrock. | 18/09/2026 | 18/09/2026 | [AWS Lambda & Bedrock](https://docs.aws.amazon.com/bedrock/) |

---

### Nhật ký kỹ thuật chi tiết từng ngày (Detailed Technical Log):

#### Thứ Hai (14/09/2026): Onboarding & Tiếp nhận Quy chế Thực tập

1. **Gặp gỡ và kết nối đội ngũ:**
   - Tham gia buổi gặp gỡ chào đón thực tập sinh mới của chương trình **Workforce Bootcamp - First Cloud AI Journey**.
   - Làm quen với các mentor phụ trách chuyên môn và các thành viên cùng nhóm thực tập.
2. **Nghiên cứu tài liệu định hướng:**
   - Đọc kỹ hướng dẫn sử dụng template báo cáo thực tập trên Hugo.
   - Nắm rõ các mốc đánh giá quan trọng (Worklog hàng tuần, Báo cáo đồ án Proposal, Workshop chuyên đề, Đánh giá cuối kỳ).
   - Thiết lập môi trường làm việc cá nhân (Git, Visual Studio Code).

---

#### Thứ Ba (15/09/2026): Nghiên cứu Cơ chế AWS Free Tier 2025

1. **Những điểm đổi mới cốt lõi của Free Tier 2025:**
   - Khác với cơ chế cũ (giới hạn 750 giờ/tháng cố định trong 12 tháng), chương trình mới áp dụng mô hình **cấp Credit trực tiếp**:
     - **$100 Credit ban đầu:** Tự động kích hoạt ngay sau khi đăng ký tài khoản thành công.
     - **Tối đa thêm $100 Credit thưởng:** Kiếm thêm thông qua 5 nhiệm vụ thực hành cơ bản ($20/nhiệm vụ bao gồm: EC2, RDS, Lambda, Bedrock, AWS Budgets).
   - Cơ chế 2 gói tài khoản:
     - *Free Account Plan:* Thời hạn sử dụng trong 6 tháng, phù hợp nhất cho mục tiêu học tập và thực hành bootcamp.
     - *Paid Account Plan:* Dành cho hệ thống chạy lâu dài không giới hạn thời gian.
   - Danh mục hơn 30 dịch vụ thuộc nhóm **Always Free** (miễn phí vĩnh viễn trong hạn mức định sẵn).

2. **Chiến lược an toàn chi phí:**
   - Lập danh sách các dịch vụ dễ phát sinh chi phí ngoài ý muốn cần tránh bật khi chưa có nhu cầu (NAT Gateway, Elastic IP không gắn vào máy ảo, ổ cứng dung lượng lớn).

---

#### Thứ Tư (16/09/2026): Thiết lập & Củng cố Bảo mật Tài khoản AWS

1. **Khởi tạo tài khoản và kích hoạt MFA (Multi-Factor Authentication):**
   - Đăng ký tài khoản AWS cá nhân và hoàn tất xác minh danh tính quốc tế.
   - Ngay sau khi đăng nhập Root Account, truy cập **IAM Console** $\rightarrow$ **Security credentials**:
     - Kích hoạt **MFA** cho Root user bằng ứng dụng Authenticator trên điện thoại thông minh (Google Authenticator).
     - Quét mã QR và nhập 2 mã TOTP liên tiếp để đồng bộ thành công.

2. **Khởi tạo IAM User cho công việc hàng ngày (Tuân thủ Least Privilege):**
   - Truy cập **IAM** $\rightarrow$ **Users** $\rightarrow$ **Create user**:
     - Username: `duy-admin`
     - Cấp quyền truy cập AWS Management Console.
     - Gắn policy quản trị: `AdministratorAccess` (thông qua Group quản trị).
   - Đăng xuất hoàn toàn tài khoản Root. Từ thời điểm này, mọi thao tác học tập và triển khai đều sử dụng đường link đăng nhập riêng của IAM User (`https://<account-id>.signin.aws.amazon.com/console`).

---

#### Thứ Năm (17/09/2026): Thực hành Khởi chạy & Quản lý Máy ảo Amazon EC2

1. **Khởi tạo máy ảo EC2 đầu tiên:**
   - Chọn Region làm việc chính: **Singapore (`ap-southeast-1`)**.
   - Điều hướng tới **EC2 Dashboard** $\rightarrow$ chọn **Launch Instance**:
     - **Name:** `Duy-Demo-EC2-Instance`
     - **Application and OS Images (AMI):** `Amazon Linux 2023 AMI` (Free tier eligible, kiến trúc x86_64).
     - **Instance Type:** `t2.micro` (hoặc `t3.micro` tùy theo Free Tier hỗ trợ tại region).
     - **Key pair:** Tạo cặp khóa mới `duy-ec2-key.pem` và tải về máy an toàn.
   - Cấu hình **Security Group (Tường lửa ảo)**:
     - Rule 1: Cho phép **SSH (Port 22)** với Source là `My IP` để đảm bảo bảo mật.
     - Rule 2: Cho phép **HTTP (Port 80)** với Source là `Anywhere (0.0.0.0/0)`.

2. **Kết nối và dọn dẹp:**
   - Kết nối vào máy ảo trực tiếp qua giao diện web bằng **EC2 Instance Connect** (nhanh chóng, không cần cấu hình SSH client phức tạp).
   - Chạy các lệnh kiểm tra cấu hình Linux cơ bản:
     ```bash
     uname -a
     cat /etc/os-release
     free -m
     ```
   - Sau khi hoàn thành nhiệm vụ và nhận mốc $20 Credit đầu tiên, chọn instance $\rightarrow$ **Instance state** $\rightarrow$ **Terminate instance** để xóa hoàn toàn máy ảo, tránh hao hụt credit ngầm.

---

#### Thứ Sáu (18/09/2026): Thực hành Ứng dụng Serverless với Lambda & Thử nghiệm Bedrock GenAI

1. **Triển khai Hàm Serverless (AWS Lambda):**
   - Truy cập **AWS Lambda Console** $\rightarrow$ **Create function**:
     - Function name: `duy-hello-serverless`
     - Runtime: **Node.js 20.x**
     - Architecture: `x86_64`
   - Cập nhật mã nguồn hàm trong trình soạn thảo code tích hợp:
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
   - Tại mục **Configuration** $\rightarrow$ **Function URL**: Tạo một đường dẫn URL công khai có dạng `https://<id>.lambda-url.ap-southeast-1.on.aws/`. Mở link trên trình duyệt, kết quả JSON trả về ngay lập tức mà không cần quản lý bất kỳ máy chủ nào.

2. **Thử nghiệm AI Tạo sinh trên Amazon Bedrock:**
   - Chuyển sang Region **US East (N. Virginia - `us-east-1`)** để trải nghiệm đầy đủ thư viện mô hình.
   - Truy cập **Amazon Bedrock** $\rightarrow$ **Model access** $\rightarrow$ gửi yêu cầu cấp quyền sử dụng các foundation models (Anthropic Claude, Amazon Titan).
   - Mở **Bedrock Playgrounds** $\rightarrow$ **Text / Chat**:
     - Thử nghiệm gửi prompt kỹ thuật: *"Explain the core difference between Amazon EC2 and AWS Lambda in 3 concise bullet points."*
     - So sánh câu trả lời giữa các mô hình về độ trễ, tính mạch lạc và độ chính xác.
   - Nhận thêm các mốc credit thưởng cho nhiệm vụ Lambda và Bedrock.

---

### Kết quả đạt được tuần 1:

* **Bảo mật chuẩn AWS:** Nắm chắc nguyên tắc Least Privilege, kích hoạt thành công MFA cho tài khoản Root và chuyển toàn bộ thao tác sang tài khoản IAM User an toàn.
* **Hiểu rõ chính sách Free Tier mới:** Nắm vững cấu trúc nhận $200 credit của chương trình AWS Free Tier 2025, biết cách chọn dịch vụ phù hợp để tối ưu chi phí.
* **Làm chủ thao tác máy chủ ảo EC2:** Tự tay cấu hình AMI, Instance Type, Key Pair và thiết lập Security Group phòng hộ port 22/80.
* **Khái niệm Serverless đầu tiên:** Hiểu bản chất chạy code theo sự kiện (Event-driven) của AWS Lambda thông qua Function URL mà không cần hệ điều hành máy chủ.
* **Trải nghiệm GenAI trên Cloud:** Biết cách kích hoạt quyền truy cập mô hình (Model Access) và tương tác với các Foundation Models tiên tiến trên Amazon Bedrock Playground.
* **Thành tích tài khoản:** Hoàn thành các nhiệm vụ khởi động và ghi nhận số credit thưởng ban đầu thành công.

---

### Khó khăn gặp phải & Cách giải quyết:

| Vấn đề kỹ thuật gặp phải | Nguyên nhân gốc rễ | Cách khắc phục đã thực hiện |
| ------------------------ | ------------------ | --------------------------- |
| **Không thể dùng các foundation model trên Amazon Bedrock ngay** | Mặc định các model bị khóa bản quyền và cần yêu cầu cấp quyền trước; một số model chỉ hỗ trợ ở một vài region nhất định. | Vào mục **Model access** trong Bedrock Console, gửi form yêu cầu cấp quyền (Use case submission) và chuyển sang Region hỗ trợ đầy đủ (`us-east-1` - N. Virginia). |
| **Lỗi kết nối SSH vào EC2 (Connection timed out / Permission denied)** | Security Group chưa cho phép port 22 từ IP mạng hiện tại hoặc phân quyền file SSH key `.pem` chưa đúng trên máy cá nhân. | Cập nhật Inbound Rules của Security Group cho phép port 22 với nguồn là `My IP`. Sử dụng thêm tính năng **EC2 Instance Connect** trực tiếp trên web console để kết nối nhanh chóng mà không cần phần mềm bên ngoài. |
| **Lo ngại rủi ro phát sinh chi phí ngoài ý muốn** | Chưa quen với mô hình tính phí theo giờ và dung lượng của các dịch vụ đám mây. | Tạo thói quen kiểm tra trạng thái và **Terminate** ngay các instance EC2 sau khi hoàn tất bài thực hành; thường xuyên theo dõi mục **Billing & Cost Management Dashboard**. |

---

### Bài học kinh nghiệm rút ra:

1. **Nguyên tắc phân quyền tối thiểu (Least Privilege & Security First):**
   * Tuyệt đối không sử dụng tài khoản Root cho các hoạt động lập trình và cấu hình thường ngày.
   * Kích hoạt xác thực 2 lớp (MFA) ngay từ ngày đầu tiên để tạo lá chắn vững chắc cho tài khoản cloud.
2. **Ý thức quản trị chi phí Cloud (Cost Awareness):**
   * Dù có credit Free Tier, việc dọn dẹp các tài nguyên sau mỗi buổi lab là nguyên tắc vàng để tránh hao hụt credit hoặc phát sinh phí vào thẻ tín dụng.
3. **Mô hình Serverless vs Máy chủ truyền thống:**
   * Trải nghiệm AWS Lambda giúp nhận thấy ưu thế vượt trội của Serverless khi muốn chạy các tác vụ nhanh gọn mà không tốn công cài đặt hệ điều hành, cấu hình mạng hay lo lắng về việc tắt máy chủ sau khi dùng.

---

### Kế hoạch tuần tiếp theo (Tuần 2):

* Nghiên cứu công cụ dòng lệnh **AWS CLI**: cài đặt cục bộ, cấu hình profile credentials để quản trị tài nguyên qua terminal.
* Tìm hiểu sâu về nền tảng mạng cốt lõi: **Amazon VPC**, Public/Private Subnets, Internet Gateway và Route Tables.
* Thực hành dịch vụ lưu trữ đối tượng **Amazon S3** và lưu trữ khối **Amazon EBS**.
* Hoàn thành các thử thách Free Tier còn lại (tạo database với **Amazon RDS**, thiết lập cảnh báo ngân sách với **AWS Budgets**).
