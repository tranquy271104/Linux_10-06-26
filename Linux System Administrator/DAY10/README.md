## 📅 Bài 10: Chuyển đổi môi trường trong Linux Runlevels (SysVinit)

Mục tiêu bài học giúp học viên củng cố vững chắc kiến thức về các cấp độ chạy (**Runlevels**), nắm vững các câu lệnh kiểm tra, chuyển đổi môi trường thủ công (`runlevel`, `telinit`), và kỹ thuật can thiệp vào trình quản lý khởi động GRUB để chỉ định Runlevel cứu hộ phần cứng khi boot máy.

---

### 🧠 1. Hệ Thống 7 Cấp Độ Runlevels Trong SysVinit (Ôn Tập)

Hệ thống quản lý khởi tạo truyền thống phân chia trạng thái vận hành của máy chủ Linux thành 7 cấp độ cụ thể:

* **Runlevel 0:** Tắt máy hoàn toàn (**Halt / Power Off / Shutdown**).
* **Runlevel 1 (Single-user mode - Chế độ đơn người dùng):** Không có cấu hình mạng, không chạy các dịch vụ công cộng, chỉ cho phép tài khoản quản trị tối cao `root` đăng nhập để bảo trì hệ thống và sửa lỗi.
* **Runlevel 2:** Chế độ đa người dùng (**Multi-user mode**) nhưng không hỗ trợ chia sẻ tệp tin qua mạng NFS.
* **Runlevel 3 (Full Multi-user với CLI):** Chế độ đa người dùng đầy đủ mạng, chạy tất cả dịch vụ nhưng chỉ hoạt động trên giao diện dòng lệnh văn bản (**Command Line Interface**), không tải giao diện đồ họa. Đây là chế độ tối ưu nhất cho máy chủ (Server).
* **Runlevel 4 (Unused):** Cấp độ dự phòng cho người dùng tự tùy biến môi trường riêng.
* **Runlevel 5 (Full Multi-user với GUI):** Chế độ đa người dùng đầy đủ mạng và có **giao diện màn hình đồ họa trực quan (Desktop Environment)**. Thường đặt làm chế độ mặc định khởi động của các máy trạm cá nhân.
* **Runlevel 6:** Khởi động lại hệ thống (**Reboot**).

---

### 💻 2. Các Câu Lệnh Chuyển Đổi Môi Trường Vận Hành

Khi hệ thống đang hoạt động, người quản trị có thể kiểm tra trạng thái hoặc ra lệnh ép hệ thống chuyển sang một môi trường runlevel khác bằng các lệnh chuyên dụng:

#### A. Lệnh `runlevel`
* **Chức năng:** Hiển thị Runlevel hiện tại hệ thống đang sử dụng và Runlevel ngay trước đó.
* **Cấu trúc kết quả trả về:** Gồm hai ký tự phân tách nhau (Ví dụ: `N 5` hoặc `5 3`).
  * Ký tự thứ nhất đại diện cho Runlevel trước đó (`N` nghĩa là None - không có, hệ thống vừa bật nguồn lên thẳng).
  * Ký tự thứ hai đại diện cho Runlevel hiện tại hệ thống đang chạy.

#### B. Lệnh `telinit` (hoặc `init`)
* **Chức năng:** Ra lệnh cho tiến trình `init` chuyển đổi ngay lập tức sang một Runlevel được chỉ định.
* **Cú pháp:** `telinit <số_runlevel>` (hoặc `init <số_runlevel>`).
```bash
  # Chuyển từ môi trường đồ họa (Runlevel 5) sang môi trường dòng lệnh (Runlevel 3)
  telinit 3

  # Ra lệnh tắt máy chủ ngay lập tức bằng cơ chế chuyển về Runlevel 0
  init 0

```

---

### 🛠️ Kỹ Thuật Can Thiệp GRUB Để Thay Đổi Runlevel Khi Khởi Động

Trong trường hợp hệ thống gặp sự cố cấu hình sai (Ví dụ: Bị treo ở giao diện đồ họa Runlevel 5 không vào được), bạn cần can thiệp trực tiếp vào Bootloader khi bật nguồn để ép máy vào Runlevel cứu hộ (Runlevel 3 hoặc Runlevel 1):

1. Bật nguồn máy chủ, ngay tại màn hình đếm giây khởi động của **GRUB Boot Menu**, nhấn một phím bất kỳ trên bàn phím để dừng quá trình boot tự động.
2. Sử dụng phím mũi tên di chuyển để chọn dòng Hạt nhân (Kernel) cần cấu hình.
3. Nhấn **phím `a**` (hoặc phím `e` tùy phiên bản) trên bàn phím để tiến vào chế độ chỉnh sửa dòng lệnh nạp Kernel (Append/Edit mode).
4. Di chuyển con trỏ xuống cuối dòng lệnh, nhấn dấu cách (**Space**) để tạo khoảng trống, sau đó gõ số của Runlevel mong muốn khởi động (Ví dụ: gõ số `3` để vào môi trường CLI, hoặc số `1` để vào chế độ đơn người dùng cứu hộ).
5. Nhấn phím **Enter** để hệ thống áp dụng và tự động nạp các dịch vụ nền tương ứng với Runlevel vừa chỉ định thời gian thực.

---

### 📝 Kịch Bản Thử Nghiệm Trên Hệ Thống SysVinit Cũ (CentOS 6 trở về trước)

*Lưu ý: Bài học này thao tác mô phỏng trên cấu trúc SysVinit truyền thống. Các hệ điều hành mới dùng Systemd (CentOS 7/8/9, Ubuntu) sử dụng cơ chế Target Units sẽ được hướng dẫn ở bài tiếp theo.*

```bash
# 1. Kiểm tra môi trường runlevel hiện hành
runlevel
# Kết quả trả về thông thường: N 5 (Đang ở chế độ đồ họa mặc định)

# 2. Xem tệp cấu hình quy định runlevel mặc định của hệ thống
cat /etc/inittab
# Tìm dòng cấu hình: id:5:initdefault:

# 3. Ép hệ thống tắt giao diện đồ họa, chuyển về dòng lệnh để tối ưu RAM
sudo telinit 3

# 4. Đăng nhập lại tài khoản root, kiểm tra lại trạng thái
runlevel
# Kết quả trả về: 5 3 (Chuyển từ cấp độ 5 sang cấp độ 3 thành công)

# 5. Tắt máy chủ an toàn thông qua việc gọi Runlevel 0
sudo init 0

```

---

*Ghi chú: Cách đọc kết quả lệnh `runlevel`, cú pháp tệp `/etc/inittab` (`id:5:initdefault:`), lệnh chuyển đổi môi trường `telinit` và kỹ thuật can thiệp đối số dòng lệnh Kernel trong GRUB là những câu hỏi thực hành trọng tâm của chứng chỉ LPIC-1.*
