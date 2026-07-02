# Quản lý Tài khoản Quản trị Tường lửa

Khi triển khai tường lửa vào mạng sản xuất của mình, cần đảm bảo rằng các thành viên khác trong nhóm có quyền truy cập quản trị vào thiết bị.  muốn tận dụng một máy chủ LDAP hiện có để quản lý thông tin tài khoản và mật khẩu cho các thành viên trong nhóm. Tuy nhiên, tổ chức của  gần đây đã sáp nhập với một công ty khác, nơi các tài khoản quản trị được duy trì trong cơ sở dữ liệu RADIUS.

Chưa ai có thời gian di chuyển tất cả các tài khoản từ RADIUS sang LDAP, vì vậy  cần cấu hình tường lửa để kiểm tra cả LDAP và RADIUS nhằm xác thực tài khoản khi quản trị viên đăng nhập.![[Pasted image 20260629212945.png]]

---
## 1. Mục tiêu của Lab
- Tạo tài khoản quản trị viên cục bộ trên tường lửa
- Cấu hình Hồ sơ Máy chủ LDAP
- Cấu hình Hồ sơ Máy chủ RADIUS
- Cấu hình Hồ sơ Xác thực LDAP
- Cấu hình Hồ sơ Xác thực RADIUS
- Cấu hình Chuỗi Xác thực
- Tạo tài khoản quản trị viên không cục bộ

---
## 2. Các bước Tổng quan Cấp cao

### Tạo Hồ sơ Xác thực Cơ sở dữ liệu Cục bộ
Tạo Hồ sơ Xác thực Cơ sở dữ liệu Cục bộ có tên `Local-database`. Đặt Danh sách Cho phép (Allow List) cho Hồ sơ này là `all`.
### Tạo Tài khoản trong Cơ sở dữ liệu Người dùng Cục bộ
Tạo một mục trong Cơ sở dữ liệu Người dùng Cục bộ có tên `adminBob` với mật khẩu `Pal0Alt0!`.
### Tạo Tài khoản Quản trị viên
Tạo tài khoản Quản trị viên sử dụng mục Cơ sở dữ liệu Cục bộ cho `adminBob`. Đặt Hồ sơ Xác thực là `Local-database`.
### Commit cấu hình
Commit các thay đổi trước khi tiếp tục.
### Đăng nhập với Tài khoản Quản trị viên mới
- Đăng xuất khỏi giao diện web và đăng nhập lại với tên người dùng `adminBob` và mật khẩu `Pal0Alt0!`.
- Sử dụng Nhật ký Hệ thống (System log) để xác minh tài khoản `adminBob` đã được xác thực bằng `local-database`.
- Đăng xuất và đăng nhập lại với thông tin `admin/Pal0Alt0!`.
### Cấu hình Xác thực LDAP
Sử dụng thông tin trong bảng dưới đây để cấu hình Hồ sơ Máy chủ LDAP.

| Tham số | Giá trị |
|---------|---------|
| Tên Hồ sơ | LDAP-Server-Profile |
| Tên Máy chủ | ldap.panw.lab |
| Địa chỉ IP LDAP | 192.168.50.89 |
| Cổng | 389 |
| Loại Máy chủ | other |
| Base DN | dc=panw,dc=lab |
| Bind DN | cn=admin,dc=panw,dc=lab |
| Mật khẩu / Xác nhận | Pal0Alt0! |
| Yêu cầu kết nối SSL/TLS | Bỏ chọn |

Sử dụng thông tin trong bảng dưới đây để tạo Hồ sơ Xác thực LDAP.

| Tham số | Giá trị |
|---------|---------|
| Tên | LDAP-Auth-Profile |
| Loại | LDAP |
| Hồ sơ Máy chủ | LDAP-Server-Profile |
| Danh sách Cho phép (Tab Advanced) | all |

Sử dụng thông tin trong bảng dưới đây để tạo tài khoản quản trị viên mới sẽ được xác thực bằng LDAP.

| Tham số | Giá trị |
|---------|---------|
| Tên | adminSally |
| Hồ sơ Xác thực | LDAP-Auth-Profile |
### Commit cấu hình
Commit các thay đổi trước khi tiếp tục.
### Đăng nhập với Tài khoản Quản trị viên mới
- Kiểm tra Xác thực LDAP bằng cách đăng nhập với thông tin `adminSally/Pal0Alt0!`.
- Sử dụng Nhật ký Hệ thống để xác minh tài khoản `adminSally` đã được xác thực bằng LDAP.
### Cấu hình Xác thực RADIUS

Sử dụng thông tin trong bảng dưới đây để cấu hình Hồ sơ Máy chủ RADIUS.

| Tham số             | Giá trị               |
| ------------------- | --------------------- |
| Tên Hồ sơ           | RADIUS-Server-Profile |
| Giao thức Xác thực  | CHAP                  |
| Tên Máy chủ         | radius.panw.lab       |
| Máy chủ RADIUS      | 192.168.50.150        |
| Mật khẩu / Xác nhận | Pal0Alt0!             |
| Cổng                | 1812                  |

Sử dụng thông tin trong bảng dưới đây để tạo Hồ sơ Xác thực RADIUS.

| Tham số | Giá trị |
|---------|---------|
| Tên | RADIUS-Auth-Profile |
| Loại | RADIUS |
| Hồ sơ Máy chủ | RADIUS-Server-Profile |
| Danh sách Cho phép (Tab Advanced) | all |

Sử dụng thông tin trong bảng dưới đây để tạo tài khoản quản trị viên mới sẽ được xác thực bằng RADIUS.

| Tham số | Giá trị |
|---------|---------|
| Tên | adminHelga |
| Hồ sơ Xác thực | RADIUS-Auth-Profile |

### Commit cấu hình
Commit các thay đổi trước khi tiếp tục.
### Đăng nhập với Tài khoản Quản trị viên mới
- Kiểm tra Xác thực RADIUS bằng cách đăng nhập với thông tin `adminHelga/Pal0Alt0!`.
- Sử dụng Nhật ký Hệ thống để xác minh tài khoản `adminHelga` đã được xác thực bằng RADIUS.
### Cấu hình Chuỗi Xác thực
Tạo một chuỗi xác thực có tên `LDAP-then-RADIUS` sử dụng `LDAP-Auth-Profile` trước và `RADIUS-Auth-Profile` sau.
### Commit cấu hình

Commit các thay đổi trước khi tiếp tục.

---

## 3. Các bước Chi tiết

Sử dụng phần này nếu  muốn hướng dẫn chi tiết để hoàn thành các mục tiêu của lab. Chúng tôi đặc biệt khuyến nghị sử dụng phần này nếu  chưa có nhiều kinh nghiệm làm việc với tường lửa Palo Alto Networks.

### 3.1 Tạo Hồ sơ Xác thực Cơ sở dữ liệu Cục bộ

10. Tạo Hồ sơ Xác thực Cơ sở dữ liệu Cục bộ bằng cách chọn **Device > Authentication Profile**.
11. Nhấp **Add** ở cuối cửa sổ.
12. Trong tab **Authentication**, nhập `Local-database` cho **Name**.
13. Đối với **Type**, sử dụng danh sách thả xuống để chọn **Local Database**.
14. Giữ nguyên các thiết lập còn lại.
    ![[Pasted image 20260629213452.png]]
15. Chọn tab **Advanced**.
16. Trong phần **Allow List**, nhấp **Add**.
17. Chọn **all**.![[Pasted image 20260629213531.png]]

    > Các mục trong Allow List cho phép  chọn các thành viên riêng lẻ trong cơ sở dữ liệu cục bộ nếu  muốn giới hạn quyền truy cập vào tường lửa cho các quản trị viên cụ thể. Bằng cách chọn `all`,  cho phép bất kỳ tài khoản quản trị viên nào trong cơ sở dữ liệu cục bộ truy cập vào tường lửa.

18. Giữ nguyên các thiết lập còn lại.
19. Nhấp **OK**.

### 3.2 Tạo Tài khoản trong Cơ sở dữ liệu Người dùng Cục bộ

Trong phần này,  sẽ tạo một mục mới trong Cơ sở dữ liệu Người dùng Cục bộ trên tường lửa. Mục này dành cho thành viên mới trong nhóm, `adminBob`.

20. Chọn **Device > Local User Database > Users**.
21. Ở góc dưới bên trái của cửa sổ, nhấp **Add**.
22. Tại **Name**, nhập `adminBob`.
23. Nhập `Pal0Alt0!` cho **Password** và **Confirm Password**.
24. Giữ nguyên các thiết lập còn lại.
25. Nhấp **OK**.
![[Pasted image 20260629213635.png]]
### 3.3 Tạo Tài khoản Quản trị viên

Trong phần này,  sẽ tạo một tài khoản quản trị viên cho `adminBob`. Tài khoản `adminBob` sẽ sử dụng Hồ sơ Xác thực `Local-database`.

26. Tạo Tài khoản Quản trị viên từ người dùng Cơ sở dữ liệu Cục bộ bằng cách chọn **Device > Administrators**.
27. Nhấp **Add** ở cuối cửa sổ.
28. Tại **Name**, nhập `adminBob`.
29. Tại **Description**, nhập `Bob F. superuser admin`.
30. Đối với **Authentication Profile**, sử dụng danh sách thả xuống để chọn `Local-database`.
31. Giữ nguyên các thiết lập còn lại.
32. Nhấp **OK**.
![[Pasted image 20260629213657.png]]
### 3.4 Commit cấu hình

33. Nhấp nút **Commit** ở góc trên bên phải của giao diện web.
34. Giữ nguyên các thiết lập và nhấp **Commit**.
35. Chờ cho đến khi quá trình Commit hoàn tất.
36. Nhấp **Close** để tiếp tục.

### 3.5 Đăng nhập với Tài khoản Quản trị viên mới

37. Đăng xuất khỏi giao diện web của tường lửa bằng cách nhấp nút **Logout** ở góc dưới bên trái của cửa sổ.
38. Đăng nhập lại vào tường lửa với tên người dùng `adminBob` và mật khẩu `Pal0Alt0!`.
39. Đóng bất kỳ cửa sổ chào mừng nào xuất hiện.
40. Chọn **Monitor > System**.
41. Tìm mục có **Type** là `auth`.

![[Pasted image 20260629214058.png|697]]

 Nếu  không thấy mục trong Nhật ký Hệ thống cho biết xác thực thành công cho `adminBob`,  có thể tạo và áp dụng bộ lọc với cú pháp `(subtype eq auth)`.

42. Lưu ý rằng mục trong nhật ký hệ thống của tường lửa cho biết `adminBob` đã được xác thực thành công với `Local-database`.
43. Đăng xuất khỏi tường lửa.
44. Đăng nhập lại vào tường lửa với thông tin `admin/Pal0Alt0!`.

### 3.6 Cấu hình Xác thực LDAP

Tổ chức của  sử dụng máy chủ LDAP để duy trì cơ sở dữ liệu người dùng, bao gồm cả quản trị viên mạng. Nhóm nhân viên bảo mật của  đang tăng lên mỗi tháng và  muốn tận dụng máy chủ LDAP hiện có để xác thực quản trị viên khi họ cố gắng đăng nhập vào tường lửa.

Bước đầu tiên trong quy trình này là định nghĩa Hồ sơ Máy chủ LDAP chứa thông tin cụ thể mà tường lửa có thể sử dụng khi gửi truy vấn xác thực.

45. Chọn **Device > Server Profiles > LDAP**.
46. Ở cuối cửa sổ, nhấp **Add**.
47. Tại **Profile Name**, nhập `LDAP-Server-Profile`.
48. Trong phần **Server List**, nhấp **Add**.
49. Trong trường **Name**, nhập `ldap.panw.lab`.
50. Trong trường **LDAP Server**, nhập `192.168.50.89`.
51. Giữ nguyên trường **Port** là `389`.
52. Trong phần **Server Settings**, xác minh rằng **Type** được đặt là `other`.
53. Nhập `dc=panw,dc=lab` cho **Base DN**.
54. Nhập `cn=admin,dc=panw,dc=lab` cho **Bind DN**.
55. Nhập `Pal0Alt0!` cho **Password** và **Confirm Password**.
56. Bỏ chọn tùy chọn **Require SSL/TLS secured connection**.
57. Giữ nguyên các thiết lập còn lại.![[Pasted image 20260629214241.png]]

    > Lưu ý rằng không có dấu cách giữa các giá trị trong trường Base DN và Bind DN.

58. Nhấp **OK** để tạo Hồ sơ Máy chủ LDAP.

Với Hồ sơ Máy chủ LDAP đã sẵn sàng,  sẽ tạo Hồ sơ Xác thực và tham chiếu đến Hồ sơ Máy chủ LDAP vừa tạo.

59. Chọn **Device > Authentication Profile**.
60. Nhấp nút **Add** ở cuối cửa sổ.
61. Tại **Name**, nhập `LDAP-Auth-Profile`.
62. Trong tab **Authentication**, sử dụng danh sách thả xuống **Type** để chọn `LDAP`.
63. Trong **Server Profile**, sử dụng danh sách thả xuống để chọn `LDAP-Server-Profile`.
    ![[Pasted image 20260629214345.png]]
64. Chọn tab **Advanced**.
65. Trong phần **Allow List**, nhấp **Add**.
66. Chọn `all`.
67. Giữ nguyên các thiết lập còn lại.
68. Nhấp **OK**.![[Pasted image 20260629214405.png]]

69. Tạo một quản trị viên mới bằng cách chọn **Device > Administrators**.
70. Nhấp **Add**.
71. Tại **Name**, nhập `adminSally`.
72. Tại **Description**, nhập `Sally C superuser admin`.
73. Đối với **Authentication Profile**, sử dụng danh sách thả xuống để chọn `LDAP-Auth-Profile`.
74. Giữ nguyên các thiết lập còn lại.![[Pasted image 20260629214435.png]]

    > Tài khoản `adminSally` là một tài khoản tồn tại trên máy chủ LDAP.

75. Nhấp **OK**.

### 3.7 Commit cấu hình

76. Nhấp nút **Commit** ở góc trên bên phải của giao diện web.
77. Giữ nguyên các thiết lập và nhấp **Commit**.
78. Chờ cho đến khi quá trình Commit hoàn tất.
79. Nhấp **Close** để tiếp tục.

### 3.8 Đăng nhập với Tài khoản Quản trị viên mới

80. Đăng xuất khỏi tường lửa bằng cách nhấp nút **Logout** ở góc dưới bên trái của cửa sổ.
81. Đăng nhập lại vào tường lửa với tên người dùng `adminSally` và mật khẩu `Pal0Alt0!`.
82. Đóng bất kỳ cửa sổ chào mừng nào xuất hiện.
83. Chọn **Monitor > System**.
84. Tìm mục có **Type** là `auth`.
![[Pasted image 20260629214512.png]]

85. Lưu ý rằng mục trong nhật ký hệ thống của tường lửa cho biết `adminSally` đã được xác thực thành công với `LDAP-Auth-Profile`.
86. Đăng xuất khỏi tường lửa.
87. Đăng nhập lại vào tường lửa với thông tin `admin/Pal0Alt0!`.

### 3.9 Cấu hình Xác thực RADIUS

Tổ chức của  gần đây đã mua lại một công ty khác. Công ty mới được mua lại duy trì tất cả tài khoản quản trị viên mạng trong một máy chủ RADIUS.  cần tích hợp xác thực RADIUS cho tường lửa để các quản trị viên mạng mới đã gia nhập nhóm của  có thể truy cập tường lửa cho mục đích quản lý.

88. Tạo Hồ sơ Máy chủ RADIUS bằng cách chọn **Device > Server Profiles > RADIUS**.
89. Nhấp **Add**.
90. Tại **Name**, nhập `RADIUS-Server-Profile`.
91. Đối với **Authentication Protocol**, sử dụng danh sách thả xuống để chọn `CHAP`.

    > Lưu ý: Không bao giờ sử dụng CHAP trong môi trường sản xuất vì nó không an toàn. Chúng tôi sử dụng nó trong lab để đơn giản hóa.

92. Trong phần **Servers**, nhấp **Add**.
93. Trong trường **Name** của máy chủ, nhập `radius.panw.lab`.
94. Trong trường **RADIUS Server**, nhập `192.168.50.150`.
95. Nhập `Pal0Alt0!` cho **Secret** và **Confirm Secret**.
96. Giữ nguyên **Port** là `1812`.
97. Giữ nguyên các thiết lập còn lại.
    ![[Pasted image 20260629214710.png]]
98. Nhấp **OK**.

99. Tạo Hồ sơ Xác thực RADIUS bằng cách chọn **Device > Authentication Profile**.
100. Nhấp **Add**.
101. Tại **Name**, nhập `RADIUS-Auth-Profile`.
102. Đối với **Type**, chọn `RADIUS`.
103. Đối với **Server Profile**, chọn `RADIUS-Server-Profile`.
104. Giữ nguyên các thiết lập còn lại.
     ![[Pasted image 20260629214752.png]]
105. Chọn tab **Advanced**.
106. Trong phần **Allow List**, nhấp **Add**.
107. Chọn `all`.
108. Giữ nguyên các thiết lập còn lại.
109. Nhấp **OK**.![[Pasted image 20260629214806.png]]

110. Tạo tài khoản quản trị viên cho `adminHelga` (người vừa gia nhập nhóm của  từ công ty được mua lại) bằng cách chọn **Device > Administrators**.
111. Nhấp **Add**.
112. Tại **Name**, nhập `adminHelga`.
113. Tại **Description**, nhập `Helga R superuser admin`.
114. Đối với **Authentication Profile**, chọn `RADIUS-Auth-Profile`.
115. Giữ nguyên các thiết lập còn lại.![[Pasted image 20260629214850.png]]
116. Nhấp **OK**.

117. Nhấp nút **Commit** ở góc trên bên phải của giao diện web.
118. Giữ nguyên các thiết lập và nhấp **Commit**.
119. Chờ cho đến khi quá trình Commit hoàn tất.
120. Đăng xuất khỏi tường lửa bằng cách nhấp nút **Logout** ở góc dưới bên trái của cửa sổ.
121. Đăng nhập lại vào tường lửa với tên người dùng `adminHelga` và mật khẩu `Pal0Alt0!`.
122. Đóng bất kỳ cửa sổ chào mừng nào xuất hiện.
123. Chọn **Monitor > System**.
124. Tìm mục có **Type** là `auth`.
![[Pasted image 20260629214949.png]]
Nếu  không thấy mục trong Nhật ký Hệ thống cho biết xác thực thành công cho `adminHelga`,  có thể sử dụng bộ lọc `(subtype eq auth)`.

125. Lưu ý rằng mục trong nhật ký hệ thống của tường lửa cho biết `adminHelga` đã được xác thực thành công với `RADIUS-Auth-Profile`.
126. Đăng xuất khỏi tường lửa.
127. Đăng nhập lại vào tường lửa với thông tin `admin/Pal0Alt0!`.

### 3.10 Cấu hình Chuỗi Xác thực

Sau khi sáp nhập, một số tài khoản quản trị viên tồn tại trong LDAP và các tài khoản khác tồn tại trong RADIUS. Với các tài khoản quản trị viên trong hai hệ thống khác nhau này,  cần cấu hình tường lửa để có thể kiểm tra cả hai cơ sở dữ liệu bên ngoài khi quản trị viên cố gắng đăng nhập.

 sẽ thực hiện điều này bằng cách tạo một Chuỗi Xác thực. Chuỗi sẽ hướng dẫn tường lửa kiểm tra một tài khoản trong LDAP trước và sau đó trong RADIUS nếu tài khoản không tồn tại trong LDAP (hoặc nếu máy chủ LDAP không khả dụng).

128. Chọn **Device > Authentication Sequence**.
129. Nhấp **Add**.
130. Tại **Name**, nhập `LDAP-then-RADIUS`.
131. Trong phần **Authentication Profiles**, nhấp **Add**.
132. Chọn `LDAP-Auth-Profile`.
133. Nhấp **Add** lần nữa.
134. Chọn `RADIUS-Auth-Profile`.
135. Giữ nguyên các thiết lập còn lại.
136. Nhấp **OK**.![[Pasted image 20260629215237.png]]

### 3.11 Commit cấu hình

137. Nhấp nút **Commit** ở góc trên bên phải của giao diện web.
138. Giữ nguyên các thiết lập và nhấp **Commit**.
139. Chờ cho đến khi quá trình Commit hoàn tất.
140. Nhấp **Close** để tiếp tục.

---
