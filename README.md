# 🛡️ Báo Cáo Phân Tích: Ngăn Chặn Rò Rỉ Thông Tin Bí Mật Trên GitHub (Secret Leak Prevention)

---

## 📌 Thông Tin Bài Nộp
* **Họ và tên:** Lê Hồ Thanh Thảo
* **Mã lớp / MSSV:** N26DTCN030
* **Môn học:** IT108-K26 - Nhập Môn Công Nghệ Thông Tin

---

## 1. Xây Dựng Pre-push Checklist Quét API Keys Hardcode

Để đảm bảo tuyệt đối không rò rỉ chìa khóa API, chứng thực hoặc mật khẩu khi đẩy mã nguồn lên **GitHub Public**, lập trình viên bắt buộc phải tuân thủ quy trình kiểm tra 4 bước trước mỗi lệnh `git push`:

### 1.1. Bảng Pre-push Checklist
- [ ] **Bước 1: Rà soát tệp thay đổi (`git status & git diff`):** Kiểm tra toàn bộ mã nguồn đang nằm trong trạng thái `Staged` để phát hiện các chuỗi bí mật bị ghi cứng (hardcoded).
- [ ] **Bước 2: Quét Regex chuỗi nhạy cảm (Automated Scan):** Sử dụng các biểu thức chính quy (Regex) để tìm kiếm các định dạng khóa phổ biến (AWS Access Key, Stripe API Key, JWT Secret, Private Key).
- [ ] **Bước 3: Tự động hóa qua Git Pre-push Hook / Gitleaks:** Cấu hình công cụ **Gitleaks** hoặc script tự động ngăn chặn lệnh `push` nếu phát hiện dữ liệu nhạy cảm.
- [ ] **Bước 4: Kiểm tra trạng thái `.gitignore`:** Đảm bảo tệp `.env` chứa các biến môi trường thực tế đã nằm trong danh sách loại trừ của Git.

---

### 1.2. Cấu hình tự động Script `pre-push` Hook
Tạo tệp `.git/hooks/pre-push` trong thư mục dự án với nội dung script sau để ngăn chặn các đợt push chứa thông tin bí mật:

```bash
#!/bin/sh
# Pre-push hook: Kiểm tra API Key / Secret hardcode trước khi push lên GitHub

# 1. Định nghĩa biểu thức Regex tìm kiếm các khóa phổ biến
SECRET_PATTERN="(AWS_SECRET_ACCESS_KEY|stripe_sk_live|AIzaSy|BEGIN PRIVATE KEY|ghp_[a-zA-Z0-9]{36})"

# 2. Tìm kiếm trong các commit chuẩn bị push
if git diff origin/main..HEAD | grep -E -q "$SECRET_PATTERN"; then
    echo "❌ ERROR: Phát hiện API Key hoặc Secret Key bị hardcode trong commit!"
    echo "⚠️ Vui lòng xóa thông tin nhạy cảm, đưa vào tệp .env và thử lại."
    exit 1
fi

echo "✅ Pre-push check passed: Không tìm thấy khóa bí mật bị hardcode."
exit 0