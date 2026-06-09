## 📅 Bài 08: Tiến trình khởi động đầu tiên init (SysVinit)

Mục tiêu bài học giúp học viên nắm vững cơ chế hoạt động của hệ thống khởi tạo truyền thống **SysVinit** (System V init), hiểu rõ khái niệm **Runlevels** (Cấp độ chạy), cấu trúc tệp tin cấu hình `/etc/inittab`, và cách hệ thống nạp các dịch vụ nền tuần tự.

---

### 🧠 1. Khái Niệm Về Hệ Thống Khởi Tạo SysVinit

* **`init` (Initialization):** Là tiến trình cha đầu tiên được hạt nhân (Kernel) Linux kích hoạt trong bộ nhớ RAM ngay sau khi hoàn tất nạp phần cứng driver. Tất cả các tiến trình khác trên hệ thống đều là tiến trình con của `init`.
* **SysVinit (System V init):** Là tiêu chuẩn quản lý hệ thống khởi tạo truyền thống trên UNIX/Linux (được phát triển dựa trên cấu trúc System V cũ).
* **Cơ chế nạp dịch vụ tuần tự:** Đặc trưng cốt lõi của SysVinit là khởi động các dịch vụ hệ thống (như Web Server `httpd`, dịch vụ thời gian `ntpd`, mạng `network`...) theo kiểu **lần lượt và nối tiếp (tuần tự 1-2-3-4)**. Dịch vụ phía trước phải nạp xong hoàn toàn thì dịch vụ phía sau mới bắt đầu chạy.


---

### 🗂️ 2. Khái Niệm Runlevels (Cấp Độ Chạy) Trong Linux

SysVinit quản lý hệ thống thông qua các trạng thái hoạt động khác nhau gọi là **Runlevel** (được đánh số từ 0 đến 6). Mỗi runlevel đại diện cho một chế độ vận hành cụ thể:

* **Runlevel 0 (Halts/Shut down):** Chế độ tắt máy hoàn toàn.
* **Runlevel 1 (Single-user mode):** Chế độ một người dùng. Hệ thống tắt toàn bộ kết nối mạng (No Networking) và không chạy các dịch vụ công cộng. Chế độ này chỉ cho phép tài khoản quản trị tối cao `root` đăng nhập để bảo trì, sửa lỗi hệ thống.
* **Runlevel 2 (Multi-user mode mà không có mạng NFS):** Chế độ đa người dùng nhưng không hỗ trợ chia sẻ tệp tin qua mạng chia sẻ (NFS).
* **Runlevel 3 (Full Multi-user mode với CLI):** Chế độ đa người dùng đầy đủ chức năng mạng, nạp toàn bộ dịch vụ nền nhưng hoạt động hoàn toàn trên **giao diện dòng lệnh (Command Line Interface)**, không có đồ họa. Thường dùng cho các dòng máy chủ (Server).
* **Runlevel 4 (Unused):** Cấp độ dự phòng, chưa được định nghĩa sử dụng.
* **Runlevel 5 (Full Multi-user mode với GUI):** Chế độ đa người dùng đầy đủ tính năng mạng và có **giao diện đồ họa trực quan (X11 / Desktop Environment)**. Đây thường là chế độ khởi động mặc định trên máy tính cá nhân (PC/Laptop).
* **Runlevel 6 (Reboot):** Chế độ khởi động lại hệ thống.

---

### 📄 3. Cấu Trúc Tệp Tin Cấu Hình `/etc/inittab`

Ngay sau khi khởi động, tiến trình `init` sẽ tìm đọc tệp cấu hình `/etc/inittab` để xác định Runlevel mặc định và thực thi các tập lệnh kịch bản tương ứng. 

Mỗi dòng cấu hình trong tệp `/etc/inittab` tuân theo cú pháp phân tách nhau bởi **hai dấu chấm (`:`)** gồm 4 trường thông tin:
```text
id:runlevels:action:process

```

* **`id` (Identifier):** Mã định danh duy nhất (từ 1 đến 4 ký tự) đại diện cho dòng cấu hình đó.
* **`runlevels`:** Cấp độ chạy được áp dụng cấu hình (Ví dụ: điền `3` tức là chỉ chạy ở Runlevel 3; nếu để trống tức là áp dụng cho tất cả Runlevels).
* **`action`:** Hành động mà `init` sẽ thực hiện với tiến trình. Một số từ khóa phổ biến:
* `initdefault`: Xác định Runlevel mặc định mà hệ thống sẽ tự động đăng nhập khi bật máy.
* `sysinit`: Tập lệnh khởi tạo nền tảng hệ thống cần chạy trước khi kiểm tra các Runlevel.


* **`process`:** Đường dẫn thực tế đến mã nguồn tập lệnh Script (nằm trong thư mục hệ thống `/etc/rc.d/...`) cần được thực thi.

**Ví dụ thực tế cấu hình trong `/etc/inittab`:**

```text
id:3:initdefault:

```

*Giải thích:* Dòng này khai báo hệ thống sẽ lấy **Runlevel 3** (Giao diện dòng lệnh đa người dùng) làm chế độ khởi động mặc định, trường `process` để trống vì đây là lệnh thiết lập cấu hình tổng thể.

---

### 📁 4. Quản Lý Thư Mục Kịch Bản Dịch Vụ (`/etc/rc.d` & `/etc/init.d`)

Khi hệ thống chuyển đổi vào một Runlevel cụ thể, `init` sẽ quét các thư mục con tương ứng để bật/tắt dịch vụ nền:

* **Thư mục chứa mã nguồn gốc `/etc/init.d`:** Nơi lưu trữ toàn bộ các tệp tin kịch bản gốc (Shell Scripts) dùng để điều khiển các dịch vụ (Ví dụ: Tập lệnh khởi chạy mạng `network`, tường lửa `iptables`...).
* **Các thư mục Runlevel `/etc/rc.d/rc0.d` đến `/etc/rc.d/rc6.d`:**
* Mỗi Runlevel từ 0 đến 6 có một thư mục quản lý riêng (Ví dụ: Runlevel 3 sử dụng thư mục `rc3.d`).
* **Bản chất liên kết:** Các tệp tin bên trong thư mục `rcX.d` thực chất chỉ là các đường dẫn liên kết ký hiệu (**Symbolic Links / Symlink** - tương tự như Shortcut trên Windows) trỏ về tệp tin kịch bản gốc nằm trong thư mục `/etc/init.d`.


* **Tệp tin `/etc/rc.local`:** Là tệp tin đặc biệt cho phép người quản trị điền các câu lệnh tùy biến cá nhân. Tập lệnh này sẽ luôn được thực thi cuối cùng khi máy tính khởi động xong và nó không phụ thuộc vào bất kỳ quy tắc Runlevel nào.

---

*Ghi chú: Bản chất tuần tự của SysVinit, ý nghĩa chi tiết của 7 cấp độ Runlevels, cấu trúc 4 trường của tệp `/etc/inittab` và cơ chế liên kết Symlink trong thư mục `rcX.d` là những kiến thức nền tảng bắt buộc phải nhớ cho bài thi LPIC-1.*
