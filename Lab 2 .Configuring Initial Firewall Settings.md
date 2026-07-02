# Configuring Initial Firewall Settings

## Tổng quan bài Lab (Lab Overview)
Tổ chức của  vừa nhận được một thiết bị tường lửa Palo Alto Networks mới và  được giao nhiệm vụ triển khai thiết bị này. Các bước đầu tiên sẽ là kết nối với địa chỉ giao diện quản trị (management interface) của tường lửa và cấu hình các thiết bị cơ bản để cấp quyền truy cập mạng cho tường lửa.
![[Pasted image 20260627205510.png]]
Đã thay đổi. 
- MGT: 192.168.1.200
- Client-A: 192.168.1.21 
## Mục tiêu bài Lab (Lab Objectives)
* Kết nối với giao diện web quản trị của tường lửa (firewall web interface)
* Thiết lập máy chủ DNS cho tường lửa
* Thiết lập máy chủ NTP cho tường lửa
* Cấu hình biểu ngữ đăng nhập (login banner) cho tường lửa
* Thiết lập Vĩ độ (Latitude) và Kinh độ (Longitude) cho tường lửa
* Cấu hình các địa chỉ IP được phép quản trị tường lửa (permitted IP addresses)

---

## Các bước thực hiện ở mức cao (High-Level Lab Steps)

### 1. Kết nối tới Tường lửa Sinh viên (Connect to Your Student Firewall)
* Sử dụng trình duyệt cấu hình để kết nối với giao diện web của tường lửa.

### 2. Cấu hình Máy chủ DNS và NTP (Configure the DNS and NTP Servers)
* Đặt **Primary DNS Server** thành `8.8.8.8` và **Secondary DNS Server** thành `192.168.50.53`.
* Đặt **Primary NTP Server** thành `0.pool.ntp.org` và **Secondary NTP Server** thành `1.pool.ntp.org`.

### 4. Cấu hình Cài đặt Chung (Configure General Settings)
* Thiết lập Domain thành `corp.company.com`.
* Tạo một Login Banner hiển thị nội dung: `Authorized Access Only`.
* Đặt Latitude và Longitude để phản ánh vị trí địa lý của tường lửa tại `HN 21.0285,  105.8542.`

### 5. Chỉnh sửa Giao diện Quản trị (Modify Management Interface)
* Xác minh rằng cổng mặc định (default gateway) cho giao diện quản trị của tường lửa được đặt thành `192.168.1.1`.
* Chỉ cho phép truy cập vào giao diện quản trị từ mạng `192.168.1.0/24`.

### 6. Lưu cấu hình (Commit the Configuration)
* Thực hiện Commit các thay đổi trước khi tiếp tục.

### 7. Kiểm tra phần mềm PAN-OS mới (Check for New PAN-OS Software)
* Kiểm tra các phiên bản phần mềm PAN-OS mới (nhưng không thực hiện nâng cấp tường lửa).

---

## Các bước thực hiện chi tiết (Detailed Lab Steps)

### Phần 1: Kết nối tới Tường lửa 
1. Khởi chạy trình duyệt cấu hình (Configuration Browser - khuyến nghị sử dụng Chromium) và kết nối tới địa chỉ:  
   `https://192.168.1.200`
2. Bỏ qua các cảnh báo bảo mật của trình duyệt (security warnings) cho đến khi  thấy cửa sổ đăng nhập giao diện web.
3. Đăng nhập vào tường lửa Palo Alto Networks bằng thông tin xác thực sau:
   * **Username:** `admin`
   * **Password:** `admin`

### Phần 2: Cấu hình Máy chủ DNS và NTP
Các cài đặt máy chủ DNS được sử dụng cho tất cả các truy vấn DNS mà tường lửa khởi tạo nhằm hỗ trợ cho các đối tượng địa chỉ FQDN, ghi nhật ký (logging) và quản trị tường lửa.
1. Trên giao diện web, chọn **Device** > **Setup** > **Services**.
2. Nhấp vào biểu tượng bánh răng (gear icon) của phần **Services** để mở cửa sổ cấu hình.
3. Xác minh hoặc thiết lập **Primary DNS Server** thành `8.8.8.8`.
4. Đặt **Secondary DNS Server** thành `192.168.50.53`.
			![[Pasted image 20260627210558.png|402]]
5. Chuyển sang tab **NTP** (nếu có) hoặc tìm vùng cấu hình NTP:
   * Đặt **Primary NTP Server** thành `0.pool.ntp.org`.
   * Đặt **Secondary NTP Server** thành `1.pool.ntp.org`.
     ![[Pasted image 20260627211635.png|591]]
   
6. Giữ nguyên các cài đặt còn lại và nhấp **OK** để đóng cửa sổ *Services*.

### Phần 3: Cấu hình Cài đặt Chung (General Settings)
1. Chọn **Device** > **Setup** > **Management**.
2. Nhấp vào biểu tượng bánh răng trong bảng **General Settings** để mở cửa sổ cấu hình.
3. Tại trường **Hostname**, nhập: `FW-HN-EDGE-01`
4. Tại trường **Domain**, nhập: `corp.company.com`
5. Tại vùng **Login Banner**, nhập: `Authorized Access Only`
6. Tại vùng **Time Zone**, nhập: `Asia/Saigon`
7. Tại trường **Latitude**, nhập: `21.0285`
8. At trường **Longitude**, nhập: `105.8542`  
   *(Lưu ý: Asia/Saigon.)*
9. Giữ nguyên các cài đặt khác và nhấp **OK** để đóng cửa sổ *General Settings*.
![[Pasted image 20260627211734.png]]

### Phần 4: Chỉnh sửa Giao diện Quản trị (Management Interface)
1. Chọn **Device** > **Setup** > **Interfaces**.
2. Nhấp vào liên kết có tên **Management** để mở cửa sổ *Management Interface Settings*.
3. Xác minh hoặc thiết lập **Default Gateway** thành `192.168.1.1`.
4. Giữ nguyên các cài đặt mạng khác.
![[Pasted image 20260627212831.png]]

5. Tại phần phía dưới của vùng **Permitted IP Addresses**, nhấp **Add**.
6. Trong trường nhập địa chỉ, điền mạng quản trị được phép: `192.168.1.0/24`
7. Trong trường Description, nhập `Mgt access from these hosts only`
![[Pasted image 20260627213021.png]]
8. Nhấp **OK** để ghi nhận và đóng lại.
	Chắc chắn nhập giải quản lý đúng, nếu không có thể mất kết nối mạng tới fw.
	
### Phần 5: Lưu cấu hình (Commit Configuration)
1. Nhấp vào nút **Commit** ở góc trên cùng bên phải của giao diện web.
2. Giữ nguyên các tùy chọn và nhấp tiếp **Commit**.
3. Chờ cho đến khi tiến trình Commit đạt 100% và hiển thị kết quả thành công.
4. Nhấp **Close** để hoàn tất bài Lab.

---
