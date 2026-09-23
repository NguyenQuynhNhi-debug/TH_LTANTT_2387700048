## 1. Tổng quan bài Lab
Bài thực hành tập trung vào việc tích hợp hệ thống ghi log bảo mật (`securelogger`) kết hợp với thư viện kiểm tra đầu vào (`securevalidator`) thông qua một Flask API (`/validate`)[cite: 7, 8]. Quá trình kiểm thử giúp làm rõ cách xử lý các ngoại lệ và quy tắc định tuyến (routing) trong ứng dụng web.

---

## 2. Quá trình Kiểm thử API và Xử lý Lỗi thực tế

Trong quá trình gọi API kiểm thử bằng công cụ, một số trường hợp lỗi định tuyến đã được ghi nhận và xử lý:

* **Lỗi 404 Not Found (Sai đường dẫn URL):**
  * *Nguyên nhân:* Gửi yêu cầu `POST` trực tiếp vào trang chủ `http://127.0.0.1:5000/` trong khi server chỉ định nghĩa route tại `/validate`[cite: 11].
  * *Khắc phục:* Cập nhật đúng đường dẫn endpoint thành `http://127.0.0.1:5000/validate`[cite: 12].

* **Lỗi 405 Method Not Allowed (Sai phương thức HTTP):**
  * *Nguyên nhân:* Truy cập đường dẫn `/validate` nhưng sử dụng phương thức `GET`[cite: 12].
  * *Khắc phục:* Chuyển đổi phương thức sang `POST` đúng theo định nghĩa `@app.route("/validate", methods=["POST"])`[cite: 8, 12].

---