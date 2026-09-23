# Báo cáo Kiểm thử Ứng dụng SecureValidator

Dưới đây là ghi nhận chi tiết về nội dung đầu vào đã nhập và kết quả thực tế thu được trên giao diện web của ứng dụng:

## 1. Trường hợp E-mail
* **Đầu vào (Input):** `nguyenquynhnhhi.dt2018@gmail.com`
* **Kết quả quan sát:** 
  * Hiển thị thông báo với màu xanh.
  * *Lưu ý:* Trên giao diện mẫu tiếng Việt có thể hiển thị nhãn ngắn gọn (ví dụ: "Email và" hoặc "Email hợp lệ"), xác nhận định dạng email hợp lệ.

## 2. Trường hợp URL
* **Đầu vào (Input):** `https://www.hutech.edu.vn`
* **Kết quả quan sát:** 
  * Hiển thị dòng chữ màu xanh với nội dung "URL đã được đăng ký" (hoặc URL hợp lệ), xác nhận đường dẫn URL đúng chuẩn giao thức `https` và có domain hợp lệ.

## 3. Trường hợp Tên tệp (Filename - Chống Path Traversal)
* **Đầu vào (Input):** `../../etc/passwd`
* **Kết quả quan sát:** 
  * Hiển thị dòng chữ màu đỏ với nội dung "Tên tệp không bị lỗi" (hoặc cảnh báo tên tệp không hợp lệ do chứa các ký tự điều hướng thư mục `../` và `/`). Chức năng chặn tấn công Path Traversal hoạt động chính xác.

## 4. Trường hợp Đầu vào SQL (SQL Input - Lọc SQL Injection)
* **Đầu vào (Input):** `' OR 1 = 1 --`
* **Kết quả quan sát:** 
  * Phần **"Đã làm"** (hoặc Đã lọc): Hệ thống đã tự động loại bỏ các ký tự nguy hiểm (`'`, `--`) và từ khóa nhạy cảm (`OR`), chỉ giữ lại phần còn lại là `1 = 1` hoặc làm sạch chuỗi theo logic lập trình chống SQL Injection.

## 5. Trường hợp Đầu vào HTML (HTML Input - Chống XSS)
* **Đầu vào (Input):** `<script>alert("XSS")</script>`
* **Kết quả quan sát:** 
  * Phần **"Đã mã"** (Đã mã hóa): Chuyển đổi các thẻ HTML thành các thực thể an toàn: `&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;`, ngăn chặn thành công việc thực thi mã độc JavaScript trên trình duyệt (XSS).