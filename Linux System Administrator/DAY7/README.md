## 📅 Bài 07: Quá trình khởi động Linux (Linux Boot Sequence)

Mục tiêu bài học nhằm cung cấp cái nhìn tổng quan, ngắn gọn về các giai đoạn diễn ra trong chuỗi trình tự khởi động của hệ điều hành Linux (Linux Boot Sequence), bản chất của bộ đệm vòng hạt nhân (Kernel Ring Buffer) và các câu lệnh kiểm tra log khởi động.

---

### 🧠 1. Sơ Đồ Các Giai Đoạn Khởi Động Linux (Linux Boot Sequence)

Quá trình khởi động hệ điều hành Linux từ lúc bật nút nguồn nguồn máy tính đến khi hiển thị màn hình đăng nhập được chia thành các bước tuần tự cốt lõi sau:


1. **Giai đoạn BIOS/UEFI:**  Ngay khi bật máy tính, chương trình **BIOS** (hoặc UEFI) tích hợp trên bo mạch chủ (Mainboard) sẽ được kích hoạt trước tiên.
   * BIOS thực hiện quá trình kiểm tra phần cứng cơ bản, kiểm tra tất cả các thiết bị đầu vào/đầu ra (Input/Output) như RAM, ổ cứng, bàn phím xem có hoạt động ổn định hay không.
2. **Giai đoạn Boot Loader (Trình tải khởi động):**
   * Sau khi kiểm tra phần cứng thành công, BIOS sẽ tìm đến phân vùng khởi động (**Boot Sector**) đầu tiên trên ổ cứng để gọi trình quản lý khởi động.
   * Trên các bản phân phối Linux hiện đại (như CentOS 7/8/9, Ubuntu...), trình quản lý khởi động phổ biến nhất được sử dụng là **GRUB** (*Grand Unified Bootloader*).
3. **Giai đoạn Tải Hạt Nhân Kernel & initramfs:**
   * GRUB tiến hành nạp tệp tin hạt nhân hệ điều hành (**Linux Kernel**) vào bộ nhớ.
   * Đồng thời, hệ thống nạp thêm tệp tin hệ thống đĩa RAM khởi tạo (**`initramfs`** hoặc *initrd*). Tệp tin này chứa một loạt các trình điều khiển thiết bị (Drivers) tạm thời cần thiết để Kernel có thể nhận diện và gắn kết (Mount) hệ thống tệp tin chính thức từ ổ đĩa cứng vật lý vào hệ điều hành.
4. **Giai đoạn Hệ Thống Khởi Tạo (Initialization System):**
   * Sau khi Kernel được thiết lập hoàn tất và sẵn sàng hoạt động, nó sẽ tự động giải phóng và xóa sạch các dữ liệu tạm thời không còn cần thiết trong RAM đĩa khởi tạo.
   * Kế tiếp, Kernel kích hoạt tiến trình cha đầu tiên của hệ thống, gọi là **Initialization System** (trên các hệ thống Linux hiện đại chính là dịch vụ **`systemd`** với tiến trình quản trị mang mã PID số 1).
   * Tiến trình khởi tạo này chịu trách nhiệm nạp tất cả các dịch vụ nền (**Services** / **Daemons**) và đưa máy tính về trạng thái sẵn sàng để người dùng đăng nhập sử dụng.

---

### 📂 2. Bộ Đệm Vòng Hạt Nhân (Kernel Ring Buffer) Là Gì?

* **Bản chất log khởi động:** Trong suốt quá trình máy tính khởi động và nạp driver phần cứng, Kernel sẽ liên tục tạo ra các thông báo log (Nhật ký hệ thống). 
* **Đặc điểm lưu trữ:** Các thông điệp log này ban đầu có tính chất **không ổn định**, vì chúng được ghi trực tiếp vào một vùng nhớ tạm của RAM gọi là **Kernel Ring Buffer** (Bộ đệm vòng hạt nhân).
* **Cơ chế hoạt động:** Do nằm trên RAM, bộ đệm vòng này sẽ bị xóa sạch hoàn toàn mỗi khi máy chủ tắt hoặc khởi động lại. Khi máy tính bật lên, Kernel sẽ ghi lại một chuỗi thông báo log mới hoàn toàn từ đầu theo thời gian thực.

---

### 💻 3. Các Câu Lệnh Kiểm Tra Log Khởi Động Hệ Thống

Để điều tra xem trong quá trình khởi động, phần cứng có bị lỗi hay driver nào nạp thất bại hay không, người quản trị hệ thống sử dụng hai câu lệnh chuyên dụng sau:

#### A. Lệnh `dmesg` (Display Message)
* **Chức năng:** Đọc và hiển thị trực tiếp toàn bộ nội dung thông báo tin nhắn đang được lưu trữ trong bộ đệm vòng hạt nhân (**Kernel Ring Buffer**).
* **Ứng dụng:** Thường dùng để kiểm tra thông tin phần cứng, trạng thái driver vừa được nạp lúc boot máy.
```bash
dmesg

```

#### B. Lệnh `journalctl -k`

* **Chức năng:** Đây là câu lệnh của hệ thống quản lý nhật ký tập trung trong **`systemd`** (được áp dụng trên các hệ điều hành như CentOS 8/9, Ubuntu).
* **Tham số `-k` (Kernel):** Chỉ định hệ thống lọc và hiển thị riêng các thông điệp log của hạt nhân Kernel (tương tự như dữ liệu từ bộ đệm vòng).
```bash
journalctl -k

```



---

*Ghi chú: Khái niệm về GRUB, vai trò của tệp `initramfs`, bản chất lưu trữ tạm thời của Kernel Ring Buffer và hai câu lệnh `dmesg`, `journalctl -k` là những kiến thức cốt lõi xuất hiện trong các câu hỏi thi chứng chỉ LPIC-1.*
