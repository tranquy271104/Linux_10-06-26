## 📅 Bài 05: Làm việc với Kernel Modules

Mục tiêu bài học nhằm giúp học viên hiểu rõ bản chất kiến trúc hạt nhân nguyên khối của Linux, cơ chế hoạt động của các mô-đun hạt nhân (**Kernel Modules / Drivers**) và thành thạo các câu lệnh quản trị, nạp/gỡ bỏ mô-đun trên hệ thống mà không cần khởi động lại máy chủ.

---

### 🧠 1. Tổng Quan Về Linux Kernel & Cấu Trúc Nguyên Khối (Monolithic)

* **Hệ điều hành Linux GNU:** Thường được gọi là hệ điều hành Linux, trong đó **GNU** cung cấp hệ thống các công cụ tiện ích bao quanh (như Shell Bash, các trình biên dịch, tập lệnh cơ bản) và **Linux** đóng vai trò là hạt nhân (Kernel) cốt lõi của toàn bộ hệ thống.
* **Chức năng của Kernel:** Đóng vai trò làm cầu nối trung gian, cho phép các phần mềm ứng dụng tương tác trực tiếp với thiết bị phần cứng (Bộ nhớ RAM, ổ đĩa cứng, card mạng, CPU...).


* **Kiến trúc Monolithic Kernel (Hạt nhân nguyên khối):**
  * Hạt nhân Linux tự xử lý toàn bộ các tương tác cốt lõi bao gồm quản lý bộ nhớ, tiến trình và thiết bị phần cứng một cách tập trung để tối ưu hiệu năng.
  * **Cơ chế mô-đun động:** Thay vì phải tích hợp tất cả driver vào một tệp tin hạt nhân duy nhất (gây nặng máy), Linux cho phép mở rộng chức năng bằng cách tải (Load) hoặc gỡ bỏ (Unload) các thành phần tính năng/driver linh hoạt khi hệ thống đang chạy. Các thành phần này được gọi là **Kernel Modules**.
  * **Ưu điểm vượt trội:** Khi bạn thêm một thiết bị phần cứng mới hoặc cập nhật Driver của bên thứ ba (Third-party), hệ thống sẽ tự động nạp mô-đun tương ứng vào bộ nhớ mà **không cần phải khởi động lại (Reboot) máy chủ**.

---

### 💻 2. Các Câu Lệnh Quản Trị Kernel Modules Thường Gặp

Trong công việc thực tế của một **System Administrator**, bạn sẽ thường xuyên tương tác với các lệnh quản lý mô-đun sau đây để xử lý sự cố phần cứng hoặc tối ưu dịch vụ:

#### A. Lệnh `uname` (Unix Name)
Dùng để hiển thị thông tin chi tiết về hệ thống và phiên bản hạt nhân Kernel đang vận hành.
* `uname`: Chỉ hiển thị tên hệ điều hành cốt lõi (`Linux`).
* `uname -m`: Hiển thị kiến trúc phần cứng của máy tính (Ví dụ: `x86_64`).
* `uname -r`: Hiển thị chính xác phiên bản phát hành của Kernel hiện tại (Release version, ví dụ: `4.18.0...`).
* `uname -a` (All): In toàn bộ thông tin hệ thống ra màn hình bao gồm: Tên hệ điều hành, Hostname của máy, Phiên bản Kernel, Ngày build Kernel và kiến trúc phần cứng.

#### B. Lệnh `lsmod` (List Modules)
* **Chức năng:** Liệt kê toàn bộ các Kernel Modules hiện đang được nạp và hoạt động trong bộ nhớ RAM của hệ thống.
* **Cấu trúc bảng hiển thị:** Gồm 3 cột thông tin chính:
  1. **Module:** Tên của mô-đun (driver).
  2. **Size:** Kích thước dung lượng của mô-đun chiếm dụng trong bộ nhớ (đơn vị Bytes).
  3. **Used by:** Mô-đun này đang được sử dụng bởi các thiết bị mạng, tiến trình hoặc các mô-đun phụ thuộc (Dependencies) nào khác.

#### C. Lệnh `modinfo` (Module Information)
* **Chức năng:** Tra cứu thông tin chi tiết và nguồn gốc của một mô-đun cụ thể.
* **Cú pháp:** `modinfo <tên_module>` (Ví dụ: `modinfo cdrom`).
* **Thông tin cung cấp:** Đường dẫn tệp tin thực tế của mô-đun trên ổ đĩa (`filename`), bản quyền tác giả, mô tả chức năng và các tham số cấu hình nâng cao.

#### D. Lệnh `modprobe` & `rmmod` (Quản lý nạp/gỡ mô-đun)
* **Gỡ bỏ mô-đun cứng nhắc bằng `rmmod` (Remove Module):** Gỡ bỏ trực tiếp một mô-đun ra khỏi RAM. Lệnh này sẽ thất bại nếu có mô-đun khác đang phụ thuộc vào nó.
* **Gỡ bỏ thông minh bằng `modprobe -r`:** Tự động phân tích cây phụ thuộc, hỗ trợ gỡ bỏ mô-đun một cách an toàn và dọn dẹp các mô-đun liên đới liên quan.
* **Nạp mô-đun bằng `modprobe`:** Tìm kiếm driver trong thư mục hệ thống để nạp trực tiếp vào RAM kèm theo việc tự động giải quyết các mô-đun phụ thuộc đi kèm.

---

### 🛠️ Kịch Bản Thực Hành Thực Tế (Lab Demo)

*Yêu cầu: Chỉ thực hiện trên môi trường Lab thử nghiệm, không thao tác trực tiếp trên các hệ thống Production đang chạy thực tế của doanh nghiệp.*

#### Bước 1: Điều tra phiên bản hệ thống hiện tại
```bash
# Xem toàn bộ thông tin hệ thống
uname -a

```

#### Bước 2: Tìm kiếm mô-đun ổ đĩa CD-ROM (`cdrom`)

```bash
# Kiểm tra xem driver cdrom có đang chạy trong RAM không
lsmod | grep cdrom

```

#### Bước 3: Xem thông tin chi tiết của Driver `cdrom`

```bash
modinfo cdrom

```

#### Bước 4: Thao tác gỡ bỏ mô-đun an toàn (Xử lý Dependencies)

Nếu bạn chạy trực tiếp lệnh gỡ bỏ Driver chính:

```bash
sudo modprobe -r cdrom

```

Hệ thống sẽ báo lỗi do mô-đun `cdrom` đang bị chiếm dụng và sử dụng bởi một mô-đun con khác là `sr_mod` (Driver của ổ đĩa quang SCSI). Do đó, bạn phải gỡ mô-đun con trước:

```bash
# Gỡ bỏ driver phụ thuộc trước
sudo rmmod sr_mod

# Gỡ bỏ driver cdrom chính sau khi không còn bị chiếm dụng
sudo modprobe -r cdrom

```

*(Sau khi chạy xong, lệnh `lsmod | grep cdrom` sẽ không còn trả về kết quả).*

#### Bước 5: Nạp lại các mô-đun vào hệ thống

```bash
# Nạp lại driver cdrom chính
sudo modprobe cdrom

# Nạp lại driver sr_mod phụ thuộc
sudo modprobe sr_mod

```

---

*Ghi chú: Cơ chế tự động giải quyết các thành phần phụ thuộc (Dependencies) của lệnh `modprobe` là một trong những điểm lý thuyết quan trọng nhất cần ghi nhớ cho kỳ thi LPIC-1.*
