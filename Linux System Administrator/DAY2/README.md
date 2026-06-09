## 📅 Bài 02: Cài đặt cấu hình VMware và Linux CentOS 8 (Phần 1)

Mục tiêu bài học nhằm xây dựng môi trường lab ảo hóa chuẩn hóa, phân chia phân vùng thủ công đáp ứng các tiêu chuẩn triển khai hạ tầng thực tế.

### 🛠️ Quy Trình Triển Khai Lab

#### 1. Cài đặt phần mềm ảo hóa VMware Workstation Player
* Tải xuống phiên bản mới nhất từ trang chủ `vmware.com` (Khuyến nghị bản 17.x dành cho Windows 64-bit).
* Cài đặt bằng quyền Quản trị viên (`Run as administrator`), chấp nhận các điều khoản giấy phép và giữ cấu hình mặc định.

#### 2. Tải File ISO Hệ Điều Hành
* Truy cập trang chủ `centos.org` -> mục Download -> Chọn phiên bản **CentOS Stream 8 (x86_64)**.
* Tải xuống từ các máy chủ Mirror nội địa Việt Nam để tối ưu hóa tốc độ tải file `CentOS-Stream-8-x86_64-dvd1.iso`.

#### 3. Thiết Kế Cấu Hình Phần Cứng Máy Ảo (Virtual Machine Settings)
Để hệ thống CentOS 8 Stream chạy ổn định và phục vụ tốt cho các bài thực hành chuyên sâu về lưu trữ/mạng, cấu hình phần cứng ảo được thiết lập như sau:

| Linh kiện ảo | Thông số cấu hình | Ghi chú / Mục đích sử dụng |
| :--- | :--- | :--- |
| **RAM (Memory)** | **4 GB** (4024 MB) | Dung lượng tối thiểu khuyến nghị cho CentOS 8 GUI |
| **CPU (Processor)**| **2 Cores** | Tối ưu hiệu năng xử lý tác vụ nền hệ thống |
| **Hard Disk 1** | **20 GB** (SATA, Single File) | Phân vùng chính dùng để cài đặt Hệ điều hành |
| **Hard Disk 2** | **1 GB** (SATA, Single File) | Ổ đĩa phụ phục vụ bài lab quản lý phân vùng đĩa (Fdisk/LVM) |
| **Hard Disk 3** | **1 GB** (SATA, Single File) | Ổ đĩa phụ phục vụ bài lab quản lý RAID và File System |
| **Network Adapter 1**| **NAT** | Cho phép máy ảo truy cập Internet đi qua IP máy vật lý |
| **Network Adapter 2**| **Bridged** | Máy ảo nhận IP trực tiếp từ Router, cùng lớp mạng với máy thật |
| **CD/DVD Drive** | Gắn file ISO CentOS 8 | Cung cấp bộ cài đặt nguồn cho máy ảo |

> ⚠️ **Lưu ý quan trọng:** Tại bước cấu hình vị trí lưu trữ máy ảo (`Location`), hãy nhấn **Browse** để chuyển đường dẫn lưu trữ từ ổ mặc định `C:\` sang ổ đĩa dữ liệu khác (`D:\`, `E:\`...) nhằm tránh mất dữ liệu máy ảo khi cài đặt lại hệ điều hành Windows máy thật.

#### 4. Phân Chia Phân Vùng Ổ Cứng Thủ Công (Manual Partitioning)
Khi đến bước **Installation Destination**, chọn ổ cứng **20 GB** và tích chọn cấu hình **Custom**. Tại giao diện thủ công, chuyển từ cấu hình *LVM* sang **Standard Partition** và chia các phân vùng (Mount Point) theo tỷ lệ sau:

1. **`/` (Root Directory):** Cấp phát **10 GiB**, định dạng File System là **`ext4`**. (Nơi lưu trữ mã nguồn hệ thống và các ứng dụng chính).
2. **`/boot`:** Cấp phát **1 GiB**, định dạng File System là **`ext4`**. (Chứa tệp tin khởi động và Kernel của Linux).
3. **`/home`:** Cấp phát **3 GiB**, định dạng File System là **`ext4`**. (Không gian lưu trữ dữ liệu cá nhân của các người dùng thường).
4. **`/var`:** Cấp phát **4 GiB**, định dạng File System là **`xfs`**. (Chứa dữ liệu biến đổi theo thời gian như log hệ thống, hàng đợi mail, database).
5. **`swap`:** Cấp phát **2 GiB** (Bằng 1/2 lượng RAM thật). (Bộ nhớ đệm trên ổ cứng hỗ trợ khi hệ thống bị quá tải RAM).

#### 5. Thiết Lập Hệ Thống Ban Đầu & Cấu Hình Mạng
* **Tài khoản:** Đặt mật khẩu cho tài khoản `root` và tạo thêm một tài khoản người dùng thường để đăng nhập.
* **Múi giờ:** Chọn Region: **Asia** / City: **Ho Chi Minh** để đồng bộ thời gian chuẩn.
* **Kích hoạt Card mạng tự động:**
  * Mặc định sau khi cài xong, các card mạng ở trạng thái Tắt (Off). Vào **Wired Settings**, gạt bật **On** cho cả 2 card mạng (`ens160` và `ens192`).
  * Nhấn vào biểu tượng bánh răng cài đặt của từng card -> Tại tab *Identity*, tích chọn ô **Connect automatically** để hệ thống tự động kết nối mạng mỗi khi khởi động máy ảo.
* **Kiểm tra kết nối:** Mở ứng dụng **Terminal** và chạy lệnh:
  ```bash
  ping google.com
