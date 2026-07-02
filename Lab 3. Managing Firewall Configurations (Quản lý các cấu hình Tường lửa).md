# Managing Firewall Configurations (Quản lý các cấu hình Tường lửa)

## 1. Tổng quan bài Lab (Lab Overview)
Sau khi thiết lập tường lửa để cho phép quyền truy cập quản trị, cần đảm bảo rằng mình có thể sao lưu, tải (load) và khôi phục các tệp cấu hình trên thiết bị.  cũng cần làm quen với các tệp nhật ký (logs) có sẵn và cách tìm kiếm qua logs để xác định các sự kiện cụ thể. Vì tường lửa chưa được đưa vào triển khai chính thức ngay, có thể dành thời gian thực hiện các tác vụ này mà không lo ảnh hưởng đến mạng sản xuất (production network).
![[Pasted image 20260627221431.png]]

## 2. Mục tiêu bài Lab (Lab Objectives)
* Xuất (export) tệp cấu hình được đặt tên ra thiết bị lưu trữ bên ngoài.
* Lưu các thay đổi cấu hình đang diễn ra trước khi thực hiện commit.
* Hoàn tác (revert) các thay đổi cấu hình đang diễn ra.
* Xem trước (preview) các thay đổi cấu hình so với cấu hình đang chạy.
* Kiểm tra các tệp nhật ký hệ thống (System log) và cấu hình (Configuration log).
* Tạo bộ lọc cho tệp log (log file filter).
* Sử dụng trình xây dựng bộ lọc (Filter Builder).

---

## 3. Các bước thực hiện mức cao (High-Level Steps)

### Bước 1: Lưu Snapshot cấu hình có tên (Named Configuration Snapshot)
* Lưu cấu hình hiện tại của tường lửa với tên tệp: `firewal-a-<Today's Date>.xml`.
### Bước 2: Xuất Snapshot cấu hình ra ngoài (Export)
* Xuất tệp cấu hình `firewall-a-<Today’s Date>.xml` vừa lưu về thư mục **Downloads** của máy trạm.
### Bước 3: Hoàn tác các thay đổi cấu hình đang diễn ra (Revert Changes)
* Thay đổi giá trị của **Primary DNS Server** thành `88.8.8.8` (giả lập một lỗi nhập liệu phổ biến).
* Xác nhận lỗi trong phần **Services**.
* Sử dụng tính năng **Revert Changes** để khôi phục Primary DNS Server về giá trị gốc (`8.8.8.8`).
### Bước 4: Xem trước các thay đổi cấu hình (Preview Changes)
* Sửa đổi cấu hình SNMP với các thông số:
  * **Physical Location**: `Santa Clara, CA, USA`
  - **Contact**: `Sherlock Holmes`
  - **SNMP Community String**: `paloalto42`
* Sử dụng tính năng **Preview Changes** để so sánh sự khác biệt giữa cấu hình đang chạy (Running Configuration) và cấu hình đang chờ (Candidate Configuration).
*Lưu ý: Không thực hiện commit ở giai đoạn này.*
### Bước 5: Tạo bộ lọc file nhật ký hệ thống (System Log Filter)
* Ẩn cột **Object** trong giao diện hiển thị log hệ thống.
* Tạo và áp dụng một bộ lọc trong mục **System Log** để chỉ hiển thị các bản ghi có mức độ nghiêm trọng (**Severity**) là `informational`.
### Bước 6: Sử dụng trình dựng bộ lọc (Filter Builder)
* Sử dụng **Filter Builder** để tạo một bộ lọc hiển thị tất cả các log hệ thống phát sinh trong vòng 60 phút qua (hoặc sử dụng toán tử logic kết hợp với điều kiện thời gian).

---

## 4. Các bước cấu hình chi tiết (Detailed Steps)

### Phần 1: Lưu Snapshot cấu hình có tên riêng (Save Named Snapshot)
10. Tiếp tục truy cập mục **Device > Setup > Operations**.
11. Click vào liên kết **Save named configuration snapshot**.
    ![[Pasted image 20260629211333.png]]
12. Trong hộp thoại mở ra, nhập tên tệp: `firewall-a-Jun-6-2026.xml` (hoặc thay bằng ngày hiện tại của ).
	![[Pasted image 20260627232540.png]]
	
13. Click **OK** để lưu lại snapshot candidate này.
	***nó sẽ lưu file cấu hình trên chính firewall đấy luôn***

### Phần 2: Xuất Snapshot cấu hình ra ngoài (Export Snapshot)
14. Trong cùng giao diện **Device > Setup > Operations**, di chuyển xuống phần **Export**.
15. Click vào liên kết **Export named configuration snapshot**.
    ![[Pasted image 20260627232917.png]]
16. 
17. Trong cửa sổ mở ra, bấm menu thả xuống và tìm đúng tệp cấu hình  vừa lưu: `firewall-a-<Today’s Date>.xml`.
    ![[Pasted image 20260627232830.png]]
18. Click **OK**.
19. Trình duyệt/Hệ điều hành sẽ nhắc  tải tệp xuống, hãy lưu tệp vào thư mục **Downloads** trên máy trạm.
20. Mở thư mục **Downloads** trên máy tính để kiểm tra: tệp tin có dung lượng khoảng `12.2 KB` sẽ xuất hiện tại đây.
    ![[Pasted image 20260627233038.png]]
21. Đóng cửa sổ thư mục Downloads sau khi xác nhận thành công.

### Phần 3: Hoàn tác các thay đổi đang thực hiện (Revert Ongoing Changes)
*Trong quá trình quản trị, việc cấu hình sai hoặc nhầm lẫn là hoàn toàn có thể xảy ra. Nếu  thực hiện nhiều thay đổi cùng lúc và không nhớ hết mình đã sửa những gì,  có thể khôi phục nhanh candidate configuration về trạng thái của running configuration hiện tại.*

21. Điều hướng tới mục **Device > Setup > Services**.
22. Nhấp vào biểu tượng **bánh răng (Gear)** ở góc phải của bảng **Services** để mở cửa sổ chỉnh sửa.
23. Tại trường **Primary DNS Server**, hãy đổi giá trị thành `88.8.8.8` (đây là giá trị sai nhằm mục đích thử nghiệm). Nhấp **OK**.
    ![[Pasted image 20260628103005.png]]
24. In the upper right corner of the web interface, click the Changes button and select Revert Changes:![[Pasted image 20260628104000.png]]
25.   In the Revert Changes window, leave the settings unchanged:.
    ![[Pasted image 20260628104057.png]]
26. Click Close in the Message window: ![[Pasted image 20260628104224.png]]
27. Quay lại mục **Device > Setup > Services** và xác minh rằng trường **Primary DNS Server** đã tự động khôi phục về địa chỉ gốc là `8.8.8.8`.![[Pasted image 20260628104154.png]]

### Phần 4: Xem trước thay đổi cấu hình (Preview Configuration Changes)
27. Truy cập **Device > Setup > Operations**.
28. Di chuyển sang bảng bên phải (hoặc khu vực cấu hình phụ thuộc) và tìm mục cấu hình **SNMP** (hoặc chọn mục **Miscellaneous > SNMP Setup** nếu có). Click vào biểu tượng bánh răng để chỉnh sửa.
    ![[Pasted image 20260629211854.png]]
29. Cấu hình các thông số sau:
    * **Physical Location**: `Santa Clara, CA, USA`
    * **Contact**: `Sherlock Holmes`
    * **SNMP Community String**: `paloalto42`
      ![[Pasted image 20260629211736.png]]
30. Nhấp **OK** để đóng cửa sổ thiết lập SNMP.
31. Bây giờ, nhấp vào nút **Commit** ở góc trên bên phải màn hình.
32. Trong cửa sổ cấu hình Commit, thay vì nhấn nút Commit ngay, hãy nhấp vào liên kết **Preview Changes** nằm ở phía dưới bên trái của hộp thoại.
    ![[Pasted image 20260629212027.png]]
33. Chọn số dòng muốn xem xung quanh đoạn thay đổi (ví dụ: dòng mặc định là 10 dòng) rồi nhấp **OK**.
    ![[Pasted image 20260629212054.png]]
34. Một cửa sổ so sánh mã cấu hình (XML) sẽ hiển thị:
    * Các dòng màu **xanh lá cây** (có dấu `+`) biểu thị các thành phần mới được thêm vào candidate configuration (Location, Contact, Community string).
    * Nếu có dòng màu **đỏ** (có dấu `-`), đó là các thành phần đã bị xóa.
      ![[Pasted image 20260629212156.png]]
35. Sau khi xem xét và hiểu rõ các thay đổi, nhấp vào dấu **X** ở góc trên bên phải để đóng cửa sổ so sánh.
36. Nhấp **Cancel** trên cửa sổ Commit để hủy bỏ lệnh commit (chúng ta không lưu các thay đổi SNMP này vào cấu hình chạy chính thức).

### Phần 5: Tạo bộ lọc bộ dữ liệu nhật ký hệ thống (System Log Filter)
37. Điều hướng tới tab điều khiển giám sát: **Monitor > Logs > System**.
    ![[Pasted image 20260629212301.png]]
38. Để tùy chỉnh các cột hiển thị nhằm tối ưu không gian làm việc:
    * Nhấp chuột vào mũi tên nhỏ trên bất kỳ tiêu đề cột nào trong bảng log.
    * Di chuột tới mục **Columns**.
    * Bỏ tích chọn cột **Object** để ẩn cột này đi.
      ![[Pasted image 20260629212344.png]]
    *  có thể kéo thả cột **Action** để di chuyển vị trí của nó ra giữa cột **Name** và cột **Tag** (thao tác này là tùy chọn).
39. Nhấp vào nút **Bộ lọc (Filter Icon)** hoặc nhấp trực tiếp vào chữ **informational** trong cột **Severity** của một dòng log bất kỳ. Hệ thống sẽ tự động điền cú pháp lọc lên thanh tìm kiếm ở phía trên.
    ![[Pasted image 20260629212442.png]]
40. Hãy quan sát thanh tìm kiếm ở đầu trang, cú pháp lọc chính xác sẽ hiển thị là: `(severity eq informational)`.
    ![[Pasted image 20260629212516.png]]
41. Nhấn nút **Apply** (mũi tên xanh) để áp dụng bộ lọc. Xác mảng rằng danh sách log bây giờ chỉ hiển thị các sự kiện có độ nghiêm trọng là *informational*.
    ![[Pasted image 20260629212529.png]]

### Phần 6: Sử dụng Filter Builder để lọc theo thời gian
42. Tại giao diện **Monitor > Logs > System**, nhấp vào biểu tượng **Filter Builder** (hình phễu hoặc biểu tượng nâng cao bên cạnh thanh tìm kiếm).
43. Trình dựng bộ lọc **Add Log Filter** sẽ mở ra, cấu hình phần đầu tiên của bộ lọc:
    * **Attribute**: Chọn `Severity`
    * **Operator**: Chọn `equal`
    * **Value**: Chọn `informational`
    * Nhấp **Add**.
      ![[Pasted image 20260629212621.png]]
44. Không đóng cửa sổ này, tiếp tục xây dựng phần thứ hai của bộ lọc để kết hợp điều kiện:
    * **Connector**: Chọn `and`
    * **Attribute**: Chọn `Time Generated`
    * **Operator**: Chọn `greater than or equal to` (lớn hơn hoặc bằng)
    * **Value**: Sử dụng danh sách thả xuống đầu tiên chọn `today`.
      ![[Pasted image 20260629212716.png]]
45. Quan sát ô hiển thị cú pháp ở trên cùng tự động cập nhật chính xác theo logic  vừa xây dựng.
46. Nhấp **Apply** để thực thi bộ lọc nâng cao trên bảng logs hệ thống, sau đó nhấp **Close** để đóng trình dựng.

