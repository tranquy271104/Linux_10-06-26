## 📅 Bài 06: Quản trị thiết bị phần cứng Linux

Mục tiêu bài học nhằm giúp học viên hiểu rõ cơ chế phát hiện, giao tiếp phần cứng của hệ điều hành Linux thông qua các dịch vụ hệ thống ảo (`udev`, `dbus`), bản chất của thư mục thiết bị `/dev` và thành thạo các câu lệnh điều tra thông tin phần cứng cho kỳ thi LPIC-1.

---

### 🧠 1. Cơ Chế Hoạt Động & Quản Lý Thiết Bị Phần Cứng

Khi một thiết bị phần cứng mới (ví dụ: ổ cứng, USB) được kết nối vào máy chủ Linux, hệ thống sẽ xử lý thông tin dựa trên kiến trúc luồng dữ liệu sau:


1. **`udev` (Device Manager):** Là trình quản lý thiết bị dành cho hạt nhân Linux Kernel. `udev` lắng nghe các sự kiện phần cứng ở thời gian thực, tự động nhận diện thiết bị và ánh xạ chúng thành các tệp tin đại diện nằm trong hệ thống tệp tin ảo `/dev`.
2. **`dbus` (Desktop Bus):** Là dịch vụ trung gian truyền thông điệp (Inter-process communication). Khi `udev` phát hiện thiết bị mới, nó sẽ thông qua `dbus` để gửi tín hiệu, thông báo trạng thái này cho toàn bộ các ứng dụng khác và người dùng trong hệ thống biết.
3. **Thư mục `/dev` (Device File System):** * Đây là một hệ thống tệp tin ảo chứa các tệp xử lý đặc biệt (Device Nodes) đại diện cho mọi phần cứng thực tế.
  * Khác với `/proc` hay `/sys` (chứa dữ liệu văn bản để đọc), các tệp tin trong `/dev` được lưu trữ dưới dạng nhị phân (**Binary**). Do đó, con người không thể đọc trực tiếp các tệp này bằng lệnh `cat` mà phải sử dụng các câu lệnh chuyên dụng để giải mã thông tin.

---

### 📂 2. Quy Tắc Đặt Tên Thiết Bị Phổ Biến Trong Thư Mục `/dev`

* `/dev/vga`: Tệp đại diện cho card màn hình (Video Card).
* `/dev/dri`: Thư mục chứa thông tin xử lý đồ họa (Direct Rendering Infrastructure). Nếu chỉ có tệp `card0` tức là máy có 1 card màn hình, có thêm `card1` tức là 2 card.
* `/dev/cpu`: Thư mục quản lý các nhân xử lý của CPU (Ví dụ: gồm thư mục `0` và `1` đại diện cho hệ thống máy ảo cấu hình 2 Cores CPU).
* `/dev/nvme*`: Đại diện cho các ổ cứng thể rắn chuẩn NVMe tốc độ cao.
  * `/dev/nvme0n1`: Ổ đĩa cứng NVMe số 0, phân sụn đĩa số 1.
  * `/dev/nvme0n1p1`, `/dev/nvme0n1p2`: Các phân vùng phân chia (Partition) trên ổ cứng NVMe đó.
* `/dev/bus`: Chứa thông tin về cấu trúc các bo mạch chủ (Mainboard/Bus).

---

### 💻 3. Các Câu Lệnh Điều Tra Phần Cứng Chuyên Dụng (LS-Commands)

Vì hệ thống lưu trữ tệp tin trong `/dev` dưới dạng mã nhị phân, người quản trị hệ thống buộc phải sử dụng chuỗi lệnh bắt đầu bằng ký tự `ls` (List) để dịch thông tin phần cứng sang ngôn ngữ con người có thể hiểu được:

#### A. Lệnh `lsblk` (List Block Devices)
* **Chức năng:** Liệt kê danh sách tất cả các thiết bị lưu trữ dạng khối (Ổ cứng, phân vùng, ổ đĩa quang CD-ROM).
* **Cấu trúc hiển thị:** Tên ổ cứng, kích thước dung lượng (SIZE), kiểu thiết bị (vùng đĩa hay phân vùng) và điểm gắn kết (Mount Point).
* **Tham số mở rộng:**
```bash
# Hiển thị thông tin ổ cứng chi tiết dưới dạng bảng cấu trúc định dạng hệ thống tệp tin
lsblk -f

```

#### B. Lệnh `lspci` (List PCI Devices)

* **Chức năng:** Hiển thị thông tin về toàn bộ các thiết bị kết nối qua cổng khe cắm bus PCI (Card mạng, Card âm thanh, USB Controller, Card đồ họa...).
* **Tham số mở rộng:**
```bash
# Xem tên Driver Kernel (Kernel driver in use) đang giao tiếp trực tiếp với thiết bị PCI đó
lspci -k

# Hiển thị thông tin phần cứng chi tiết, đầy đủ hơn (Verbose mode)
lspci -v

```



#### C. Lệnh `lsusb` (List USB Devices)

* **Chức năng:** Hiển thị danh sách các thiết bị và bộ điều khiển cổng USB (USB Controllers / USB Hubs) đang kết nối với máy tính.
* **Tham số mở rộng:**
```bash
# Hiển thị chi tiết cấu trúc sơ đồ kết nối của các thiết bị USB dưới dạng cấu trúc cây (Tree)
lsusb -t

```



#### D. Lệnh `lscpu` (List CPU)

* **Chức năng:** Hiển thị toàn bộ thông số kỹ thuật chi tiết của bộ vi xử lý CPU (Kiến trúc chip, số lượng nhân Cores, luồng Threads, tên nhà sản xuất như Intel Core / AMD, tốc độ xung nhịp, bộ nhớ đệm Cache...).
```bash
lscpu

```



---

### 🛠️ Kịch Bản Lab Thực Hành Gợi Ý

1. **Kiểm tra thông tin đĩa đệm phân vùng hiện có:**
```bash
lsblk

```


2. **Kiểm tra thông tin card mạng và các thiết bị phần cứng khác gắn trên mainboard máy ảo VMware:**
```bash
lspci -k | grep -i ethernet

```


3. **Thử nghiệm gắn thêm một thiết bị (Ví dụ: USB thật cắm vào máy ảo):**
Chạy lệnh sau trước và sau khi cắm để thấy sự thay đổi driver được `udev` và `dbus` cập nhật tự động:
```bash
lsusb -t

```


4. **Xem thông số chip xử lý Intel ảo hóa cấp phát cho hệ thống:**
```bash
lscpu

```



---

*Ghi chú: Bản chất nhị phân của `/dev`, vai trò điều phối sự kiện của bộ đôi `udev`/`dbus` và ý nghĩa các tham số lệnh `lsblk -f`, `lspci -k` là các phần trọng tâm lý thuyết cần ghi nhớ cho kỳ thi LPIC-1.*
