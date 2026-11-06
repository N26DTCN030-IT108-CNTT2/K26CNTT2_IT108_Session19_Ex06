## 3. Hướng Dẫn Ép Buộc Xác Thực 2 Lớp (2FA) Cho Repository Team

Để đảm bảo toàn bộ thành viên dự án không bị chiếm đoạt tài khoản GitHub và làm rò rỉ mã nguồn, Trưởng nhóm (Team Leader) cần triển khai quy trình 2FA bắt buộc:

### 3.1. Hướng dẫn thành viên bật 2FA cá nhân
* Truy cập GitHub $\rightarrow$ **Settings** $\rightarrow$ **Account security**.
* Tại mục **Two-factor authentication**, chọn **Enable two-factor authentication**.
* Sử dụng ứng dụng **Google Authenticator** quét mã QR và nhập mã 6 chữ số để xác nhận.
* Tải và cất giữ tập **Recovery Codes** ở nơi an toàn.

### 3.2. Ép buộc 2FA ở cấp Tổ chức (GitHub Organization Security Enforcement)
* Truy cập trang quản trị Organization trên GitHub $\rightarrow$ chọn **Settings**.
* Tìm đến mục **Security** $\rightarrow$ chọn **Authentication policies**.
* Tích chọn **Require two-factor authentication for everyone in your organization**.
* Nhấn **Save**. Hệ thống sẽ tự động vô hiệu hóa quyền truy cập đối với các tài khoản thành viên chưa bật 2FA cho đến khi họ kích hoạt đầy đủ.

---

## 4. Đánh Giá Rủi Ro An Toàn Thông Tin Khi Thiếu Tệp `.gitignore`

Tệp `.gitignore` chỉ định cho Git biết những tệp hoặc thư mục nào cần bỏ qua không theo dõi. Việc thiếu tệp `.gitignore` là một trong những nguyên nhân hàng đầu gây ra sự cố lộ dữ liệu nghiêm trọng trong ngành công nghệ.

### 4.1. Phân tích tác hại theo loại dữ liệu bị rò rỉ

| Loại tệp bị lộ | Dữ liệu nhạy cảm chứa bên trong | Hậu quả bảo mật & Thực tế |
| :--- | :--- | :--- |
| `.env`, `config.json` | Mật khẩu Database, AWS Access Keys, API Keys thanh toán (Stripe, PayPal). | **Thảm họa:** Hacker dùng Bot quét Public GitHub chỉ trong vài giây, trộm tài khoản AWS để đào tiền ảo (gây thiệt hại hàng ngàn USD) hoặc trộm dữ liệu khách hàng. |
| `id_rsa`, `*.pem` | Khóa SSH Private Key, Chứng chỉ TLS/SSL Server. | Hacker chiếm quyền đăng nhập từ xa (SSH Root Access) vào máy chủ production của doanh nghiệp. |
| `node_modules/`, `vendor/` | Toàn bộ các thư viện phụ thuộc và file mã nguồn bên thứ 3. | Làm phình to dung lượng Repository lên hàng trăm MB/GB, gây xung đột phiên bản và chậm tiến trình CI/CD. |
| `*.log`, `*.sqlite`, `*.db` | Nhật ký hệ thống, dữ liệu cá nhân người dùng thật (PII). | Vi phạm nghiêm trọng luật bảo mật dữ liệu (Nghị định 13/2023/NĐ-CP & GDPR), bị phạt tiền hoặc truy cứu trách nhiệm pháp lý. |
| `.DS_Store`, `.vscode/` | Cấu hình trình soạn thảo, đường dẫn thư mục cục bộ của máy dev. | Lộ diện sơ đồ hệ điều hành và cấu trúc đĩa cứng của lập trình viên, làm mục tiêu cho các đợt tấn công leo thang. |

---

## 5. Kết Luận & Quy Trình Khắc Phục Sự Cố Khi Lỡ Push Secret

* **Tuyệt đối không sử dụng `git commit` sửa lại rồi push đè:** Lịch sử Git (Git History) vẫn lưu lại commit cũ chứa chuỗi bí mật.
* **Thu hồi (Revoke) ngay lập tức:** Khi phát hiện lỡ push API Key/Password lên GitHub Public, việc đầu tiên cần làm là vào trang quản trị của dịch vụ đó để ĐỔI / THU HỒI khóa ngay lập tức.
* **Làm sạch lịch sử Git:** Sử dụng công cụ **BFG Repo-Cleaner** hoặc `git filter-repo` để xóa triệt để vết tích của tệp nhạy cảm khỏi toàn bộ lịch sử các commit trước khi public lại repository.