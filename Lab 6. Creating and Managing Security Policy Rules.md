# Lab 6: Tạo và Quản lý Quy tắc Chính sách Bảo mật

 Đã triển khai tường lửa và kết nối với tất cả các mạng phù hợp. Bước tiếp theo là bắt đầu tạo các quy tắc Chính sách Bảo mật.  sẽ bắt đầu bằng cách tạo các quy tắc cho phép máy chủ trong vùng `Users_Net` giao tiếp với máy chủ trong vùng `Extranet`. Sau đó,  sẽ tạo các quy tắc Chính sách Bảo mật để cho phép máy chủ trong vùng `Users_Net` kết nối với máy chủ trong vùng `Internet`.
cũng cần cho phép máy chủ trong vùng `Extranet` giao tiếp với máy chủ trong vùng `Internet`
![[Pasted image 20260630092628.png]]

---
## Mục tiêu của Lab

- Cấu hình quy tắc Chính sách Bảo mật cho phép truy cập từ `Users_Net` đến `Extranet`
- Kiểm tra truy cập từ máy khách đến máy chủ Extranet
- Xem Nhật ký Lưu lượng (Traffic log)
- Kiểm tra Số lần truy cập quy tắc (Rule Hit Count)
- Đặt lại bộ đếm số lần truy cập quy tắc
- Tùy chỉnh bảng Chính sách
- Bật ghi nhật ký cho quy tắc nội vùng và liên vùng
- Tạo các quy tắc Chính sách Bảo mật đến vùng Internet

---
## Các bước Tổng quan Cấp cao

### Tạo Quy tắc Chính sách Bảo mật

Sử dụng thông tin bên dưới để tạo quy tắc Chính sách Bảo mật cho phép lưu lượng từ vùng `Users_Net` đến vùng `Extranet`.

| Tham số | Giá trị |
|---------|---------|
| Tên quy tắc | Users_to_Extranet |
| Mô tả | Cho phép máy chủ trong vùng Users_Net truy cập máy chủ trong vùng Extranet |
| Vùng nguồn | Users_Net |
| Vùng đích | Extranet |
| Ứng dụng | Any |
| Dịch vụ | application-default |
| Danh mục URL | Any |
| Hành động | Allow |
### Commit cấu hình
Commit các thay đổi trước khi tiếp tục.
### Sửa đổi các cột trong bảng Chính sách Bảo mật
Ẩn các cột sau trong bảng Chính sách Bảo mật để tạo thêm không gian xem thông tin hữu ích:
- Type (Loại)
- Source Device (Thiết bị nguồn)
- Destination Device (Thiết bị đích)
- Options (Tùy chọn)
Kéo và thả cột **Action (Hành động)** từ vị trí hiện tại để nó nằm giữa cột **Name (Tên)** và cột **Tag (Thẻ)**.
### Kiểm tra Quy tắc Chính sách Bảo mật mới

Từ máy chủ Client-A, ping đến `192.168.50.80`, là địa chỉ IP của máy chủ web trong vùng Extranet.
Sử dụng trình duyệt web trên máy khách Client-A để kết nối đến trang web Extranet tại `192.168.50.80`.

### Kiểm tra Số lần truy cập Quy tắc
- Trong bảng quy tắc Chính sách Bảo mật, xác định cột **Hit Count (Số lần truy cập)** và ghi lại số lần truy cập của quy tắc `Users_to_Extranet`.
- Từ máy chủ Client-A, ping đến máy chủ web Extranet - `192.168.50.80`.
- Làm mới Hit Count và ghi nhận sự tăng lên của giá trị cho quy tắc `Users_to_Extranet`.
### Đặt lại Bộ đếm Số lần truy cập Quy tắc

Đặt lại Hit Count cho quy tắc `Users_to_Extranet`.
### Kiểm tra Nhật ký Lưu lượng
Ẩn các cột sau trong Nhật ký Lưu lượng:
- Type (Loại)
- Source Dynamic Address Group (Nhóm địa chỉ động nguồn)
- Destination Dynamic Address Group (Nhóm địa chỉ động đích)
- Dynamic User Group (Nhóm người dùng động)
Từ cửa sổ terminal trên máy chủ Client-A, ping `8.8.8.8`.
 sẽ không nhận được phản hồi.
Kiểm tra lại nhật ký lưu lượng và sử dụng một bộ lọc đơn giản để xem có bất kỳ mục nhập nào cho phiên ping thất bại không.
**Câu hỏi:** Tại sao không có mục nhập nào trong Nhật ký Lưu lượng cho phiên ping đến `8.8.8.8` của ?
Viết câu trả lời vào ô được cung cấp hoặc trên giấy ghi chú trong lớp.
### Bật Ghi nhật ký cho Quy tắc Liên vùng Mặc định
Sửa quy tắc Chính sách Bảo mật `interzone-default` và bật **Log at Session End (Ghi nhật ký khi kết thúc phiên)**.
### Commit cấu hình
Commit các thay đổi trước khi tiếp tục.
### Ping đến một Máy chủ trên Internet
Từ cửa sổ terminal trên máy chủ Client-A, ping `8.8.8.8`.
 sẽ không nhận được phản hồi.
Kiểm tra lại Nhật ký Lưu lượng và sử dụng bộ lọc đơn giản để xem có bất kỳ mục nhập nào cho phiên này thất bại không.
Các mục nhập trong Nhật ký Lưu lượng sẽ cho thấy các phiên ping đang khớp với quy tắc `interzone-default`.

### Tạo Quy tắc Chặn cho các Địa chỉ IP Đã biết Xấu
Sử dụng thông tin bên dưới để tạo quy tắc ở đầu Chính sách Bảo mật nhằm chặn lưu lượng đến các địa chỉ IP xấu đã biết do Palo Alto Networks cung cấp.

| Tham số | Giá trị |
|---------|---------|
| Tên quy tắc | Block-to-Known-Bad-Addresses |
| Mô tả | Chặn lưu lượng từ Users và Extranet đến các địa chỉ IP xấu đã biết |
| Vùng nguồn | Users_Net <br> Extranet |
| Vùng đích | Internet |
| Địa chỉ đích | • Palo Alto Networks - Bulletproof IP addresses <br> • Palo Alto Networks - High risk IP addresses <br> • Palo Alto Networks - Known malicious IP addresses |
| Ứng dụng | Any |
| Dịch vụ | any |
| Danh mục URL | Any |
| Hành động | Deny |

Sử dụng thông tin bên dưới để tạo một quy tắc Chính sách Bảo mật khác chặn lưu lượng từ các địa chỉ IP xấu đã biết. Đặt quy tắc này ở đầu Chính sách Bảo mật, ngay bên dưới quy tắc `Block-to-Known-Bad-Addresses`.

| Tham số | Giá trị |
|---------|---------|
| Tên quy tắc | Block-from-Known-Bad-Addresses |
| Mô tả | Chặn lưu lượng từ các địa chỉ IP xấu đã biết đến Users và Extranet |
| Vùng nguồn | Internet |
| Địa chỉ nguồn | • Palo Alto Networks - Bulletproof IP addresses <br> • Palo Alto Networks - High risk IP addresses <br> • Palo Alto Networks - Known malicious IP addresses |
| Vùng đích | Users_Net <br> Extranet |
| Ứng dụng | Any |
| Dịch vụ | application-default |
| Danh mục URL | Any |
| Hành động | Deny |

### Tạo Quy tắc Bảo mật cho Truy cập Internet

#### Tạo Quy tắc Chính sách Bảo mật Users to Internet

Sử dụng thông tin bên dưới để tạo quy tắc Chính sách Bảo mật cho phép lưu lượng từ vùng `Users_Net` đến vùng `Internet`.

| Tham số | Giá trị |
|---------|---------|
| Tên quy tắc | Users_to_Internet |
| Mô tả | Cho phép máy chủ trong vùng Users_Net truy cập vùng Internet |
| Vùng nguồn | Users_Net |
| Vùng đích | Internet |
| Ứng dụng | Any |
| Dịch vụ | application-default |
| Danh mục URL | Any |
| Hành động | Allow |
#### Tạo Quy tắc Chính sách Bảo mật Extranet to Internet
Sử dụng thông tin bên dưới để tạo quy tắc Chính sách Bảo mật cho phép lưu lượng từ vùng `Extranet` đến vùng `Internet`.

| Tham số | Giá trị |
|---------|---------|
| Tên quy tắc | Extranet_to_Internet |
| Mô tả | Cho phép máy chủ trong vùng Extranet truy cập vùng Internet |
| Vùng nguồn | Extranet |
| Vùng đích | Internet |
| Ứng dụng | Any |
| Dịch vụ | application-default |
| Danh mục URL | Any |
| Hành động | Allow |

### Commit cấu hình
Commit các thay đổi trước khi tiếp tục.

### Ping Máy chủ Internet từ Client A
Từ cửa sổ terminal trên máy chủ Client-A, ping `8.8.8.8`.
 sẽ không nhận được phản hồi.

Kiểm tra lại Nhật ký Lưu lượng và sử dụng bộ lọc đơn giản để xem có bất kỳ mục nhập nào cho phiên này thất bại không.
Các mục nhập trong Nhật ký Lưu lượng sẽ cho thấy các phiên ping đang khớp với quy tắc `Users_to_Internet`.
**Câu hỏi:**  có thể giải thích tại sao phiên ping từ máy khách đến máy chủ Internet không nhận được phản hồi mặc dù tường lửa đang cho phép lưu lượng không?
Viết câu trả lời vào ô được cung cấp hoặc trên giấy ghi chú trong lớp.

---

## Các bước Chi tiết
### Create a Security Policy Rule

 cần cho phép lưu lượng mạng từ vùng bảo mật Users_Net đến vùng bảo mật Extranet để nhân viên có thể truy cập các ứng dụng kinh doanh khác nhau. Trong phần này,  sẽ tạo quy tắc Chính sách Bảo mật để cho phép truy cập giữa hai vùng này.![[Pasted image 20260629234750.png]]

10. Chọn **Policies > Security**.
11. Nhấp **Add** ở cuối cửa sổ.
12. Trong tab **General**, tại trường **Name**, nhập `Users_to_Extranet`.
13. Tại **Description**, nhập `Allows hosts in Users_Net zone to access servers in Extranet zone`.
14. Giữ nguyên các thiết lập khác.![[Pasted image 20260629235444.png]]
15. Chọn tab **Source**.
16. Trong phần **Source Zone**, nhấp **Add**.
17. Chọn **Users_Net**.
18. Giữ nguyên các thiết lập còn lại.![[Pasted image 20260629234953.png]]
19. Chọn tab **Destination**.
20. Trong phần **Destination Zone**, nhấp **Add**.
21. Chọn **Extranet**.
22. Giữ nguyên các thiết lập khác.![[Pasted image 20260629235009.png]]
23. Chọn tab **Application**.
24. Không thay đổi bất kỳ thiết lập nào ở đây, nhưng lưu ý rằng hộp **Any** được chọn.![[Pasted image 20260629235608.png]]
   > Sau này trong khóa học, chúng ta sẽ tìm hiểu về Ứng dụng và cách sử dụng chúng trong các quy tắc Chính sách Bảo mật.

25. Chọn tab **Service/URL Category**.
26. Không thay đổi bất kỳ thiết lập nào trong tab này, nhưng lưu ý rằng **Service** được đặt là `application-default`.![[Pasted image 20260629235628.png]]

   > Cài đặt `application-default` hướng dẫn tường lửa cho phép một ứng dụng như `web-browsing` miễn là ứng dụng đó đang sử dụng dịch vụ (hoặc cổng đích) được xác định trước. Đối với ứng dụng như `web-browsing`, dịch vụ mặc định là TCP 80; đối với ứng dụng như `ssl`, dịch vụ mặc định là TCP 443. Chúng ta sẽ dành nhiều thời gian sau trong khóa học để thảo luận về Ứng dụng và cài đặt `application-default`.

27. Chọn tab **Actions**.
28.  không cần thay đổi gì trong phần này, nhưng lưu ý rằng **Action** được đặt mặc định là **Allow**.![[Pasted image 20260629235700.png]]

   > Khi  tạo một quy tắc Chính sách Bảo mật mới, Hành động được tự động đặt là Allow. Nếu  đang tạo quy tắc để chặn lưu lượng, hãy đảm bảo chọn tab Actions và thay đổi Action trước khi commit quy tắc.

29. Nhấp **OK** trên cửa sổ Security Policy Rule.
30. Quy tắc Chính sách Bảo mật mới xuất hiện trong bảng.![[Pasted image 20260630091820.png]]

### Commit cấu hình

31. Nhấp nút **Commit** ở góc trên bên phải của giao diện web.
32. Giữ nguyên các thiết lập và nhấp **Commit**.
33. Chờ cho đến khi quá trình Commit hoàn tất.
34. Nhấp **Close** để tiếp tục.

### Sửa đổi các cột trong bảng Chính sách Bảo mật

 có thể tùy chỉnh thông tin được hiển thị trong bảng Chính sách Bảo mật để phù hợp với nhu cầu của mình. Trong phần này,  sẽ ẩn một số cột và hiển thị những cột khác mà  có thể quan tâm hơn.  cũng sẽ di chuyển các cột và sử dụng tính năng Adjust Column.

35. Nhấp vào biểu tượng thả xuống nhỏ bên cạnh cột **Name** trong bảng Chính sách Bảo mật.![[Pasted image 20260629235820.png|497]]

   > Biểu tượng này có sẵn bên cạnh tất cả các tiêu đề cột.

36. Chọn **Columns** và lưu ý các cột có sẵn mà  có thể ẩn hoặc hiển thị trong bảng này.
37. Bỏ chọn các mục sau:
   - Type
   - Source Device
   - Destination Device
   - Options![[Pasted image 20260629235945.png|261]]

36. Kéo và thả cột **Action** từ vị trí hiện tại để nó nằm giữa cột **Name** và cột **Tag**.![[Pasted image 20260630000352.png]]

   > Lưu ý: Những thay đổi này là tùy chọn.  không bắt buộc phải hiển thị hoặc ẩn các cột hoặc sắp xếp lại các mục trong bất kỳ bảng nào của tường lửa. Tuy nhiên,  có thể thấy rằng có một số cột nhất định trong một số bảng mà  không bao giờ sử dụng và  có thể ẩn chúng để tạo thêm không gian trong bảng.  cũng có thể thấy rằng có một số cột mà  thường xuyên quét và có thể di chuyển chúng đến vị trí dễ nhìn hơn.

37. Ở đầu cột **Name**, nhấp vào biểu tượng thả xuống lần nữa và chọn **Adjust Columns**.![[Pasted image 20260630000538.png]]
38. Thao tác này sẽ thay đổi kích thước các cột được hiển thị để phù hợp nhất với cửa sổ trình duyệt.

### Kiểm tra Quy tắc Chính sách Bảo mật mới

41. Để đảm bảo quy tắc Chính sách Bảo mật của  hoạt động, hãy mở cửa sổ terminal trên máy chủ.
42. Sử dụng lệnh sau để ping đến `192.168.50.80`, là địa chỉ IP của máy chủ web trong vùng Extranet.

    ```
    lab-user@client-a:~/Desktop/Lab-Files$ ping 192.168.50.80
    ```

43. Sau vài phản hồi, nhấn **Ctrl+C** để dừng ping.

    > Nếu  thấy phản hồi từ 192.168.50.80, thì quy tắc Chính sách Bảo mật của  đã được cấu hình chính xác! Nếu không, hãy xem lại các bước trước và thử lại lần kiểm tra này.

44. Trên máy trạm, mở trình duyệt **Firefox** để kiểm tra.
45. Sử dụng thanh bookmark và chọn **Extranet > Extranet**.
46.  sẽ thấy một trang web được hiển thị bởi máy chủ.![[Pasted image 20260630091900.png]]
47. Đóng trình duyệt kiểm tra.

### Kiểm tra Số lần truy cập Quy tắc

Sau khi quy tắc được thiết lập thành công,  có thể kiểm tra bộ đếm số lần truy cập trong bảng quy tắc Chính sách Bảo mật. Những bộ đếm này rất hữu ích cho việc xử lý sự cố. Nếu một quy tắc không được truy cập,  có thể cần sửa đổi nó.

48. Trên giao diện web của tường lửa, chọn **Policies > Security**.
49. Cuộn sang phải và xác định cột **Hit Count**.![[Pasted image 20260630000834.png]]

50. Ghi lại số lần truy cập trên quy tắc này.
51. Quay lại cửa sổ terminal trên máy trạm của .
52. Ping lại máy chủ bằng lệnh:

    ```
    lab-user@client-a:~/Desktop/Lab-Files$ ping 192.168.50.80
    ```

53. Sau vài phản hồi, nhấn **Ctrl+C** để dừng ping.

    ```
    lab-user@client-a:~/Desktop/Lab-Files$ ping 192.168.50.80
    PING 192.168.50.80 (192.168.50.80) 56(84) bytes of data.
    64 bytes from 192.168.50.80: icmp_seq=1 ttl=63 time=0.641 ms
    64 bytes from 192.168.50.80: icmp_seq=2 ttl=63 time=0.615 ms
    64 bytes from 192.168.50.80: icmp_seq=3 ttl=63 time=0.731 ms
    64 bytes from 192.168.50.80: icmp_seq=4 ttl=63 time=0.732 ms
    ^C
    --- 192.168.50.80 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3059ms
    rtt min/avg/max/mdev = 0.615/0.679/0.732/0.052 ms
    lab-user@client-a:~/Desktop/Lab-Files$
    ```

54. Quay lại giao diện web của tường lửa và làm mới bảng quy tắc Chính sách Bảo mật bằng cách nhấp nút **Refresh** ở góc trên bên phải của cửa sổ.
55. Ghi nhận sự tăng lên của Hit Count cho quy tắc của .

### Đặt lại Bộ đếm Số lần truy cập Quy tắc

Số lần truy cập quy tắc rất hữu ích để theo dõi xem quy tắc có được cấu hình chính xác hay không.  có thể đặt lại bộ đếm cho tất cả quy tắc Chính sách Bảo mật hoặc cho một quy tắc duy nhất. Trong phần này,  sẽ đặt lại bộ đếm cho quy tắc `Users_to_Extranet`.

56. Chọn **Policies > Security**.
57. Đánh dấu mục `Users_to_Extranet` nhưng không mở nó.
58. Ở cuối cửa sổ, chọn **Reset Rule Hit Counter > Selected rules**.
    ![[Pasted image 20260630001005.png]]

    > Thao tác này không yêu cầu commit.

59. Hit Count được đặt về 0.

### Kiểm tra Nhật ký Lưu lượng

Nhật ký Lưu lượng chứa thông tin về các phiên mà tường lửa cho phép hoặc chặn. Trong phần này,  sẽ kiểm tra Nhật ký Lưu lượng để xác định các mục cho phiên giữa vùng Users_Net và vùng Extranet.

60. Chọn **Monitor > Logs > Traffic**.![[Pasted image 20260630001118.png]]
61. Nhấp vào biểu tượng thả xuống bên cạnh **Receive time** và chọn **Columns**.
62. Bỏ chọn các mục sau để ẩn các cột:
   - Type
   - Source Dynamic Address Group
   - Destination Dynamic Address Group![[Pasted image 20260630001201.png|412]]

   > Đây không phải là yêu cầu bắt buộc, nhưng chúng ta sẽ không sử dụng thông tin từ các cột này trong bất kỳ lab nào của khóa học.

63. Từ cửa sổ terminal trên máy trạm, ping một địa chỉ trên internet bằng lệnh:

    ```
    lab-user@client-a:~/Desktop/Lab-Files$ ping 8.8.8.8
    ```

64.  sẽ không nhận được phản hồi, vì vậy sau vài giây, nhấn **Ctrl+C** để dừng ping.
65. Kiểm tra lại nhật ký lưu lượng và sử dụng một bộ lọc đơn giản để xem có mục nhập nào cho phiên này thất bại không.
66. Chọn **Monitor > Logs > Traffic**.
67. Trong trường bộ lọc, nhập chính xác văn bản sau:![[Pasted image 20260630101123.png]]

    ```
    (addr.dst eq 8.8.8.8)
    ```

    > Bộ lọc phân biệt chữ hoa chữ thường, vì vậy hãy nhập chính xác! Ngoài ra, lưu ý rằng có một dấu cách sau dấu ngoặc đơn đầu tiên và trước dấu ngoặc đơn cuối cùng.

68. Nhấp nút **Apply filter** ở góc trên bên phải của cửa sổ (hoặc  có thể nhấn phím **Enter**).![[Pasted image 20260630101200.png]]
69. Nhật ký Lưu lượng sẽ cập nhật hiển thị nhưng không có mục nào khớp.

70. **Trả lời câu hỏi sau:**

    **Tại sao không có mục nhập nào trong Nhật ký Lưu lượng cho phiên ping đến `8.8.8.8` của ?**

    Viết câu trả lời vào ô được cung cấp hoặc trên giấy ghi chú trong lớp.

### Bật Ghi nhật ký cho Quy tắc Liên vùng Mặc định

Nếu  không thể giải thích tại sao tường lửa không ghi nhật ký phiên ping đến địa chỉ bên ngoài,  không đơn độc. Hầu hết học viên trong lớp có lẽ cũng không tìm ra.

Có hai lý do:

Thứ nhất,  chưa có quy tắc Chính sách Bảo mật cho phép lưu lượng từ vùng Users_Net đến vùng Internet. Khi tường lửa kiểm tra phiên ping, quy tắc duy nhất khớp là `interzone-default`, quy tắc này từ chối mọi lưu lượng từ vùng này sang vùng khác. Phiên ping khớp với quy tắc này; tuy nhiên, không có mục nhập nào trong Nhật ký Lưu lượng cho biết sự khớp này.

Thứ hai, hãy nhớ rằng lưu lượng khớp với quy tắc `interzone-default` không được tự động ghi nhật ký.  phải thay đổi thủ công một cài đặt trên quy tắc này để thấy các mục trong Nhật ký Lưu lượng.  sẽ bật cài đặt này ngay bây giờ và thực hiện lại bài kiểm tra.

71. Chọn **Policies > Security**.
72. Đánh dấu mục `interzone-default` trong danh sách Chính sách nhưng không mở nó.![[Pasted image 20260630101220.png]]
73. Nhấp nút **Override** ở cuối cửa sổ.
74. Chọn tab **Actions**.
75. Đánh dấu chọn vào ô **Log at session end**.
76. Giữ nguyên các thiết lập còn lại.![[Pasted image 20260630101245.png]]
77. Nhấp **OK**.

### Commit cấu hình

78. Nhấp nút **Commit** ở góc trên bên phải của giao diện web.
79. Giữ nguyên các thiết lập và nhấp **Commit**.
80. Chờ cho đến khi quá trình Commit hoàn tất.
81. Nhấp **Close** để tiếp tục.

### Ping đến một Máy chủ trên Internet

82. Bây giờ  đã bật **Log at session end** cho các quy tắc Chính sách Bảo mật mặc định, hãy ping một máy chủ trên internet và kiểm tra Nhật ký Lưu lượng để xem kết quả.
83. Từ cửa sổ Terminal trên máy trạm, ping một địa chỉ trên Internet bằng lệnh:

    ```
    lab-user@client-a:~/Desktop/Lab-Files$ ping 8.8.8.8
    ```

84.  sẽ không nhận được phản hồi, vì vậy sau vài giây, nhấn **Ctrl+C** để dừng ping.
85. Kiểm tra lại nhật ký lưu lượng và sử dụng bộ lọc đơn giản để xem có mục nhập nào cho phiên này thất bại không.
86. Chọn **Monitor > Logs > Traffic**.
87. Trong trường bộ lọc, nhập chính xác văn bản sau:![[Pasted image 20260630101307.png]]

    ```
    (addr.dst eq 8.8.8.8)
    ```

    > Bộ lọc của  có thể đã được đặt từ trước.

88. Nhấp nút **Apply Filter** ở góc trên bên phải của cửa sổ (hoặc nhấn **Enter**).
89. Nhật ký Lưu lượng sẽ cập nhật và  sẽ thấy các mục khớp với bộ lọc.
90.  có thể thấy các phiên đang khớp với quy tắc `interzone-default`.
91. Nhấp vào biểu tượng **X** để xóa bộ lọc khỏi hộp văn bản bộ lọc.

### Tạo Quy tắc Chặn cho các Địa chỉ IP Đã biết Xấu

Palo Alto Networks cung cấp một số danh sách địa chỉ IP được biết là độc hại. Thực hành tốt là  nên tạo các quy tắc Chính sách Bảo mật để chặn lưu lượng đến và từ các địa chỉ đã biết này.

92. Trong **Policies > Security**, nhấp **Add** ở cuối cửa sổ.
93. Tại **Name**, nhập `Block-to-Known-Bad-Addresses`.
94. Tại **Description**, nhập `Blocks traffic from users and Extranet to known bad IP addresses`.
95. Chọn tab **Source**.
96. Trong phần **Source Zone**, nhấp **Add**.
97. Chọn vùng **Users_Net**.
98. Trong phần **Source Zone**, nhấp **Add** lần nữa.
99. Chọn vùng **Extranet**.

    > Lưu ý rằng  đang thêm cả hai vùng nội bộ vào phần Source Zone của quy tắc.

100. Chọn tab **Destination**.
101. Trong phần **Destination Zone**, nhấp **Add**.
102. Chọn vùng **Internet**.
103. Trong phần **Destination Address** của tab Destination, nhấp **Add**.
104. Chọn **Palo Alto Networks - Bulletproof IP addresses**.
105. Nhấp **Add** lần nữa trong phần **Destination Address**.
106. Chọn **Palo Alto Networks - High risk IP addresses**.
107. Nhấp **Add** lần nữa trong phần **Destination Address**.
108. Chọn **Palo Alto Networks - Known malicious IP addresses**.

    Khi hoàn tất,  sẽ có ba danh sách địa chỉ IP Palo Alto Networks trong phần Destination Address của quy tắc.

109. Chọn tab **Application**.
110. Để Application là `any`.
111. Trong tab **Service/URL Category**, thay đổi **Service** từ `application-default` thành `any`.![[Pasted image 20260630101417.png]]

Khi tạo quy tắc từ chối, Palo Alto Networks khuyến nghị đặt Service là `any` thay vì sử dụng `application-default`.

112. Chọn tab **Actions**.
113. Thay đổi **Action** thành **Deny**.![[Pasted image 20260630101451.png]]
114. Nhấp **OK**. Quy tắc mới xuất hiện trong bảng Chính sách Bảo mật.
115. Di chuyển quy tắc này lên đầu Chính sách Bảo mật, bằng cách đánh dấu mục `Block-to-Known-Bad-Addresses` (không mở nó).
116. Ở cuối cửa sổ, chọn **Move** và chọn **Move Top**.![[Pasted image 20260630101514.png]]
![[Pasted image 20260630101540.png]]
117. Tạo một quy tắc khác để chặn lưu lượng từ các địa chỉ IP xấu đã biết.
118. Trong cửa sổ Chính sách Bảo mật, nhấp **Add**.
119. Tại **Name**, nhập `Block-from-Known-Bad-Addresses`.
120. Tại **Description**, nhập `Blocks traffic from known bad IP addresses to Users and Extranet`.
121. Chọn tab **Source**.
122. Trong phần **Source Zone**, nhấp **Add**.
123. Chọn vùng **Internet**.
124. Trong phần **Source Address**, nhấp **Add**.
125. Chọn **Palo Alto Networks - Bulletproof IP addresses**.
126. Nhấp **Add** lần nữa trong phần **Source Address**.
127. Chọn **Palo Alto Networks - High risk IP addresses**.
128. Nhấp **Add** lần nữa trong phần **Source Address**.
129. Chọn **Palo Alto Networks - Known malicious IP addresses**.

    Khi hoàn tất,  sẽ có ba danh sách địa chỉ IP Palo Alto Networks trong phần Source Address của quy tắc.

130. Chọn tab **Destination**.
131. Trong phần **Destination Zone**, nhấp **Add**.
132. Chọn vùng **Users_Net**.
133. Nhấp **Add** lần nữa trong **Destination Zone**.
134. Chọn **Extranet**.

    > Lưu ý rằng  đang thêm cả hai vùng nội bộ vào phần Destination Zone của quy tắc.

135. Chọn tab **Application**.
136. Để Application là `any`.
137. Trong tab **Service/URL Category**, đặt **Service** thành `any`.
138. Chọn tab **Actions**.
139. Thay đổi **Action** thành **Deny**.
140. Nhấp **OK**.
141. Quy tắc mới xuất hiện trong bảng Chính sách Bảo mật.
142. Di chuyển quy tắc `Block-to-Known-Bad-Addresses` lên đầu Chính sách Bảo mật (nếu chưa ở đó).
143. Đánh dấu mục `Block-from-Known-Bad-Addresses` nhưng không mở nó.
144. Ở cuối cửa sổ, chọn **Move** và chọn **Move Top**.
145. Cả hai quy tắc chặn lưu lượng đến/từ các địa chỉ IP xấu đã biết phải ở đầu Chính sách Bảo mật.
![[Pasted image 20260630101631.png]]

### Tạo Quy tắc Chính sách Bảo mật cho Truy cập Internet

Trong phần này,  sẽ tạo các quy tắc Chính sách Bảo mật để cho phép các máy chủ trong mạng của  truy cập Internet.  cần tạo một quy tắc cho máy chủ trong vùng bảo mật Users_Net để truy cập vào máy chủ trong vùng bảo mật Internet.  cũng cần tạo một quy tắc để cho phép máy chủ trong vùng bảo mật Extranet truy cập vào máy chủ trong vùng bảo mật Internet.![[Pasted image 20260630101713.png]]

#### Tạo Quy tắc Chính sách Bảo mật Users to Internet

146. Chọn **Policies > Security**.
147. Nhấp **Add** ở cuối cửa sổ.
148. Trong tab **General**, tại trường **Name**, nhập `Users_to_Internet`.
149. Tại **Description**, nhập `Allows hosts in Users_Net zone to access Internet zone`.
150. Giữ nguyên các thiết lập khác.![[Pasted image 20260630101731.png]]
151. Chọn tab **Source**.
152. Trong phần **Source Zone**, nhấp **Add**.
153. Chọn **Users_Net**.
154. Giữ nguyên các thiết lập còn lại.![[Pasted image 20260630101752.png]]
155. Chọn tab **Destination**.
156. Trong phần **Destination Zone**, nhấp **Add**.
157. Chọn **Internet**.
158. Giữ nguyên các thiết lập khác.![[Pasted image 20260630101804.png]]
159. Chọn tab **Application**.
160. Không thay đổi bất kỳ thiết lập nào, nhưng lưu ý hộp **Any** được chọn.![[Pasted image 20260630101825.png]]
161. Chọn tab **Service/URL Category**.
162. Không thay đổi bất kỳ thiết lập nào, nhưng lưu ý **Service** được đặt là `application-default`.![[Pasted image 20260630101847.png]]
163. Chọn tab **Actions**.
164. Đảm bảo **Action** được đặt là **Allow**.
165. Nhấp **OK** trên cửa sổ Security Policy Rule.![[Pasted image 20260630101908.png]]
166. Quy tắc mới xuất hiện trong bảng.
167. Đánh dấu quy tắc mới và sử dụng **Move > Move Bottom** để đặt quy tắc này ở cuối Chính sách Bảo mật.

![[Pasted image 20260630101958.png]]
#### Tạo Quy tắc Chính sách Bảo mật Extranet to Internet

 cũng cần tạo một quy tắc Chính sách Bảo mật để cho phép máy chủ trong vùng bảo mật Extranet truy cập vào máy chủ trong vùng bảo mật Internet.

168. Chọn **Policies > Security**.
169. Nhấp **Add** ở cuối cửa sổ.
170. Trong tab **General**, tại trường **Name**, nhập `Extranet_to_Internet`.
171. Tại **Description**, nhập `Allows hosts in Extranet zone to access Internet zone`.
172. Giữ nguyên các thiết lập khác.![[Pasted image 20260630102032.png]]
173. Chọn tab **Source**.
174. Trong phần **Source Zone**, nhấp **Add**.
175. Chọn **Extranet**.
176. Giữ nguyên các thiết lập còn lại.![[Pasted image 20260630102109.png]]
177. Chọn tab **Destination**.
178. Trong phần **Destination Zone**, nhấp **Add**.
179. Chọn **Internet**.
180. Giữ nguyên các thiết lập khác.![[Pasted image 20260630102127.png]]
181. Chọn tab **Application**.
182. Không thay đổi bất kỳ thiết lập nào, nhưng lưu ý hộp **Any** được chọn.
183. Chọn tab **Service/URL Category**.
184. Không thay đổi bất kỳ thiết lập nào, nhưng lưu ý **Service** được đặt là `application-default`.![[Pasted image 20260630102148.png]]
185. Chọn tab **Actions**.
186. Đảm bảo **Action** được đặt là **Allow**.![[Pasted image 20260630102214.png]]
187. Nhấp **OK** trên cửa sổ Security Policy Rule.
188. Quy tắc mới xuất hiện trong bảng.
189. Đặt quy tắc ở cuối Chính sách Bảo mật bằng cách sử dụng **Move > Move Bottom**.
	![[Pasted image 20260630102243.png]]

### Commit cấu hình

190. Nhấp nút **Commit** ở góc trên bên phải của giao diện web.
191. Giữ nguyên các thiết lập và nhấp **Commit**.
192. Chờ cho đến khi quá trình Commit hoàn tất.
193. Nhấp **Close** để tiếp tục.

### Ping Máy chủ Internet từ Client A

194. Để xác minh quy tắc Chính sách Bảo mật của  đang cho phép lưu lượng,  sẽ ping một máy chủ Internet từ máy trạm và kiểm tra Nhật ký Lưu lượng để xem kết quả.
195. Từ cửa sổ Terminal trên máy trạm, ping một địa chỉ trên internet bằng lệnh:

    ```
    lab-user@client-a:~/Desktop/Lab-Files$ ping 8.8.8.8
    ```

196.  sẽ không nhận được phản hồi, vì vậy sau vài giây, nhấn **Ctrl+C** để dừng ping.
197. Kiểm tra lại nhật ký lưu lượng và sử dụng bộ lọc để xem có mục nhập nào cho phiên này thất bại không.
198. Chọn **Monitor > Logs > Traffic**.
199. Trong trường bộ lọc, cập nhật cú pháp để bao gồm ứng dụng `ping`:

    ( addr.dst in 8.8.8.8 ) and ( app eq ping )

200. Nhấp nút **Apply filter** ở góc trên bên phải của cửa sổ (hoặc nhấn **Enter**).
201. Nhật ký Lưu lượng sẽ cập nhật và  sẽ thấy các mục khớp với bộ lọc.
202.  có thể thấy các phiên đang khớp với quy tắc `Users_to_Internet`.

203. **Trả lời câu hỏi sau:**
     ***có thể giải thích tại sao phiên ping từ máy khách đến máy chủ Internet không nhận được phản hồi mặc dù tường lửa đang cho phép lưu lượng không?***

    Gợi ý: Hãy xem tiêu đề của mô-đun tiếp theo.

Network Address Translation rules

---


