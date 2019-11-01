## Cài đặt LAMP trên CentOS7  

### 1. Cài đặt Apache web server**  
- Kiểm tra apache đã được cài đặt trong hệ thống hay chưa:  
```
rpm -q httpd
package httpd is not installed
```
```
ls -l /var/www
ls: cannot access /var/www: No such file or directory
```  
>`httpd`: là dịch vụ web chính của máy chủ, khi httpd chết thì web bạn cũng chết theo, httpd tạo ra nhiều PID (process id-số hiệu tiến trình) để phục vụ website của bạn, càng nhiều tiến trình thì càng tốn ram, khi bị DDOS (Distributed Denial of Service-tấn công từ chối truy cập) thì quá nhiều tiến trình httpd tạo ra có thể dẫn đến VPS treo. httpd thường được cài php vào để phục vụ mã nguồn php. Các lỗi không vào được web phần lớn liên quan đến dịch vụ này.  
- Cài đặt gói httpd 
```
yum install -y httpd
```
Việc cài đặt diễn ra nhanh chóng, trước khi cấu hình chi tiết Apache (httpd) các bạn nên backup file config để phòng trường hợp sai sót còn có cái phục hồi lại.  
```
cp /etc/httpd/conf/httpd.conf ~/httpd.conf.bak
```  
Câu lệnh sẽ tạo một file dự phòng có tên httpd.conf.bak trong profile của root. Khi bạn  muốn khôi phục cấu hình về mặc định chỉ cần sửa lại tên file là được.  

Khởi động web server Apache và cấu hình startup service cho Apache.
```
systemctl start httpd
systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```
- Kiểm tra phiên bản apache đã cài đặt  
```
http -v
Server version: Apache/2.4.6 (CentOS)
Server built:   Aug  8 2019 11:41:18
```
- Kiểm tra dịch vụ Apache đã lắng nghe cổng 80 chưa:
```
[root@centos7server ~]# netstat -nap | grep httpd
tcp6       0      0 :::80                   :::*                    LISTEN      2006/httpd
unix  3      [ ]         STREAM     CONNECTED     24478    2006/httpd
```
=> Đã lắng nghe được port 80. Nếu máy bạn chưa có lệnh `netstat` thì tiến hành cài đặt `net-tools`.  
- Kiểm tra cài đặt apache và module mpm prefork đã start hay chưa thì dùng lệnh sau: 
<img src="https://i.imgur.com/4WncAzh.png">  

<img src="https://i.imgur.com/9OZh4C4.png">

Kết quả như trên là đã start thành công rồi, Module pmp prefork, apache đã chạy với `pid 20149`, Web server đang lắng nghe trên `port 80`.  
- Tắt tường lửa trên máy cài Apache  
```
systemctl stop firewalld
``` 
> `Firewalld` là một phần của hệ thống máy tính hoặc mạng được thiết kế để chặn truy cập trái phép trong khi cho phép liên lạc được ủy quyền. Nó là 1 thiết bị hoặc 1 bộ thiết bị được cấu hình để: cho phép, từ chối, mã hóa, giải mã hoặc ủy quyền tất cả lưu lượng máy tính (vào và ra) giữa các miền bảo mật khác nhau dựa trên 1 bộ quy tắc và tiêu chí.  
`SELinux` (Security-Enhanced Linux) là một tính năng của Linux cung cấp cơ chế hỗ trợ các chính sách bảo mật, kiểm soát truy cập vào ứng dụng/file.  
*Tại sao phải tắt firewall, SELinux*?   
httpd chạy trên port 80, firewalld trên CentOS7 mặc định block port 80 trên server => tắt firewall/mở port 80 để có thể truy cập dịch vụ web.

Để xem từ ngoài internet đã truy cập được website hay chưa bạn mở trình duyệt ra rồi nhập địa chỉ IP Public của máy chủ ảo. Nếu hiện ra trang Apache 2 Test Page là bạn đã thành công rồi.
```
http:// 192.168.40.250
```
Kết quả:

<img src="https://i.imgur.com/ktEJafN.png">  

Nếu trong mạng có firewall hoặc router, mở port cho Apache server trong trường hợp muốn truy cập thông qua remote.  
```
firewall-cmd --permanent --add-service=http
systemctl restart firewallp
```

**Cấu hình vitual host trên apache**  
Nếu muốn chạy nhiều website trên một VPS thì bạn phải tạo Virtual Host cho Apache, kể cả khi chỉ chạy 1 website thì bạn vẫn nên tạo để thuận tiện cho việc cài cắm sau này.  
- **Tạo thư mục cần thiết và phân quyền**  

Khi triển khai bất kỳ một trang web nào thì ta nên tạo một thư mục có tên là domain của trang web. 
```
mkdir -p /var/www/ngahong.com/public_html
```

>public_html: nơi bạn sẽ để source code của trang web  
index.html: là tệp chứa soure code.  

Tạo và chỉnh sửa một file index.html  
```
CTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>ngahong.com</title>
  </head>
  <body>
    <h1>This is my website - ngahong.com!</h1>
  </body>
</html>
``` 

- Tiếp theo cần phân quyền lại cho thư mục vì chúng được tạo từ tài khoản root, ta sẽ chuyển quyền sở hữu cho user apache, là user quản trị webserer.  
```
chown -R apache:apache /var/www/ngahong.com/public_html
chmod -R 755 /var/www
```
- **Tạo và chỉnh sửa tập tin máy chủ ảo**  
Tạo file `ngahong.com.conf` chứa tập tin máy chủ ảo: `/etc/httpd/conf.d/ngahong.com.conf`
- Nếu truy cập bằng domain thì nội dung file sẽ là:
```
<VirtualHost *:80>
    ServerName ngahong.com
    ServerAlias www.ngahong.com
    ServerAdmin webmaster@ngahong.com
    DocumentRoot "/var/www/ngahong.com/public_html"
    ErrorLog "/var/log/httpd/ngahong.com-error_log"
    CustomLog "/var/log/httpd/ngahong.com-access_log" combined
    <Directory "/var/www/ngahong.com/public_html">
       DirectoryIndex index.html
       Options FollowSymLinks
       AllowOverride All
       Require all granted
    </Directory>
</VirtualHost>
```
Nếu truy cập bằng địa chỉ ip và cổng thì nội dung file sẽ là:  

```
Listen 8081
<VirtualHost *:8081>
    ServerName 192.168.40.250
    ServerAlias www.ngahong.com
    ServerAdmin webmaster@ngahong.com
    DocumentRoot "/var/www/ngahong.com/public_html"
    ErrorLog "/var/log/httpd/ngahong.com-error_log"
    CustomLog "/var/log/httpd/ngahong.com-access_log" combined
    <Directory "/var/www/ngahong.com/public_html">
       DirectoryIndex index.html
       Options FollowSymLinks
       AllowOverride All
       Require all granted
    </Directory>
</VirtualHost>
```
