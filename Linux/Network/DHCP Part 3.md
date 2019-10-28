### Cách thêm tự động file cấu hình cho card mạng 

Thông thường khi ta gán thêm 1 card mạng vào máy thì máy chưa tự động tạo file cấu hình card mạng. Giả sử tôi add thêm 1 card và chọn chế độ Host-only. Tôi sẽ kiểm tra đã tồn tại thiết bị này chưa bằng lệnh `nmcli dev status`.   

<img src ="https://i.imgur.com/INoFuOu.png"> 

Như vậy card mạng này đã tồn tại nhưng chưa có địa chỉ IP. Hệ thống báo `getting IP configuration`. Ta tiếp tục đi đến thư mục `/etc/sysconfig/network-scripts` để xem đã có file cấu hình chưa  

<img src="https://i.imgur.com/excpViv.png">  

Ta thấy file cấu hình card `ens38` chưa tồn tại. Để thêm tự động file này ta tiến hành các bước sau:  
- *Bước 1*: Đổi tên `Wired connection 1` thành `ens38` bằng lệnh `nmcli c modify "Wired connection 1" connection.id ens38`.  

- *Bước 2*: Gõ lệnh `nmcli device modify ens38 ipv4.method auto`.  
Kiểm tra xem file cấu hình đã được tự động thêm vào hay chưa  
<img src="https://i.imgur.com/wrlnqhL.png">  

Ta dùng lệnh `cat` để xem nội dung file cấu hình  

<img src="https://i.imgur.com/xXZvVhx.png">  

Muốn card mạng `ens38` được có địa chỉ IP. Ta có thể cấu hình địa chỉ IP tĩnh hoặc IP động. Ở đây tôi sẽ cấp IP động cho nó bằng cách chỉnh nó về chế độ `VMnet2` kết nối với máy chủ server (Ở bài trước ta đã cấu hình cho máy server chế độ cấp phát IP động)

<img src="https://i.imgur.com/3PWI5U4.png">  

Restart network bằng lệnh `systemctl restart network-service` và kiểm tra NIC `ens38` đã có địa chỉ IP chưa bằng lệnh `ip a`  

<img src="https://i.imgur.com/rhrAKlX.png">  

Như vậy ta đã thực hiện thành công add tự động file cấu hình card mạng và cấp phát IP động thông qua lệnh nmcli và giao thức DHCP. 
