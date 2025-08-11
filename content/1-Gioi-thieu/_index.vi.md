---
title: "1. Giới thiệu"
chapter: false
weight: 1
---
## Tổng quan Workshop

Workshop này sẽ hướng dẫn bạn triển khai một hệ thống Identity Governance toàn diện với Access Certification trên AWS, bao gồm:

- **Access Governance**: Quản lý và kiểm soát quyền truy cập
- **Certification Automation**: Tự động hóa quy trình xác nhận quyền
- **Privilege Analytics**: Phân tích và giám sát đặc quyền
- **Risk Assessment**: Đánh giá rủi ro bảo mật
- **Monitoring Setup**: Thiết lập giám sát liên tục
- **Operational Procedures**: Quy trình vận hành
- **Audit Procedures**: Quy trình kiểm toán
- **Compliance Validation**: Xác thực tuân thủ

## Kiến trúc Tổng thể

![Architecture Diagram](/images/architecture-diagram.png)

## AWS Services Sử dụng (Kiến trúc Tối giản)

- **AWS IAM Identity Center** - Quản lý truy cập tập trung
- **AWS IAM** - Quản lý danh tính và truy cập
- **AWS Lambda** - Hàm tự động hóa và xử lý
- **Amazon EventBridge** - Điều phối sự kiện
- **Amazon DynamoDB** - Lưu trữ dữ liệu certification
- **Amazon S3** - Lưu trữ log và data lake
- **AWS CloudTrail** - Ghi log kiểm toán
- **Amazon CloudWatch** - Giám sát và metrics
- **Amazon SNS** - Thông báo và cảnh báo
- **Amazon QuickSight** - Dashboard phân tích
- **AWS Security Hub** - Đánh giá rủi ro và tuân thủ

## Lợi ích của Identity Governance

### 1. Bảo mật nâng cao

- Kiểm soát quyền truy cập chặt chẽ
- Phát hiện và ngăn chặn rủi ro bảo mật
- Giám sát liên tục các hoạt động

### 2. Tuân thủ quy định

- Đáp ứng các yêu cầu SOX, SOC2, ISO27001
- Tự động hóa quy trình audit
- Lưu trữ bằng chứng tuân thủ

### 3. Hiệu quả vận hành

- Tự động hóa quy trình certification
- Giảm thiểu công việc thủ công
- Cải thiện quy trình quản lý

## Tiếp theo

Chuyển sang [2. Các bước chuẩn bị](../2-cac-buoc-chuan-bi) để bắt đầu thiết lập môi trường.
