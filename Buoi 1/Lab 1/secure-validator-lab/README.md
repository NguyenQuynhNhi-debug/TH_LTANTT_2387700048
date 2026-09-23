# Báo cáo Kiểm thử Ứng dụng SecureValidator

Tài liệu này ghi nhận chi tiết về nội dung đầu vào, kết quả thực tế thu được trên giao diện web cũng như phân tích kỹ thuật của ứng dụng **SecureValidator**.

---

## 1. Trường hợp E-mail (`validate_email`)
* **Đoạn mã phân tích:** Sử dụng biểu thức chính quy `r"^[\w.-]+@[\w.-]+\.\w+$"` kết hợp với `re.fullmatch`.
* **Đầu vào (Input):** `nguyenquynhnhhi.dt2018@gmail.com`
* **Kết quả quan sát & Phân tích:**
  * Chuỗi khớp hoàn toàn với định dạng (tên tài khoản, ký tự `@`, tên miền và phần mở rộng `.com`), do đó hàm trả về `True` (hợp lệ).
  * Trên giao diện web, ứng dụng hiển thị thông báo trạng thái hợp lệ (màu xanh).

---

## 2. Trường hợp URL (`validate_url`)
* **Đoạn mã phân tích:** Sử dụng `urllib.parse.urlparse(url)` để phân tách URL, kiểm tra `scheme` nằm trong danh sách cho phép (`http`, `https`) và `netloc` (tên miền/host) tồn tại.
* **Đầu vào (Input):** `https://www.hutech.edu.vn`
* **Kết quả quan sát & Phân tích:**
  * `scheme` là `https` được chấp nhận và domain `www.hutech.edu.vn` hợp lệ, hàm trả về `True`.
  * Giao diện hiển thị thông báo dòng chữ màu xanh xác nhận URL hợp lệ / đã được đăng ký.

---

## 3. Trường hợp Tên tệp - Chống Path Traversal (`validate_filename`)
* **Đoạn mã phân tích:** Kiểm tra ký tự điều hướng thư mục (`..`, `/`, `\`) và so sánh `os.path.basename(filename)` với chuỗi gốc để tránh đường dẫn phụ.
* **Đầu vào (Input):** `../../etc/passwd`
* **Kết quả quan sát & Phân tích:**
  * Phát hiện ký tự `..` và `/`, câu lệnh điều kiện kích hoạt và trả về `False` ngay lập tức.
  * Giao diện hiển thị cảnh báo màu đỏ (tên tệp không hợp lệ), chứng minh cơ chế chống tấn công Path Traversal hoạt động chính xác để bảo vệ file hệ thống.

---

## 4. Trường hợp Đầu vào SQL - Lọc SQL Injection (`sanitize_sql_input`)
* **Đoạn mã phân tích:** Sử dụng Regular Expression (`re.sub`) để tìm và xóa các ký tự đặc biệt trong SQL (`--`, `;`, `'`, `"`, `#`) và các từ khóa nhạy cảm (`OR`, `AND`, `SELECT`, `DROP`,...) không phân biệt chữ hoa/thường (`flags=re.IGNORECASE`).
* **Đầu vào (Input):** `' OR 1 = 1 --`
* **Kết quả quan sát & Phân tích:**
  * Lệnh `re.sub` thứ nhất xóa sạch dấu nháy đơn `'` và cụm `--`.
  * Lệnh `re.sub` thứ hai xóa từ khóa `OR`.
  * Kết quả làm sạch hiển thị trên giao diện phần đã lọc là `1 = 1`, tước bỏ hoàn toàn nguy cơ bẻ khóa câu lệnh cơ sở dữ liệu.

---

## 5. Trường hợp Đầu vào HTML - Chống XSS (`sanitize_html_input`)
* **Đoạn mã phân tích:** Sử dụng hàm `html.escape(html_str)` để chuyển đổi ký tự đặc biệt trong HTML sang dạng thực thể (entity) an toàn.
* **Đầu vào (Input):** `<script>alert("XSS")</script>`
* **Kết quả quan sát & Phân tích:**
  * Ký tự `<` chuyển thành `&lt;`
  * Ký tự `>` chuyển thành `&gt;`
  * Dấu nháy kép `"` chuyển thành `&quot;`
  * Kết quả trả về trên màn hình: `&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;`. Trình duyệt chỉ hiển thị dưới dạng văn bản thuần túy, ngăn chặn thành công việc thực thi mã độc JavaScript (XSS).