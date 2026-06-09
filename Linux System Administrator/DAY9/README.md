## 📅 Bài 09: Tìm hiểu về Systemd

Mục tiêu bài học giúp học viên hiểu rõ kiến trúc quản lý hệ thống hiện đại **Systemd**, sự cải tiến vượt trội so với hệ thống cũ SysVinit thông qua việc loại bỏ trình biên dịch trung gian Bash Script, và cách quản lý dịch vụ nền bằng các **Unit files**.

---

### 🧠 1. Sự Khác Biệt Giữa Systemd Và Hệ Thống Cũ (SysVinit / Upstart)

* **Hạn chế của hệ thống cũ:** Trong kiến trúc cũ (SysVinit/Upstart), việc khởi chạy dịch vụ phụ thuộc nặng nề vào các tập lệnh **Shell Script (Bash)**. Tập lệnh này đóng vai trò là một trình biên dịch trung gian nằm giữa dịch vụ và hệ điều hành. Mỗi khi dịch vụ chạy, nó phải đi qua lớp thông dịch này, làm chậm đáng kể tốc độ khởi động của hệ thống.
* **Cải tiến của Systemd:** Systemd thay thế hoàn toàn các tập lệnh Shell Script cồng kềnh bằng mã nguồn ngôn ngữ **C** đã được biên dịch sẵn sang mã máy. Do đó, Systemd có khả năng tương tác trực tiếp với Kernel, giúp tối ưu hóa hiệu năng và tăng tốc độ khởi động toàn diện cho hệ thống.


* **Khả năng tương thích:** Systemd vẫn duy trì khả năng tương thích ngược khoảng 99% với các tập lệnh Shell Script của hệ thống SysVinit cũ. Bạn vẫn có thể mang các đoạn Script dịch vụ từ hệ thống cũ (như CentOS 6) sang chạy trên các hệ thống dùng Systemd mới (như CentOS 8/9, Ubuntu) mà không gặp trở ngại lớn.

---

### 📂 2. Unit Files Khái Niệm & Ba Thư Mục Lưu Trữ Cốt Lõi

Mọi đối tượng hệ thống được quản lý bởi Systemd (bao gồm các dịch vụ chạy ẩn nền - Daemons/Background Services, các điểm gắn kết đĩa, thiết bị...) đều được gọi là một **Unit** và được định nghĩa thông qua các tệp tin cấu hình gọi là **Unit files**.

Unit files có cấu trúc định dạng phần (Sections) tương tự như định dạng tệp tin cấu hình mở rộng `.ini` truyền thống. Có 3 thư mục hệ thống chính lưu trữ các tệp tin này với mức độ ưu tiên khác nhau:

1. **Thư mục mặc định của nhà phát triển:** `/usr/lib/systemd/system/`
   * Đây là nơi chứa các Unit files mặc định do các gói phần mềm (như Apache `httpd`, Nginx, MySQL...) tự động tạo ra khi cài đặt.
   * *Khuyến nghị:* Người quản trị hệ thống **không được phép trực tiếp chỉnh sửa** các tệp tin trong thư mục này, vì chúng sẽ bị ghi đè và mất cấu hình mỗi khi chạy lệnh cập nhật nâng cấp hệ thống (System Update).
2. **Thư mục tùy biến của Người quản trị (System Administrator):** `/etc/systemd/system/`
   * Đây là nơi người quản trị hệ thống tạo mới hoặc tùy biến các tệp tin cấu hình Unit.
   * **Mức độ ưu tiên cao nhất:** Các tệp tin nằm trong thư mục này có quyền ưu tiên cao hơn, sẵn sàng ghi đè (Override) lên cấu hình mặc định của thư mục `/usr/lib/...`.
3. **Thư mục lưu trữ tạm thời thời gian thực (Runtime):** `/run/systemd/system/`
   * Chứa các Unit files được tạo ra linh hoạt theo thời gian thực khi hệ thống đang vận hành và tự động xóa sạch khi tắt máy.

---

### 📄 3. Cấu Trúc Thành Phần Căn Bản Của Một Unit File

Một tệp cấu hình dịch vụ chuẩn (Ví dụ: `httpd.service` của Apache Web Server) thường gồm các phần chính sau:

* **`[Unit]` (Thông tin chung & Mối quan hệ):**
  * `Description`: Mô tả ngắn gọn về chức năng của dịch vụ (Ví dụ: *The Apache HTTP Server*).
  * `Documentation`: Đường dẫn tra cứu tài liệu (hệ thống hướng dẫn `man` hoặc đường dẫn web).
  * `After`, `Requires`, `Conflicts`: Khai báo các điều kiện tiên quyết, thứ tự nạp dịch vụ (Ví dụ: Phải nạp sau dịch vụ mạng `network.target`).
* **`[Service]` (Kịch bản thực thi):**
  * Đóng vai trò là thành phần quan trọng nhất, chỉ định cách thức dịch vụ hoạt động.
  * `ExecStart`: Đường dẫn tuyệt đối đến tệp lệnh thực thi để khởi chạy dịch vụ (Ví dụ: `/usr/sbin/httpd -DFOREGROUND`).
  * `ExecStop`, `ExecReload`: Câu lệnh dùng để dừng hoặc nạp lại cấu hình dịch vụ.
* **`[Install]` (Cấu hình kích hoạt):**
  * Khai báo cách dịch vụ được kích hoạt tự động cùng hệ thống khi chạy lệnh `systemctl enable`.

---

### 💻 4. Lệnh Điều Hành Hệ Thống Với Systemd

Công cụ dòng lệnh chính thức được sử dụng để quản trị, giám sát và kiểm tra trạng thái các Unit trong Systemd là **`systemctl`** (*System Control*):

* **Liệt kê tất cả Unit files hiện có trên hệ thống:**
```bash
systemctl list-unit-files

```

* **Đọc trực tiếp nội dung cấu hình Unit file của một dịch vụ:**
```bash
# Xem cấu hình dịch vụ Web Server Apache
systemctl cat httpd.service

```



---

*Ghi chú: Nguyên lý tối ưu mã nguồn C của Systemd so với Bash Script cũ, phân biệt mục đích sử dụng và độ ưu tiên của hai thư mục `/usr/lib/systemd/system` và `/etc/systemd/system`, cùng hai cấu trúc lệnh `systemctl list-unit-files` và `systemctl cat` là những kiến thức cốt lõi cho bài thi chứng chỉ LPIC-1.*
