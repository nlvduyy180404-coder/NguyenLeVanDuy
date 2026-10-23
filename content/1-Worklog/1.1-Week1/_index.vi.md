---
title: "Worklog Tuần 1"
date: 2026-09-14
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:

* Làm quen với các thành viên và nắm nội quy thực tập tại First Cloud AI Journey.
* Tìm hiểu chương trình AWS Free Tier mới (2025) và tạo tài khoản AWS.
* Thiết lập bảo mật cơ bản cho tài khoản (MFA, IAM user).
* Thực hành lần đầu với các dịch vụ: Amazon EC2, AWS Lambda, Amazon Bedrock.
* Hoàn thành các nhiệm vụ của Free Tier để nhận credit thưởng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | - Làm quen với các thành viên FCAJ <br> - Đọc và lưu ý các nội quy, quy định tại đơn vị thực tập | 14/09/2026 | 14/09/2026 | |
| 3   | - Đọc bài **"AWS Free Tier 2025: Revolution!"** để tìm hiểu Free Tier mới: <br>&emsp; + Credit khởi tạo $100, nhận thêm tối đa $100 khi hoàn thành các nhiệm vụ <br>&emsp; + Hai gói tài khoản: Free plan và Paid plan <br>&emsp; + Các dịch vụ Always Free <br>&emsp; + Các nhiệm vụ nhận credit: EC2, RDS, Lambda, Bedrock, AWS Budgets | 15/09/2026 | 15/09/2026 | <https://aws.amazon.com/free/> |
| 4   | - Tạo tài khoản AWS mới <br> - Chọn region làm việc trên AWS Console <br> - **Bảo mật tài khoản:** <br>&emsp; + Cài ứng dụng xác thực (Authenticator) trên điện thoại <br>&emsp; + Bật MFA cho root user <br>&emsp; + Tạo IAM user để sử dụng hằng ngày thay vì dùng root | 16/09/2026 | 16/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - **Thực hành Amazon EC2:** <br>&emsp; + Chọn AMI (hệ điều hành cho instance) <br>&emsp; + Chọn instance type thuộc Free Tier <br>&emsp; + Cấu hình Security Group: mở port 22 (SSH) và port 80 (HTTP) <br>&emsp; + Khởi chạy instance và kết nối vào instance <br>&emsp; + Chạy thử các lệnh cơ bản trên instance <br>&emsp; + Stop và terminate instance sau khi thử nghiệm để tránh phát sinh chi phí | 17/09/2026 | 17/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - **Thực hành AWS Lambda:** <br>&emsp; + Tạo Lambda function dùng runtime Node.js <br>&emsp; + Sửa code để function trả về nội dung riêng <br>&emsp; + Bật Function URL và gọi function qua trình duyệt <br> - **Thực hành Amazon Bedrock:** <br>&emsp; + Request quyền truy cập model (Model access) <br>&emsp; + Gửi prompt và so sánh câu trả lời giữa nhiều model khác nhau trên playground <br> - Kiểm tra credit thưởng nhận được sau khi hoàn thành các nhiệm vụ Free Tier | 18/09/2026 | 18/09/2026 | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 1:

* Hiểu cách hoạt động của chương trình AWS Free Tier mới: credit khởi tạo, credit thưởng khi hoàn thành nhiệm vụ, sự khác nhau giữa Free plan và Paid plan.

* Tạo thành công tài khoản AWS và áp dụng các bước bảo mật cơ bản:
  * Bật MFA cho root user bằng ứng dụng Authenticator
  * Tạo IAM user để làm việc hằng ngày, hạn chế dùng root
  * Hiểu được vì sao không nên dùng root user cho công việc thường ngày

* Nắm được các bước cơ bản để tạo một EC2 instance:
  * AMI: chọn hệ điều hành cho máy ảo
  * Instance type: cấu hình CPU/RAM, chọn loại thuộc Free Tier
  * Security Group: hoạt động như tường lửa, chỉ mở các port cần thiết (22 cho SSH, 80 cho HTTP)
  * Kết nối vào instance và thao tác bằng dòng lệnh
  * Terminate tài nguyên sau khi dùng để tránh tốn chi phí

* Tạo được Lambda function bằng Node.js, tự chỉnh sửa code và gọi qua Function URL; hiểu khái niệm serverless – chạy code mà không cần quản lý máy chủ.

* Biết cách sử dụng Amazon Bedrock:
  * Phải request Model access trước khi dùng model
  * Thử nhiều model khác nhau và nhận thấy mỗi model trả lời khác nhau với cùng một prompt

* Hoàn thành các nhiệm vụ Free Tier (EC2, Lambda, Bedrock) và nhận được credit thưởng vào tài khoản.

* Làm quen với AWS Management Console: tìm kiếm và truy cập dịch vụ, chọn region.
