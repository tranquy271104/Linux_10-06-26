## 📅 Bài 04: Pseudo File Systems là gì?

Mục tiêu bài học nhằm giúp học viên nắm vững khái niệm về hệ thống tệp tin ảo (**Pseudo File System**), cơ chế hoạt động của hạt nhân (Kernel) Linux trong bộ nhớ RAM, và làm chủ hai thư mục cốt lõi xuất hiện trực tiếp trong bài thi LPIC-1: `/proc` và `/sys` [00:02:00].

---

### 🧠 1. Khái Niệm & Bản Chất Của Pseudo File System

* **Hệ thống tệp tin (File System) thông thường:** Là phương pháp và cấu trúc dữ liệu mà hệ điều hành sử dụng để theo dõi, tổ chức các tập tin trên ổ cứng vật lý hoặc các phân vùng (Partition) [00:00:05].
* **Pseudo File System (Hệ thống tệp tin giả/ảo):** * Từ *"Pseudo"* mang ý nghĩa là giả (Fake) hoặc không có thật (Not real) [00:00:53].
  * Hệ thống tệp tin ảo **không tồn tại trên ổ cứng vật lý** [00:00:53].
  * Nó chỉ được sinh ra và **tồn tại duy nhất trong bộ nhớ RAM** khi hệ điều hành đang chạy [00:01:00].
  * **Cơ chế hoạt động:** Khi máy tính tắt hoặc khởi động lại (Reboot), toàn bộ dữ liệu trong hệ thống tệp tin ảo này sẽ bị xóa sạch hoàn toàn (do RAM là bộ nhớ tạm) [00:01:18]. Ngay khi hệ thống khởi động lên, hạt nhân Linux (**Linux Kernel**) sẽ tự động tái tạo lại hệ thống tệp tin ảo này theo thời gian thực để phục vụ nhu cầu giao tiếp hệ thống [00:01:29].

> 💡 **Triết lý cốt lõi của Linux:** *"Everything is a file" (Tất cả mọi thứ đều là tệp tin)*. Trong Linux, từ ổ cứng, card mạng, bàn phím, màn hình cho đến các tiến trình đang chạy và thông tin hạt nhân, tất cả đều được hệ điều hành nhìn nhận và quản lý dưới dạng các tệp tin [00:00:30].

---

### 📂 2. Hai Thư Mục Ảo Cốt Lõi Trong Bài Thi LPIC-1

Mặc dù hệ thống có nhiều tệp tin ảo, lộ trình thi chứng chỉ LPIC-1 (Exam 101) yêu cầu học viên tập trung tối đa vào hai thư mục hệ thống ảo quan trọng sau [00:01:55]:

#### A. Thư mục `/proc` (Process File System)
* **Chức năng:** Chứa thông tin về các tiến trình hệ thống đang chạy thời gian thực (**Running Processes**) và một số thông tin cấu hình phần cứng cốt lõi liên quan [00:02:21].
* **Cấu trúc bên trong:** * Các thư mục được đánh số (Ví dụ: `1`, `29`, `135`...): Mỗi con số đại diện cho một mã định danh tiến trình (**Process ID - PID**), hoạt động tương tự như số chứng minh thư của một tiến trình [00:05:26].
  * Thư mục `/proc/1`: Đại diện cho tiến trình đầu tiên được kích hoạt khi hệ thống khởi động (trong hệ thống hiện đại là **`systemd`**) [00:10:29].
  * Các tệp tin cấu hình dạng văn bản (`.txt`) để xem trực tiếp trạng thái phần cứng hệ thống [00:07:06]:
    * `/proc/cpuinfo`: Chứa chi tiết thông tin vi xử lý CPU (Tên chip, số nhân, xung nhịp...) [00:07:12].
    * `/proc/meminfo`: Chứa chi tiết trạng thái bộ nhớ RAM (Dung lượng trống, dung lượng đã dùng) [00:07:18].
    * `/proc/uptime`: Thời gian hệ thống đã hoạt động liên tục kể từ lần khởi động cuối cùng [00:07:24].
    * `/proc/version`: Phiên bản của hạt nhân Kernel và hệ điều hành Linux đang chạy [00:07:39].
    * `/proc/partitions`: Thông tin về các phân vùng ổ đĩa cứng [00:07:46].

#### B. Thư mục `/sys` (System File System)
* **Chức năng:** Chứa thông tin chuyên sâu và quản lý cấu trúc các thiết bị phần cứng, thiết bị ngoại vi kết nối vào máy tính (Bàn phím, chuột, màn hình, ổ cứng...) cùng các mô-đun hạt nhân (Kernel Modules) [00:02:49].
* **Đặc điểm lý thuyết:** Khác với `/proc`, thư mục `/sys` **không chứa thông tin về các tiến trình (Processes)** [00:02:59]. Nó được phân tách rõ ràng để chuyên biệt quản lý cấu trúc driver và thiết bị phần cứng ảo [00:14:24].

---

### 💻 3. Các Câu Lệnh Thực Hành Hệ Thống Cơ Bản

Trong bài học này, chúng ta sử dụng chuỗi lệnh cơ bản sau để tương tác và điều tra hệ thống tệp tin ảo [00:03:28]:

1. **`man` (Manual):** Tra cứu hướng dẫn và tài liệu chính thức của hệ thống.
   ```bash
   man proc

```

2. **`cd` (Change Directory):** Thay đổi/chuyển đổi thư mục làm việc hiện hành [00:04:17].
```bash
cd /proc

```


3. **`ls` (List):** Liệt kê danh sách các tệp tin và thư mục con [00:04:46].
```bash
ls

```


4. **`cat` (Concatenate):** Đọc và in toàn bộ nội dung của tệp tin văn bản ra màn hình Terminal [00:08:07].
```bash
# Xem thông tin CPU của máy ảo
cat /proc/cpuinfo

# Xem tiến trình nào đang chiếm giữ PID số 1 (Dòng lệnh kích hoạt)
cat /proc/1/cmdline

```


5. **`clear`:** Làm sạch không gian màn hình Terminal để dễ thao tác [00:08:53].
```bash
clear

```



---

*Ghi chú: Hãy ghi nhớ bản chất "Chỉ tồn tại trên RAM theo thời gian thực" và chức năng phân biệt giữa `/proc` và `/sys` để xử lý chính xác các câu hỏi trắc nghiệm trong kỳ thi LPIC-1 [00:01:05].*
