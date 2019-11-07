### Configure apache on Ubuntu  
1. Cài đặt apache trên Ubuntu  
- Để kiểm tra apache đã được cài đặt hay chưa, ta sử dụng lệnh `ls -l /var/www`  
```
root@ubuntusrv:~# ls -l /var/www
ls: cannot access '/var/www': No such file or directory
root@ubuntusrv:~# dpkg -l | grep apache
```  
Như vậy hệ thống chưa cài đặt apache nên ta sẽ tiến hành cài đặt nó  
```
aptitude install apache2
```  
Sau khi cài đặt ta kiểm tra bằng lệnh:  
```
root@ubuntusrv:~# ls -l /var/www
total 4
drwxr-xr-x 2 root root 4096 Nov  5 01:19 html
``` 
- Xem trạng thái apache trên Ubuntu ta sử dụng lệnh
```
service apache2 status
```
- Khởi động apache2 và dùng lệnh `ps` để xem tiến trình  
```
root@ubuntusrv:~# service apache2 start
root@ubuntusrv:~# ps -C apache2
   PID TTY          TIME CMD
  3528 ?        00:00:00 apache2
  3530 ?        00:00:00 apache2
  3531 ?        00:00:00 apache2
```  
Hoặc sử dụng `wget` và `file index.html` để xác thực web server của bạn đã tồn tại html document chưa   
```
root@ubuntusrv:~# wget 127.0.0.1
--2019-11-05 01:52:02--  http://127.0.0.1/
Connecting to 127.0.0.1:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10918 (11K) [text/html]
Saving to: ‘index.html’

index.html                         100%[=============================================================>]  10.66K  --.-KB/s    in 0s

2019-11-05 01:52:02 (118 MB/s) - ‘index.html’ saved [10918/10918]
root@ubuntusrv:~# file index.html
index.html: HTML document, ASCII text
```  
Hoặc xác minh apache đang chạy hay không thì bạn hãy mở trình duyệt và gõ địa chỉ IP của máy, một trang thử nghiệm apache sẽ hiển thị. 

<img src="https://i.imgur.com/AKxFOlY.png">  

Bạn có thể làm như sau để nhanh chóng tránh thông báo '*could not reliably determine the fqdn (không thể xác định tin cậy fqdn)*' khi khởi động lại apache.  
```
root@ubuntusrv:~# echo ServerName ubuntusrv >> /etc/apache2/apache2.conf
root@ubuntusrv:~# service apache2 restart
```  
**default website**  
Thay đổi website mặc định của apache server mới cài đặt rất đơn giản, tất cả bạn cần là tạo (hoặc thay đổi) file index.html 