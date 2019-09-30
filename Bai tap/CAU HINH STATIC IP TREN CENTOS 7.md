


## Hướng dẫn cấu hình địa chỉ IP tĩnh trên CentOS 7
Mục lục
#### 1. [Liệt kê thông tin các Card mạng trên CentOS 7](#1)
#### 2. [Cấu hình IP tĩnh cho Card mạng](#2)
#### 3. [Khởi động network và kiểm tra cấu hình](#3)

<a name="1"></a>

1. Liệt kê thông tin các Card mạng trên CentOS 7  
Sử dụng lệnh `# ip link show` để xem thông tin card mạng

<img src="https://i.imgur.com/fcye2T2.png">
<a name="2"></a>
2. Cấu hình IP tĩnh cho Card mạng  

- Vào thư mục /etc/sysconfig/network-script/ và xem nội dung bên trong

<img src="https://i.imgur.com/2U1V4FN.png">

- Mở file *ifcfg-ens33* và tiến hành cấu hình địa chỉ IP tĩnh  
Sử dụng câu lệnh `# vi ifcfg-ens33` để chỉnh sửa file  

<img src="https://i.imgur.com/aZwjUeu.png">


Chỉnh sửa một vài thông tin trong file theo hình:  
```SH
BOOTPROTO=static  
ONBOOT=yes  
IPADDR= 192.168.152.10  
GATEWAY= 192.168.152.2  
PREFIX=24  
DNS1=8.8.8.8  
DNS2=8.8.8.4  
```
Sau khi chỉnh sửa file thì ấn Esc để thoát khỏi chế độ soạn thảo.  
Ấn :wq để lưu và thoát  
<a name ="3"></a>
3. Khởi động network và kiểm tra cấu hình  

Sau khi cấu hình xong ta tiến hành khởi động lại bằng lệnh:
`# systemctl restart network.service`  
Sau đó sử dụng lệnh `ip addr` để kiểm tra lại  

<img src="https://i.imgur.com/RHpAdc5.png">  

Như vậy bạn đã cài đặt được địa chỉ IP tĩnh trên CentOS7.