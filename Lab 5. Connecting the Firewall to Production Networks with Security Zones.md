# Connecting the Firewall to Production Networks with Security Zones

## 1. Lab Overview
Để chuẩn bị cho việc triển khai thực tế,  cần kết nối tường lửa Palo Alto Networks vào mạng sản xuất (production networks) một cách an toàn và tối ưu. Các giao diện vật lý của tường lửa đã được đấu nối vật lý vào các cổng switch tương ứng trong trung tâm dữ liệu. 

Trong bài thực hành này,  sẽ cấu hình tường lửa với các địa chỉ IP Layer 3 và thiết lập một thiết bị định tuyến logic (Logical Router/Virtual Router).  cũng sẽ tạo các vùng bảo mật (Security Zones) để phân đoạn mạng sản xuất thành các vùng logic riêng biệt nhằm kiểm soát, giám sát và thực thi chính sách bảo mật chặt chẽ đối với các luồng lưu lượng di chuyển xuyên vùng. Cuối cùng,  sẽ tạo các Interface Management Profiles để quản lý quyền truy cập và kiểm tra tính kết nối thông suốt giữa tường lửa và các máy chủ trong từng vùng bảo mật.![[Pasted image 20260629215655.png]]

## 2. Lab Objectives
Sau khi hoàn thành bài thực hành này,  sẽ đạt được các mục tiêu sau:
* **Create Layer 3 interfaces:** Cấu hình các giao diện mạng mạng vật lý hoạt động ở chế độ Layer 3 và gán địa chỉ IP tương ứng.
* **Create a Logical router:** Khởi tạo và thiết lập cấu hình bảng định tuyến logic để điều phối luồng dữ liệu.
* **Segment your production network using security zones:** Sử dụng các vùng bảo mật để phân đoạn hạ tầng mạng một cách khoa học.
* **Test connectivity from firewall to hosts in each security zone:** Đăng nhập CLI kiểm tra tính kết nối mạng tới các máy chủ đích.
* **Create Interface Management Profiles:** Thiết lập hồ sơ quản lý giao diện để kiểm soát lưu lượng quản trị/ping dịch vụ một cách an toàn.

---
## 3. High-Level Lab Steps
### 3.1. Create Layer 3 Network Interfaces
Khởi tạo và cấu hình các giao diện Layer 3 với thông số chi tiết theo bảng tiêu chuẩn kỹ thuật dưới đây:

| Giao diện (Interface) | Ô ghi chú (Comment)           | Loại cổng (Type) | IPv4 Type | Địa chỉ IPv4 (IP Address) |
| :-------------------- | :---------------------------- | :--------------- | :-------- | :------------------------ |
| **ethernet1/1**       | `Internet connection`         | Layer 3          | Static    | `203.0.113.20/24`         |
| **ethernet1/2**       | `Users network connection`    | Layer 3          | Static    | `192.168.1.1/24`          |
| **ethernet1/3**       | `Extranet servers connection` | Layer 3          | Static    | `192.168.50.1/24`         |

### 3.2. Create a Logical Router
Khởi tạo một Logical Router để thiết lập tuyến đường mặc định hướng lưu lượng ra Internet:
* **Name:** `LR-1`
* **Interfaces Linked:** `ethernet1/1`, `ethernet1/2`, `ethernet1/3`
* **IPv4 Static Route:**
  * **Name:** `Firewall-Default-Gateway`
  * **Destination:** `0.0.0.0/0`
  * **Interface:** `ethernet1/1`
  * **Next Hop:** `IP Address`
  * **Next Hop IP:** `203.0.113.1`

### 3.3. Segment Your Production Network Using Security Zones
Phân tách hạ tầng sản xuất thành ba phân vùng độc lập bằng cách thiết lập các Security Zones:
* **Zone Name:** `Internet` | **Type:** `Layer 3` | **Interface:** `ethernet1/1`
* **Zone Name:** `Users_Net` | **Type:** `Layer 3` | **Interface:** `ethernet1/2`
* **Zone Name:** `Extranet` | **Type:** `Layer 3` | **Interface:** `ethernet1/3`

### 3.4. Define and Apply Interface Management Profiles
Khởi tạo bộ hồ sơ quản lý nhằm điều phối dịch vụ quản trị trên các giao diện mạng:
* **Profile Name:** `Allow-ping` 
 Dịch vụ cho phép: `Ping` 
 Áp dụng cho: `ethernet1/1`
* **Profile Name:** `Allow-mgt` 
 Dịch vụ cho phép: `HTTPS`, `SSH`, `Ping` 
 Áp dụng cho: `ethernet1/2`, `ethernet1/3`

---

## 4. Detailed Lab Steps

### Phần 4.1: Create Layer 3 Network Interfaces
Quy trình cấu hình các giao diện logic Layer 3 trên thiết bị để liên kết trực tiếp tới ba mạng sản xuất độc lập bao gồm mạng người dùng nội bộ (`192.168.1.0/24`), mạng máy chủ Extranet (`192.168.50.0/24`) và mạng kết nối với cổng nhà mạng Upstream Internet (`203.0.113.0/24`).
![[Pasted image 20260629220503.png]]
#### Task 1: Configure Layer 3 Interface on ethernet1/1 (Internet)
1. Trên thanh công cụ WebUI của tường lửa, chọn mục **Network > Interfaces > Ethernet**.
2. Tìm đến cổng vật lý và nhấp chuột vào liên kết tên cổng **`ethernet1/1`**.![[Pasted image 20260629220923.png]]
3. Tại trường thông tin cấu hình **Comment**, nhập chuỗi ký tự: `Internet connection`.
4. Tại mục **Interface Type**, nhấp mũi tên xổ xuống và chọn giá trị: **`Layer3`**.![[Pasted image 20260629221022.png]]
5. Giữ nguyên toàn bộ cấu hình mặc định khác tại tab *Config* hiện tại và không đóng cửa sổ.
6. Nhấp chuột chuyển sang tab cấu hình mạng **IPv4**.
7. Giữ nguyên tùy chọn tại trường thông tin *Type* là **`Static`**.
8. Tại khu vực quản lý địa chỉ danh sách IP, nhấp chọn nút bấm **Add** ở góc dưới.
9. Nhập chính xác địa chỉ IP và subnet mask: **`203.0.113.20/24`** *(Lưu ý: Hãy chắc chắn ghi đầy đủ phần định danh mạng `/24`)*.![[Pasted image 20260629221049.png]]
10. Nhấp chọn nút **OK** để lưu lại tạm thời.

#### Task 2: Configure Layer 3 Interface on ethernet1/2 (Users_Net)
11. Tại danh sách giao diện mạng Ethernet, tìm và nhấp chuột vào liên kết tên cổng **`ethernet1/2`**
12. Tại trường thông tin cấu hình **Comment**, nhập chuỗi ký tự: `Users network connection`.
13. Tại mục **Interface Type**, chọn giá trị: **`Layer3`**.![[Pasted image 20260629221127.png]]
14. Nhấp chuột chuyển sang tab cấu hình mạng **IPv4**.
15. Nhấp chọn nút bấm lệnh **Add** tại khu vực thiết lập địa chỉ IP.
16. Nhập chính xác thông số địa chỉ IP: **`192.168.1.1/24`** *(Lưu ý: Đảm bảo bao gồm ký tự định danh lớp mạng `/24`)*.![[Pasted image 20260629221138.png]]
17. Nhấp chọn nút **OK**.

#### Task 3: Configure Layer 3 Interface on ethernet1/3 (Extranet)
18. Tại danh sách giao diện mạng Ethernet, tìm và nhấp chuột vào liên kết tên cổng **`ethernet1/3`**.
19. Tại trường thông tin cấu hình **Comment**, nhập chuỗi ký tự: `Extranet servers connection`.
20. Tại mục **Interface Type**, chọn giá trị: **`Layer3`**.
21. Nhấp chuột chuyển sang tab cấu hình mạng **IPv4**.![[Pasted image 20260629221223.png]]
22. Nhấp chọn nút bấm lệnh **Add** tại khu vực thiết lập địa chỉ IP.
23. Nhập chính xác thông số địa chỉ IP: **`192.168.50.1/24`** *(Lưu ý: Đảm bảo bao gồm ký tự định danh lớp mạng `/24`)*.![[Pasted image 20260629221238.png]]
24. Nhấp chọn nút **OK** để hoàn thành cấu hình phần giao diện.

---

### Phần 4.2: Create a Logical Router
Tạo lập một Logical Router (Virtual Router) nhằm thiết lập sơ đồ liên kết và điều phối định tuyến cho ba giao diện mạng vừa tạo, đồng thời thiết lập Default Gateway hướng ra bên ngoài Internet.

25. Trên thanh điều hướng menu của tường lửa, chọn mục **Network > Routing > Logical Routers** (hoặc *Virtual Routers* tùy thuộc vào phiên bản OS cài đặt).
26. Bấm nút **Add** ở phía dưới màn hình để tạo mới một Router logic.
27. Tại ô nhập liệu **Name**, nhập chính xác chuỗi ký tự: **`LR-1`**.
28. Di chuyển xuống khu vực quản lý danh sách giao diện kết nối **Interfaces** tại tab *General*, nhấp chọn nút bấm lệnh **Add**.
29. Từ danh sách xổ xuống, chọn cổng **`ethernet1/1`**.![[Pasted image 20260629221346.png|383]]
30. Bấm nút **Add** lần thứ hai, chọn tiếp cổng mạng **`ethernet1/2`**.
31. Bấm nút **Add** lần thứ ba, chọn cổng mạng còn lại **`ethernet1/3`**.![[Pasted image 20260629221507.png|382]]
32. Quan sát cây menu phân hệ quản lý ở phía bên trái cửa sổ cấu hình, tìm và nhấp chuột chọn liên kết mục **Static**.
33. Đảm bảo  đang đứng ở tab con cấu hình mạng **IPv4**, di chuyển xuống phía góc dưới giao diện và bấm nút lệnh **Add**.
34. Trong cửa sổ thiết lập tuyến đường tĩnh (*Static Route*), cấu hình đầy đủ các thông số chi tiết:
    * **Name:** `Firewall-Default-Gateway`
    * **Destination:** `0.0.0.0/0`
    * **Interface:** Chọn cổng mạng hướng ra ngoài **`ethernet1/1`**
    * **Next Hop:** Nhấp mũi tên chọn kiểu giá trị là **`IP Address`**
    * **Next Hop IP (Hộp văn bản xuất hiện ở dưới):** Nhập địa chỉ IP Gateway nhà mạng: **`203.0.113.1`**![[Pasted image 20260629221854.png]]
35. Nhấp chọn nút **OK** trên cửa sổ con thiết lập tuyến đường tĩnh.
36. Tiếp tục nhấp chọn nút **OK** trên cửa sổ cấu hình Logical Router chính để lưu cấu hình router `LR-1`.

---

### Phần 4.3: Segment Your Production Network Using Security Zones
Thiết lập quy tắc phân đoạn hạ tầng sản xuất thông qua việc khởi tạo ba Vùng bảo mật (Security Zones) Layer 3 riêng biệt và gán các cổng tương ứng vào từng Zone nhằm kiểm soát chặt chẽ luồng thông tin di chuyển chéo.![[Pasted image 20260629222452.png]]

37. Trên thanh công cụ điều hướng menu, chọn phân mục **Network > Zones**.![[Pasted image 20260629222526.png]]
38. Bấm nút **Add** ở phía bên dưới giao diện đồ họa.
39. Tại cửa sổ cấu hình Zone bảo mật, tiến hành thiết lập vùng mạng ngoài đầu tiên:
    * **Name:** **`Internet`** *(Lưu ý: Hệ thống phân biệt chữ hoa và chữ thường, vui lòng nhập chính xác chữ I viết hoa)*.
    * **Type:** Nhấp chọn giá trị **`Layer 3`**.
40. Di chuyển xuống khu vực danh sách cổng liên kết mạng *Interfaces*, nhấp nút lệnh **Add**.
41. Từ danh sách hiển thị, tích chọn cổng vật lý **`ethernet1/1`**.
42. Nhấp chọn nút **OK** để lưu lại vùng Internet.![[Pasted image 20260629222615.png|370]]
43. Tiếp tục bấm nút lệnh **Add** để thiết lập cấu hình phân vùng người dùng nội bộ:
    * **Name:** **`Users_Net`**.
    * **Type:** Chọn giá trị **`Layer 3`**.
44. Tại khu vực *Interfaces*, nhấp nút **Add** và gán cổng vật lý tương ứng là **`ethernet1/2`**.![[Pasted image 20260629222724.png|372]]
45. Nhấp chọn nút **OK**.
46. Bấm nút lệnh **Add** lần thứ ba để cấu hình phân vùng mạng chứa cụm máy chủ công cộng:
    * **Name:** **`Extranet`**.
    * **Type:** Chọn giá trị **`Layer 3`**.
47. Tại khu vực *Interfaces*, nhấp nút **Add** và chọn liên kết cổng vật lý là **`ethernet1/3`**.![[Pasted image 20260629222815.png]]
48. Nhấp chọn nút **OK** để hoàn thành việc phân đoạn các vùng mạng bảo mật.

---

### Phần 4.4: Commit và Kiểm tra kết nối mạng đến từng Zone (Test Connectivity to Each Zone)
Thực hiện lưu cấu hình chính thức và sử dụng giao diện dòng lệnh (CLI) của thiết bị tường lửa để kiểm tra khả năng giao tiếp gói tin ICMP thông suốt tới từng máy chủ ở các Zone.

49. Nhấp chọn liên kết nút lệnh **Commit** nằm ở phía góc trên cùng bên phải màn hình WebUI đồ họa của tường lửa.
50. Giữ nguyên mọi thuộc tính cấu hình hệ thống mặc định và tiếp tục bấm nút lệnh **Commit** ở hộp thoại mới xuất hiện.
51. Chờ đợi cho tới khi hệ thống thực thi nạp hoàn tất đạt mốc 100%, nhấn nút **Close** để đóng hộp thoại báo cáo.
52. Trên giao diện màn hình nền máy trạm Client-A của người dùng, tìm và khởi chạy ứng dụng kết nối từ xa mạng **Remmina**.
53. Nhấp đúp chuột vào profile kết nối đã cấu hình sẵn mang tên **`Firewall-A`** để khởi tạo phiên làm việc bảo mật qua giao thức SSH kết nối vào CLI của tường lửa.
54. Khi màn hình CLI hiển thị dấu nhắc lệnh quản trị hệ thống dạng `admin@firewall-a>`, tiến hành thực hiện lệnh ping kiểm tra kết nối xuyên vùng tới máy trạm thuộc vùng người dùng `Users_Net`:
    ```bash
    ping source 192.168.1.1 host 192.168.1.20
    ```
    *(Để lệnh gửi nhận gói tin chạy ổn định trong vòng 3-4 giây, sau đó nhấn tổ hợp phím **Ctrl+C** trên bàn phím để dừng lệnh)*.
55. Tiếp tục thực hiện nhập lệnh ping kiểm tra kết nối tới máy chủ cơ sở dữ liệu thuộc vùng máy chủ `Extranet`:
    ```bash
      ping source 192.168.50.1 host 192.168.50.150
    ```
    *(Theo dõi phản hồi và nhấn tổ hợp phím **Ctrl+C** để dừng thực thi lệnh)*.
56. Thực hiện nhập lệnh ping kiểm tra tuyến đường định tuyến hướng ra Internet ra máy chủ DNS công cộng bên ngoài:
    ```bash
    ping source 203.0.113.20 host 8.8.8.8
    ```
    *(Nhấn tổ hợp phím **Ctrl+C** để ngắt tiến trình kiểm tra sau khi xác nhận các gói tin được gửi nhận thành công 100%)*.

---

### Phần 4.5: Test Interface Access before Management Profiles
Xác minh cơ chế bảo mật cốt lõi mặc định của hệ điều hành PAN-OS: Từ chối toàn bộ mọi lưu lượng ping dịch vụ hoặc cố gắng kết nối quản trị trực tiếp vào các cổng mạng Layer 3 của thiết bị khi chưa được cấp quyền qua hồ sơ quản lý.

57. Tìm và mở ứng dụng dòng lệnh nội bộ **Terminal** có sẵn trên màn hình máy trạm quản trị Client-A.
58. Thực hiện chạy lệnh ping trực tiếp từ máy trạm đến địa chỉ IP cổng gateway của tường lửa trên giao diện `ethernet1/2`:
    ```bash
    ping 192.168.1.1
    ```
    *Kết quả thực tế:* Màn hình Terminal sẽ treo hoặc hiển thị trạng thái không nhận được bất kỳ phản hồi trả về nào từ thiết bị (Nhấn phím **Ctrl+C** để chủ động dừng lệnh).
59. Tiếp tục thực hiện lệnh kết nối từ xa SSH quản trị vào IP giao diện này của tường lửa:
    ```bash
    ssh admin@192.168.1.1
    ```
    *Kết quả thực tế:* Phiên kết nối sẽ bị từ chối hoặc báo lỗi Time out (Connection timed out / Connection refused). Nhấn tổ hợp phím **Ctrl+C** để chấm dứt kết nối lỗi và giữ nguyên trạng thái cửa sổ ứng dụng Terminal này trên màn hình.

---

### Phần 4.6: Create and Apply Interface Management Profiles
Khởi tạo các bộ quy tắc cấu hình Interface Management Profiles nhằm kích hoạt và cho phép bộ phận vận hành SecOps có quyền sử dụng công cụ ping kiểm tra thiết bị, đồng thời kích hoạt cổng quản trị an toàn trên các giao diện mạng sản xuất thích hợp.

#### Task 1: Define Interface Management Profiles
60. Quay trở lại giao diện đồ họa WebUI của tường lửa, di chuyển chuột chọn danh mục quản trị **Network > Network Profiles > Interface Mgmt**.
61. Nhấp chọn nút bấm **Add** nằm ở phía dưới cùng màn hình giao diện.
62. Tại ô cấu hình **Name**, nhập tên hồ sơ: **`Allow-ping`**.
63. Di chuyển xuống vùng lựa chọn dịch vụ mạng mạng *Network Services*, tìm và tích chọn vào ô kiểm: **`Ping`**. Giữ nguyên các thông số bảo mật khác rồi nhấp chọn **OK**.![[Pasted image 20260629232833.png]]
64. Bấm nút lệnh **Add** một lần nữa tại menu chính để khởi tạo hồ sơ quản lý thứ hai.
65. Tại ô cấu hình **Name**, nhập tên hồ sơ: **`Allow-mgt`**.
66. Di chuyển đến phân vùng quản lý các dịch vụ quản trị hệ thống chuyên sâu *Administrative Management Services*, tiến hành tích chọn vào hai ô kiểm: **`HTTPS`** và **`SSH`**.
67. Di chuyển xuống phân vùng dịch vụ mạng *Network Services* ở phía dưới, tìm và tích chọn vào ô kiểm: **`Ping`**, **`SNMP`**. Nhấp chọn **OK** để lưu lại hồ sơ quản trị nâng cao.![[Pasted image 20260629232929.png]]

#### Task 2: Apply Management Profiles to Interfaces
68. Di chuyển chuột chọn phân mục **Network > Interfaces > Ethernet** trên thanh công cụ điều hướng.
69. Nhấp chọn vào liên kết tên cổng mạng **`ethernet1/1`** (Cổng hướng mạng Internet).
70. Tại cửa sổ thiết lập thuộc tính, nhấp chuột chọn chuyển sang tab cấu hình nâng cao **Advanced**.
71. Tại phân vùng quản lý thông tin mở rộng *Other Info*, tìm kiếm trường dữ liệu **Management Profile**, nhấp chuột vào mũi tên xổ xuống và chỉ định gán hồ sơ: **`Allow-ping`**. ![[Pasted image 20260629233345.png]]Nhấp chọn **OK**.
72. Quay lại bảng danh sách tổng, nhấp chọn vào liên kết cổng mạng **`ethernet1/2`** (Cổng mạng Users nội bộ).
73. Chuyển sang tab nâng cao **Advanced**, tìm đến mục thông tin cấu hình trường **Management Profile** tại phân vùng *Other Info* và gán liên kết hồ sơ quản trị cao cấp: **`Allow-mgt`**. Nhấp chọn **OK**,![[Pasted image 20260629233421.png]] nếu hệ thống hiển thị bảng cảnh báo an toàn mạng, nhấp chọn **Yes** để xác nhận đồng ý áp dụng.
74. Tiếp tục nhấp chọn cổng mạng vật lý **`ethernet1/3`** (Cổng Extranet).
75. Chuyển sang tab nâng cao **Advanced**, tại mục thông tin trường dữ liệu **Management Profile** thuộc phân vùng *Other Info*, tiến hành gán liên kết hồ sơ: **`Allow-mgt`**. Nhấp chọn **OK** và chọn **Yes** nếu xuất hiện hộp thoại thông báo xác nhận cảnh báo hệ thống.![[Pasted image 20260629233506.png]]

---

### Phần 4.7: Commit và Kiểm tra quyền truy cập giao diện mạng sau cấu hình (Test Interface Access after Management Profiles)
Tiến hành nạp đồng bộ dữ liệu cấu hình mới và kiểm thử lại toàn bộ các dịch vụ mạng cũng như quyền hạn quản trị CLI từ máy trạm Client-A vào cổng Layer 3 của tường lửa.

76. Nhấp chọn liên kết nút lệnh **Commit** nằm ở góc trên cùng bên phải màn hình WebUI, giữ nguyên các thông số mặc định hệ thống rồi nhấn tiếp nút **Commit**. Đợi thanh tiến trình xử lý đạt trạng thái hoàn thành công việc 100%, nhấn nút **Close** để hoàn tất.
77. Quay trở lại cửa sổ ứng dụng dòng lệnh chuyên dụng **Terminal** đang mở sẵn trên màn hình máy trạm Client-A.
78. Thực hiện nhập và chạy lại lệnh kiểm tra gói tin ping tới cổng gateway nội bộ của tường lửa:
    ```bash
    ping 192.168.1.1
    ```
    *Kết quả kiểm tra thực tế:* Cổng giao diện mạng Layer 3 của tường lửa bây giờ **đã phản hồi thành công** các gói tin ICMP một cách liên tục và thông suốt! (Nhấn tổ hợp phím **Ctrl+C** trên bàn phím để dừng lệnh sau khi theo dõi).
79. Tiếp tục thực hiện nhập lại lệnh khởi tạo phiên làm việc SSH kết nối quản trị hệ thống vào IP giao diện mạng Layer 3 này:
    ```bash
    ssh admin@192.168.1.1
    ```
    *Kết quả kiểm tra thực tế:* Tường lửa Palo Alto Networks lập tức chấp nhận yêu cầu xử lý phiên kết nối từ xa. 
    * Nếu hệ thống hiển thị thông báo yêu cầu người dùng xác nhận tính hợp lệ của mã vân tay khóa bảo mật RSA (*key fingerprint*), hãy nhập chuỗi ký tự **`yes`** từ bàn phím và nhấn Enter.
80. Khi hệ thống hiển thị yêu cầu nhập mật khẩu bảo mật đăng nhập quản trị, tiến hành điền mật khẩu: **`Pal0Alt0!`** (Mật khẩu khi nhập sẽ được ẩn để bảo mật an toàn thông tin). Nhấn Enter,  sẽ truy cập thành công hoàn toàn vào giao diện dòng lệnh CLI quản trị nội tại của hệ điều hành mạng PAN-OS.
	1. Nhập lệnh **`exit`** để đóng kết thúc phiên làm việc SSH bảo mật hiện tại, và tiếp tục nhập lệnh **`exit`** một lần nữa để tắt hoàn toàn ứng dụng Terminal trên máy trạm.

***