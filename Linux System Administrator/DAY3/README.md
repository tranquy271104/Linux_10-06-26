## 📅 Bài 03: Cài đặt cấu hình Vmware và Linux Centos & Ubuntu (Phần 2)

Mục tiêu của bài học này nhằm hoàn thiện môi trường Lab ảo hóa bằng cách cài đặt hệ điều hành **Ubuntu Desktop 22.04**, thiết lập công cụ quản trị từ xa **PuTTY**, và tối ưu hóa hệ thống CentOS Stream 8 phục vụ học tập.

### 🛠️ Quy Trình Triển Khai Chi Tiết

#### 1. Triển Khai Máy Ảo Ubuntu Desktop 22.04 LTS
* **Tải bộ cài OS:** Truy cập trang chủ `ubuntu.com`, chọn tải xuống phiên bản ổn định dài hạn **Ubuntu 22.04 LTS (Desktop)** (Dung lượng ~4.6 GB).
* **Khởi tạo máy ảo trên VMware:**
  * Chọn `Create a New Virtual Machine` -> Tích chọn `I will install the operating system later`.
  * Hệ điều hành khách (**Guest OS**): Chọn **Linux** / Phiên bản (**Version**): Chọn **Ubuntu 64-bit**.
  * **Đường dẫn lưu trữ (`Location`):** Chuyển từ ổ mặc định `C:\` sang ổ đĩa dữ liệu khác (`D:\`, `E:\`...) để bảo vệ dữ liệu máy ảo.
  * **Cấu hình ổ đĩa:** Đặt dung lượng mặc định **20 GB** dưới dạng một file duy nhất (`Store virtual disk as a single file`).
* **Tiến trình cài đặt đồ họa:**
  * Gắn file ISO vào mục CD/DVD trong cấu hình phần cứng -> Chọn `Power On`.
  * Tại menu khởi động, chọn `Try or Install Ubuntu`.
  * Nhấn nút **Install Ubuntu** -> Giữ cấu hình bàn phím mặc định `English (US)`.
  * Kiểu cài đặt: Chọn phân vùng tự động mặc định (`Erase disk and install Ubuntu`) -> Chọn **Install Now**.
  * Thiết lập múi giờ: Chọn vùng **Vietnam** (Múi giờ Hồ Chí Minh).
  * Tạo tài khoản người dùng: Nhập họ tên, tên máy tính, thiết lập tên tài khoản đăng nhập (`username`) và mật khẩu hệ thống.
  * Sau khi hoàn tất sao chép tệp tin, chọn **Restart Now** để khởi động lại và đăng nhập vào màn hình chính Desktop.

---

#### 2. Cài Đặt Công Cụ Quản Trị Từ Xa PuTTY
Để thuận tiện cho việc thực hành và dễ dàng sao chép dòng lệnh từ máy vật lý vào hệ thống Linux, ta sử dụng giao thức kết nối Secure Shell (SSH) thông qua phần mềm PuTTY.
* Truy cập trang chủ `putty.org`, tải tệp thực thi độc lập `putty.exe` về máy thật Windows.
* Quay trở lại máy ảo **CentOS 8 Stream**, mở ứng dụng **Terminal** và chạy lệnh kiểm tra IP:
  ```bash
  ip address show


* Ghi nhớ địa chỉ IP hiển thị tại card mạng thứ nhất (Card mạng mạng NAT).
* Mở **PuTTY** trên Windows: Điền IP của CentOS vào ô `Host Name` (Port: 22, Connection type: SSH) -> Nhấn `Open` -> Chọn `Accept` tại bảng cảnh báo bảo mật lần đầu kết nối -> Đăng nhập bằng tài khoản tối cao `root` và mật khẩu tương ứng.

---

#### 3. Tối Ưu Hóa Hệ Thống Hậu Cài Đặt Trên CentOS Stream 8

Thực hiện các câu lệnh sau trên cửa sổ dòng lệnh PuTTY để đơn giản hóa môi trường Lab, giúp các dịch vụ mạng ở các bài học sau không bị tường lửa hay cơ chế bảo mật hạt nhân chặn lại:

* **Vô hiệu hóa SE Linux vĩnh viễn:**
Chuyển trạng thái bảo mật của SE Linux về cơ chế tắt tạm thời (Permissive) và chỉnh sửa tệp tin cấu hình hệ thống để tắt vĩnh viễn (Disabled) sau khi khởi động lại:
```bash
# Chuyển trạng thái thực thi tạm thời
sudo setenforce 0

# Cấu hình tắt vĩnh viễn trong tệp tin hệ thống
sudo sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

```


* **Vô hiệu hóa Tường lửa (Firewall):**
```bash
# Dừng ngay lập tức dịch vụ tường lửa đang chạy
sudo systemctl stop firewalld

# Vô hiệu hóa dịch vụ, ngăn tự động chạy khi khởi động lại hệ thống
sudo systemctl disable firewalld

```


* **Kiểm tra kết nối Internet từ Terminal:**
```bash
ping google.com

```



---

*Ghi chú: Hoàn thành bài học này, bạn đã xây dựng thành công hạ tầng Lab ảo hóa cơ bản gồm cả CentOS Stream 8 và Ubuntu Desktop 22.04 sẵn sàng cho lộ trình LPIC-1.*
