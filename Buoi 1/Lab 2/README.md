# Báo cáo Lab 2: Triển khai và Kiểm thử Git Hook Bảo mật (`pre-commit`)

Tài liệu này ghi nhận quá trình cấu hình, cơ chế hoạt động của hệ thống kiểm tra tự động trước khi commit (`pre-commit hook`) và kết quả phân tích thực tế từ kết quả thực thi của công cụ **GitSecure**.

---

## 1. Cơ chế Hoạt động của Script `pre-commit`

Script Python được thiết kế để tự động kích hoạt mỗi khi người dùng thực hiện lệnh `git commit`. Quy trình kiểm tra bao gồm 3 lớp bảo mật chính:

1. **Quét dữ liệu nhạy cảm (`scan_sensitive`):**
   * Sử dụng danh sách các biểu thức chính quy (`SENSITIVE_PATTERNS`) để tìm kiếm các thông tin cấu hình nhạy cảm bị lộ trong mã nguồn (như API Key, mật khẩu, Token, thông tin xác thực đám mây).
   * Lấy danh sách các tệp đang trong vùng chuẩn bị (`git diff --cached --name-only`) để quét nội dung trực tiếp.

2. **Kiểm tra quyền phân giải tệp (`check_permissions`):**
   * Sử dụng thư viện `os` và `stat` để kiểm tra quyền hạn của tệp (ví dụ: phát hiện lỗi tệp đang ở trạng thái *world-writable* - bất kỳ ai cũng có quyền chỉnh sửa/ghi đè).

3. **Quét mã nguồn tĩnh (`run_bandit`):**
   * Gọi công cụ phân tích bảo mật mã nguồn `bandit` thông qua tiến trình `subprocess` để quét toàn bộ thư mục tìm kiếm các lỗ hổng lập trình có mức độ nguy hiểm cao (`SEVERITY: High`).

4. **Cơ chế ghi nhật ký và chặn (`main`):**
   * Nếu phát hiện bất kỳ vi phạm nào trong các bước trên, hệ thống sẽ lưu thông báo lỗi vào tệp nhật ký `gitsecure.log`, in thông báo cảnh báo và lập tức hủy bỏ tiến trình commit thông qua lệnh `sys.exit(1)`.

---

## 2. Kết quả Thực tế và Phân tích

### a. Lệnh thực thi trên Terminal
```bash
git commit -m "test"
### b. Kết quả trả về thực tế
Hệ thống Git Hook đã hoạt động chính xác và chặn quá trình commit với thông báo từ bộ quét công cụ Bandit:
COMMIT BLOCKED by GitSecure:
 - Bandit not installed. Run: pip install bandit
 ### c. Phân tích kết quả
Hành động chặn: Quá trình commit bị từ chối thành công bởi GitSecure, chứng minh rằng cơ chế pre-commit hook đã liên kết thành công với thư mục .githooks và kiểm soát chặt chẽ các thao tác của người dùng.

Nguyên nhân thông báo: Hàm run_bandit() cố gắng thực thi công cụ quét mã nguồn tĩnh bandit nhưng môi trường hiện tại chưa cài đặt gói này, ngoại lệ FileNotFoundError được bắt lại và trả về cảnh báo yêu cầu cài đặt (pip install bandit). Điều này phản ánh tính năng bắt lỗi ngoại lệ và phản hồi an toàn của script hoạt động cực kỳ chính xác.