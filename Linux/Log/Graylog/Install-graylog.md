## Cài đặt Graylog  
Graylog có thể cài đặt trên nhiều hệ điều hành khác nhau:   
- Debian  
- Ubuntu  
- SLES
- RHEL/CentOS  

Ngoài ra để Graylog server hoạt động, cần cài thêm gói Elastisearch 5 hoặc 6, MongoDB 3.6 hoặc 4.0, Oracle Java SE 8.  
**Chú ý**:  
- Graylog từ bản 2.3 trở về trước không làm việc được với Elasticsearch 5.x!  
- Graylog 3.x không làm việc với Elasticsearch 7.x!  

### Điều kiện tiên quyết 

- Cài đặt gói bổ sung  
```
yum install java-1.8.0-openjdk-headless.x86_64  
```  
### MongoDB 

- Cài đặt MongoDB trên CentOS nên theo hướng dẫn tại [đây](https://docs.mongodb.com/master/tutorial/install-mongodb-on-red-hat/).   
- Đầu tiên thêm file repository `/etc/yum.repos.d/mongodb-org.repo`. Thêm nội dung sau vào file  
```
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```  
- Tiếp theo cài đặt `mongodb-org`  
```
yum install mongodb-org
```  
- Ngoài ra thêm các bước cuối này để khởi động và kích hoạt mongod  
```
systemctl daemon-reload
systemctl enable mongod.service
systemctl start mongod.service
```  
### Elasticsearch  

Graylog có thể sử dụng Elasticsearch 6.x 
- Cài đặt Elastic GPG key với 
```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
``` 
Sau đó thêm file repository `/etc/yum.repos.d/elasticsearch.repo` và ghi nội dung sau vào file  
```
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/oss-6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```  
- Thiết lập xác thực trong file cấu hình `/etc/elasticsearch/elasticsearch.yml` bằng cách sửa `cluster name` và thêm vào cuối file dòng `action.auto_create_index: false`  
```
cluster.name graylog
action.auto_create_index: false  
```  
- Sau đó xác thực file cấu hình  
```
 systemctl daemon-reload
 systemctl enable elasticsearch.service
 systemctl restart elasticsearch.service  
 ```
### Graylog  
- Cài đặt Graylog repository configuration và graylog  
```
rpm -Uvh https://packages.graylog2.org/repo/packages/graylog-3.1-repository_latest.rpm  
yum install graylog-server
```  
- Trong file `/etc/graylog/server/server.conf` cần khai báo `password_secret` và `root_password_sha2`. Các cài đặt này là bắt buộc và nếu không có chúng Graylog sẽ không khởi động. 
- Tạo chuỗi hash cho mật khẩu mà bạn muốn sử dụng
```
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```  
- Nhập passwprd xong sẽ hiển thị một dòng key. Hãy note lại nó để ta dùng cho bước sau này.  
- Dùng chuỗi được sinh ra ở bước trên để cấu hình tham số root_password_sha2 trong file cấu hình của graylog.  
```
sed -i 's|root_password_sha2 =|root_password_sha2 = bac9e3617b6d006286ff26e7ec567a64f4443491b648142d6939343f2549688b|g' /etc/graylog/server/server.conf
```
- Dùng lệnh dưới để sửa dòng password_secret. Lưu ý dùng chuỗi sinh ra trong bước hash mật khẩu ở trên  
```
pass_secret=$(pwgen -s 96)
sed -i -e 's|password_secret =|password_secret = '$pass_secret'|' /etc/graylog/server/server.conf
```
Sửa dòng  rest_listen_uri = http://127.0.0.1:9000/api/ thành giá trị mới là rest_listen_uri = http://192.168.152.134:9000/api/ . Trong đó IP 192.168.152.134 là địa chỉ của máy chủ cài đặt graylog. 
```
sed -i -e 's|rest_listen_uri = http://127.0.0.1:9000/api/|rest_listen_uri = http://192.168.152.134:9000/api/|' /etc/graylog/server/server.conf
``` 
- Sửa dòng #web_listen_uri = http://127.0.0.1:9000/ thành dòng web_listen_uri = http://192.168.152.134:9000/   
```
sed -i -e 's|#web_listen_uri = http://127.0.0.1:9000/|web_listen_uri = http://192.168.152.134:9000/|' /etc/graylog/server/server.conf 
```
- Sửa dòng #root_timezone = UTC thành dòng root_timezone = Asia/Ho_Chi_Minh  
```
sed -i 's|#root_timezone = UTC|root_timezone = Asia/Ho_Chi_Minh|' /etc/graylog/server/server.conf
```  

- Bước cuối cùng là khởi động và kích hoạt graylog  
```
chkconfig --add graylog-server
systemctl daemon-reload
systemctl enable graylog-server.service
systemctl start graylog-server.service
```  
Sau khi cài đặt xong, truy cập web với URL http://192.168.70.112:9000 và nhập tài khoản admin, mật khẩu ở bước phía trên.  



