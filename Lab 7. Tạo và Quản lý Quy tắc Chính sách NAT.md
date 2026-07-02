# Lab 7: Tạo và Quản lý Quy tắc Chính sách NAT

 cần tạo các quy tắc Network Address Translation (NAT) để cho phép các máy chủ trong không gian mạng riêng (192.168.1.0/24 và 192.168.50.0/24) truy cập được tới các máy chủ trên internet.  sẽ sử dụng địa chỉ IP trên giao diện của tường lửa làm địa chỉ nguồn cho NAT ra ngoài (outbound).

 cũng sẽ tạo một địa chỉ NAT tĩnh trên tường lửa đại diện cho một trong các máy chủ ứng dụng trong vùng Extranet. Khi lưu lượng đến địa chỉ NAT tĩnh này, tường lửa sẽ dịch và chuyển tiếp các gói tin đến máy chủ web trong vùng Extranet.

Sau khi có tất cả các thành phần này,  sẽ tạo lưu lượng thử nghiệm và kiểm tra nhật ký của tường lửa.

---
## Mục tiêu của Lab

- Cấu hình NAT nguồn (Source NAT)
- Cấu hình NAT đích (Destination NAT)

---
## Các bước Tổng quan Cấp cao
### Tạo Quy tắc Chính sách NAT Nguồn

Sử dụng thông tin trong các bảng bên dưới để tạo một Quy tắc NAT Đích mới.

**Tab General (Tổng quát)**

| Tham số | Giá trị |
|---------|---------|
| Tên | Inside_Nets_to_Internet |
| Loại NAT | ipv4 |
| Mô tả | Dịch lưu lượng từ Users_Net và Extranet sang 203.0.113.20 ra ngoài Internet |

**Tab Original Packet (Gói tin Gốc)**

| Tham số | Giá trị |
|---------|---------|
| Vùng nguồn | Users_Net <br> Extranet |
| Vùng đích | Internet |
| Giao diện đích | ethernet1/1 |
| Dịch vụ | any |
| Địa chỉ nguồn | Any |
| Địa chỉ đích | Any |

**Tab Translated Packet (Gói tin Đã dịch) - Phần Source Address Translation (Dịch địa chỉ Nguồn)**

| Tham số | Giá trị |
|---------|---------|
| Loại Dịch | Dynamic IP And Port |
| Loại Địa chỉ | Interface Address |
| Giao diện | ethernet1/1 |
| Địa chỉ IP | 203.0.113.20/24 |

### Commit cấu hình

Commit các thay đổi trước khi tiếp tục.

### Xác minh Kết nối Internet

- Từ cửa sổ Terminal trên máy trạm, ping đến `8.8.8.8`.  sẽ nhận được phản hồi.
- Sử dụng trình duyệt kiểm tra để truy cập `www.paloaltonetworks.com`.
- Truy cập một vài trang web khác để xác minh  có thể thiết lập kết nối đến vùng Internet.
- Kiểm tra Nhật ký Lưu lượng của tường lửa để xác minh có lưu lượng được phép khớp với quy tắc Chính sách Bảo mật `Users_to_Internet`.

### Tạo Quy tắc NAT Đích

Sử dụng thông tin trong các bảng bên dưới để tạo một địa chỉ NAT Đích trên tường lửa sử dụng địa chỉ IP trên mạng Users_Net. Tường lửa sẽ dịch lưu lượng đến địa chỉ này thành địa chỉ IP đích của máy chủ web trong vùng Extranet.

**Tab General (Tổng quát)**

| Tham số | Giá trị |
|---------|---------|
| Tên | Dest_NAT_To_Webserver |
| Loại NAT | ipv4 |

**Tab Original Packet (Gói tin Gốc)**

| Tham số | Giá trị |
|---------|---------|
| Vùng nguồn | Users_Net |
| Vùng đích | Users_Net |
| Giao diện đích | ethernet1/2 |
| Dịch vụ | any |
| Địa chỉ đích | 192.168.1.80 |

**Tab Translated Packet (Gói tin Đã dịch) - Phần Destination Address Translation (Dịch địa chỉ Đích)**

| Tham số | Giá trị |
|---------|---------|
| Loại Dịch | Static IP |
| Địa chỉ đã dịch | 192.168.50.80 |

### Commit cấu hình

Commit các thay đổi trước khi tiếp tục.

### Kiểm tra Quy tắc NAT Đích

- Sử dụng trình duyệt kiểm tra và truy cập `http://192.168.1.80` để xác minh truy cập trang web của máy chủ Extranet.
- Tìm kiếm trong Nhật ký Lưu lượng để xác định các mục có địa chỉ IP đích là `192.168.1.80`.
- Trong cửa sổ Chính sách Bảo mật, sử dụng tùy chọn Log Viewer cho quy tắc `Users_to_Extranet` để chuyển đến các mục trong Nhật ký Lưu lượng khớp với quy tắc.

---
## Các bước Chi tiết
### Tạo Quy tắc Chính sách NAT Nguồn

 phải tạo các mục trong bảng Chính sách NAT của tường lửa để dịch lưu lượng từ các máy chủ nội bộ (thường nằm trong mạng riêng) sang một địa chỉ có thể định tuyến công khai (thường là một giao diện trên chính tường lửa). Các quy tắc NAT cung cấp dịch địa chỉ và khác với các quy tắc Chính sách Bảo mật, vốn cho phép và từ chối các gói tin.  có thể cấu hình quy tắc Chính sách NAT để khớp với vùng nguồn và vùng đích của gói tin, giao diện đích, địa chỉ nguồn và địa chỉ đích, cũng như dịch vụ.

Trong bài kiểm tra ping trước đó đến một máy chủ Internet, lưu lượng ping từ máy khách của  được quy tắc Chính sách Bảo mật cho phép, nhưng các gói tin rời khỏi tường lửa với địa chỉ IP nguồn không thể định tuyến từ mạng riêng 192.168.1.0/24.

Trong phần này,  sẽ tạo một quy tắc Chính sách NAT để dịch lưu lượng từ các mạng riêng trong vùng bảo mật Users_Net và Extranet sang một địa chỉ có thể định tuyến.  sẽ sử dụng cùng địa chỉ IP giao diện trên tường lửa (203.0.113.20) làm địa chỉ IP nguồn cho lưu lượng đi ra từ cả hai vùng Users_Net và Extranet.![[Pasted image 20260701222144.png]]

10. Trên giao diện web, chọn **Policies > NAT**.
11. Nhấp **Add** để định nghĩa một quy tắc NAT nguồn mới.
    Cửa sổ cấu hình NAT Policy Rule sẽ mở ra.
12. Cấu hình các thông số sau:

    | Tham số | Giá trị |
    |---------|---------|
    | Tên | Inside_Nets_to_Internet |
    | Loại NAT | Đảm bảo `ipv4` được chọn |
    | Mô tả | Dịch lưu lượng từ Users_Net và Extranet sang 203.0.113.20 ra ngoài Internet |

13. Nhấp vào tab **Original Packet** và cấu hình:

    | Tham số | Giá trị |
    |---------|---------|
    | Vùng nguồn | Nhấp **Add** và chọn vùng `Users_Net` <br> Nhấp **Add** và chọn vùng `Extranet` |
    | Vùng đích | Chọn `Internet` từ danh sách thả xuống |
    | Giao diện đích | Chọn `ethernet1/1` từ danh sách thả xuống |
    | Dịch vụ | Đảm bảo `any` được chọn |
    | Địa chỉ nguồn | Đảm bảo hộp **Any** được chọn |
    | Địa chỉ đích | Đảm bảo hộp **Any** được chọn |![[Pasted image 20260701222541.png]]

    > Phần này định nghĩa gói tin sẽ như thế nào khi đến tường lửa. Lưu ý rằng chúng ta đang sử dụng một quy tắc NAT duy nhất để dịch cả hai vùng nguồn sang cùng một giao diện trên tường lửa.  có thể thực hiện cùng một tác vụ bằng cách tạo hai quy tắc riêng biệt - một cho mỗi vùng nguồn - và sử dụng cùng một giao diện tường lửa bên ngoài.

14. Nhấp vào tab **Translated Packet** và cấu hình phần **Source Address Translation** như sau:

    | Tham số | Giá trị |
    |---------|---------|
    | Loại Dịch | Chọn `Dynamic IP And Port` từ danh sách thả xuống |
    | Loại Địa chỉ | Chọn `Interface Address` từ danh sách thả xuống |
    | Giao diện | Chọn `ethernet1/1` từ danh sách thả xuống |
    | Địa chỉ IP | Chọn `203.0.113.20/24` từ danh sách thả xuống. (Đảm bảo rằng  chọn địa chỉ IP giao diện từ danh sách thả xuống và không nhập thủ công.) |![[Pasted image 20260701222759.png]]

    > Phần này định nghĩa cách tường lửa sẽ dịch gói tin.
    > Lưu ý:  chỉ đang cấu hình phần Source Address Translation của cửa sổ này. Để phần Destination Address Translation Translation Type là `None`.

15. Nhấp **OK** để đóng cửa sổ cấu hình NAT Policy Rule.
16. Xác minh rằng cấu hình của  khớp với bảng sau:
![[Pasted image 20260701222907.png]]


    (Lưu ý: Một số cột đã bị ẩn trong hình ảnh.)

### Commit cấu hình

17. Nhấp nút **Commit** ở góc trên bên phải của giao diện web.
18. Giữ nguyên các thiết lập và nhấp **Commit**.
19. Chờ cho đến khi quá trình Commit hoàn tất.
20. Nhấp **Close** để tiếp tục.

### Xác minh Kết nối Internet

Trong phần này,  sẽ kiểm tra cấu hình của các chính sách NAT và Bảo mật của mình.

21. Từ cửa sổ Terminal trên máy trạm, ping một địa chỉ trên internet bằng lệnh:

    ```
    lab-user@client-a:~/Desktop/Lab-Files$ ping 8.8.8.8
    ```

     sẽ nhận được phản hồi.

22. Sau vài giây, nhấn **Ctrl+C** để dừng ping.
23. Mở trình duyệt kiểm tra và truy cập `www.paloaltonetworks.com`.
24. Truy cập một vài trang web khác để xác minh  có thể thiết lập kết nối đến vùng Internet.
25. Đóng trình duyệt kiểm tra.
26. Trong trình duyệt cấu hình, kiểm tra Nhật ký Lưu lượng của tường lửa bằng cách chọn **Monitor > Logs > Traffic**.
27. Xóa bất kỳ bộ lọc nào  đã đặt bằng cách nhấp nút **Clear Filter** ở góc trên bên phải của cửa sổ.
28. Xác minh rằng có lưu lượng được phép khớp với quy tắc Chính sách Bảo mật `Users_to_Internet`:

    | THỜI GIAN NHẬN | VÙNG NGUỒN | VÙNG ĐÍCH | NGUỒN | ĐÍCH | CỔNG ĐÍCH | ỨNG DỤNG | HÀNH ĐỘNG | QUY TẮC |
    |----------------|------------|-----------|-------|------|-----------|-----------|-----------|---------|
    | 03/03 18:36:29 | Users_Net | Internet | 192.168.1.254 | 8.8.8.8 | 53 | dns | allow | Users_to_Internet |
    | 03/03 18:36:19 | Users_Net | Extranet | 192.168.1.25 | 192.168.50.53 | 53 | dns | allow | Users_to_Extranet |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    > Các mục nhập trong nhật ký lưu lượng sẽ xuất hiện dựa trên bài kiểm tra internet. Có thể mất một hoặc hai phút để các file nhật ký được cập nhật. Nếu các mục chưa xuất hiện, hãy nhấp vào biểu tượng làm mới.

### Tạo Quy tắc NAT Đích

Trong phần này,  sẽ tạo một địa chỉ NAT trên tường lửa sử dụng địa chỉ IP trên mạng Users_Net. Tường lửa sẽ dịch lưu lượng đến địa chỉ này thành địa chỉ IP đích của máy chủ web trong vùng Extranet.
sẽ kết nối từ máy chủ (192.168.1.20) đến địa chỉ IP NAT trên tường lửa (192.168.1.80). Tường lửa sẽ dịch kết nối này đến máy chủ DMZ tại 192.168.50.10.![[Pasted image 20260701222954.png]]

29. Trên giao diện web, chọn **Policies > NAT**.
30. Nhấp **Add** để định nghĩa một quy tắc Chính sách NAT đích mới.
    Cửa sổ cấu hình NAT Policy Rule sẽ mở ra.
31. Cấu hình các thông số sau:

    | Tham số | Giá trị |
    |---------|---------|
    | Tên | Dest_NAT_To_webserver |
    | Mô tả | Dịch lưu lượng đến máy chủ web tại 192.168.50.80 |
    | Loại NAT | Đảm bảo `ipv4` được chọn |![[Pasted image 20260701223621.png]]

32. Nhấp vào tab **Original Packet** và cấu hình:

    | Tham số | Giá trị |
    |---------|---------|
    | Vùng nguồn | Nhấp **Add** và chọn `Users_Net` |
    | Vùng đích | Chọn `Users_Net` từ danh sách thả xuống |
    | Giao diện đích | Chọn `ethernet1/2` từ danh sách thả xuống |
    | Dịch vụ | Chọn `any` từ danh sách thả xuống |
    | Địa chỉ đích | Nhấp **Add** và nhập thủ công `192.168.1.80` |![[Pasted image 20260701223637.png]]

    > Tab Original Packet định nghĩa gói tin sẽ như thế nào khi đến tường lửa. Khi chọn Destination Zone, hãy nhớ rằng địa chỉ IP chúng ta đang sử dụng (192.168.1.80) là địa chỉ nằm trên tường lửa trong vùng bảo mật Users_Net.

33. Nhấp vào tab **Translated Packet** và cấu hình:

    | Tham số | Giá trị |
    |---------|---------|
    | Loại Dịch Địa chỉ Đích | Chọn `Static IP` từ danh sách thả xuống |
    | Địa chỉ đã dịch | Nhập `192.168.50.80` (địa chỉ của máy chủ web Extranet) |![[Pasted image 20260701223726.png]]

    > Tab Translated Packet định nghĩa cách tường lửa sẽ dịch một gói tin khớp. Để phần Source Address Translation là `None` vì chúng ta chỉ thực hiện dịch đích trong bài tập này.

34. Nhấp **OK** để đóng cửa sổ cấu hình NAT Policy Rule.
    Một quy tắc Chính sách NAT mới sẽ hiển thị trên giao diện web.
35. Xác minh rằng cấu hình của  khớp với:
![[Pasted image 20260701223806.png]]
### Commit cấu hình

36. Nhấp nút **Commit** ở góc trên bên phải của giao diện web.
37. Giữ nguyên các thiết lập và nhấp **Commit**.
38. Chờ cho đến khi quá trình Commit hoàn tất.
39. Nhấp **Close** để tiếp tục.

---
