# Báo cáo Thực hành: DAST-TOOL

## 1. Giới thiệu tổng quan
Dự án này xây dựng một hệ thống công cụ kiểm tra bảo mật ứng dụng toàn diện bằng Python, bao gồm cả phân tích tĩnh (SAST) và kiểm tra động (DAST). Hệ thống có khả năng giám sát tiến trình thời gian thực, phân tích bộ nhớ và chống tràn dữ liệu, theo dõi lưu lượng mạng, kiểm tra hành vi mã nguồn, cung cấp môi trường Sandbox cô lập và xuất báo cáo chuẩn công nghiệp theo định dạng **SARIF (Static Analysis Results Interchange Format)**.

---

## 2. Các thành phần kỹ thuật chính (Architecture & Modules)

Công cụ được cấu trúc thành các module độc lập thực hiện từng nhiệm vụ cụ thể:

- **`runtime_monitor.py` (Runtime Monitoring):** Sử dụng thư viện `psutil` để theo dõi hiệu năng tiến trình theo thời gian thực bao gồm mức sử dụng CPU (`cpu_percent`), bộ nhớ RAM (`memory_info().rss`), và danh sách các tệp đang mở (`open_files`).
- **`memory_analyzer.py` (Network Traffic Analysis):** Thu thập và thống kê số liệu thống kê giao diện mạng (`net_io_counters`) phục vụ việc phân tích lưu lượng.
- **`security_behavior_checker.py` (Security Behavior Check):** Tự quét mã nguồn của tệp để phát hiện các lệnh hệ thống nhạy cảm hoặc nguy hiểm như `os.system`, `eval`, và `exec`.
- **`virtual_sandbox.py` (Virtual Sandbox):** Xây dựng môi trường cô lập tạm thời sử dụng `tempfile` và `subprocess` để thực thi các câu lệnh shell một cách an toàn mà không ảnh hưởng đến hệ thống chính.
- **`app.py` & `main.py`(API Gateway & CLI Entrypoint):** Cung cấp giao diện dòng lệnh trực tiếp và dịch vụ Web API thông qua framework **Flask**, cho phép truy vấn các chỉ số bảo mật qua HTTP endpoints.

---

## 3. Các bước thực hiện và Hướng dẫn sử dụng

### Bước 1: Cài đặt các thư viện phụ thuộc
Đảm bảo bạn đã cài đặt đầy đủ các thư viện cần thiết (psutil, Flask, v.v.) theo cấu hình trong `requirements.txt`:
```bash
pip install -r requirements.txt
```

### Bước 2: Chạy các bài kiểm thử (Unit Tests)
Hệ thống bao gồm các bài unit test tự động để xác thực độ chính xác của từng module bảo mật động. Chạy lệnh sau trong terminal:
```bash
python -m unittest discover tests
```
*Kết quả thực thi mẫu:*
![alt text](image.png)

### Bước 3: Chạy chương trình kiểm tra tổng hợp qua CLI (`main.py`)
Thực hiện chạy tệp kịch bản kiểm tra toàn diện:
```bash
python .\main.py
```
*Kết quả thực thi mẫu:*
![alt text](image-1.png)

### Bước 4: Khởi chạy Web API Server (`app.py`)
Triển khai ứng dụng dưới dạng dịch vụ Web thông qua Flask để cung cấp các API giám sát bảo mật động:
```bash
python .\app.py
```
*Kết quả đầu ra:*
![alt text](image-2.png)
---

## 4. Phân tích kết quả và Giải thích kỹ thuật

Khi thực hiện quét file `vulnerable_01.py`, công cụ đã phát hiện tổng cộng **9 vấn đề bảo mật và chất lượng mã nguồn** (như hiển thị trên giao diện kết quả SARIF Viewer trong VS Code):
![alt text](image-2.png)
1. **Use of eval() detected:** Phát hiện việc sử dụng hàm `eval()` không kiểm soát đầu vào người dùng, có thể dẫn đến lỗ hổng Remote Code Execution (RCE).
2. **Hardcoded password detected:** Phát hiện mật khẩu được gán cứng trực tiếp trong mã nguồn (`password = "123456"`), vi phạm nguyên tắc bảo mật thông tin nhạy cảm.
3. **Usage of pickle module:** Phát hiện việc import module `pickle`, vốn có rủi ro thực thi mã độc khi giải tuần tự hóa dữ liệu không tin cậy.
4. **Lint Issues (PEP 8 Violations):** Các lỗi phong cách lập trình từ `flake8` như `F401` (unused import), `E402` (module level import not at top of file), và `E302` (expected 2 blank lines).

---

## 5. Kết luận
Công cụ SAST tự phát triển đã hoạt động thành công, đáp ứng tốt yêu cầu tự động hóa việc phát hiện lỗ hổng bảo mật dựa trên cả phương pháp phân tích cú pháp (AST), khớp mẫu (Regex) và kiểm tra chuẩn chất lượng mã nguồn (Linting), đồng thời chuẩn hóa định dạng xuất báo cáo theo chuẩn bảo mật quốc tế SARIF.