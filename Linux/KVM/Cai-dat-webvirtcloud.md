# Cài đặt WebvirtCloud trên CentOS 7  

Mô hình  
<img src="https://i.imgur.com/JUV1Os4.png"> 

IP planning  

<img src="https://i.imgur.com/63IBWWl.png">  


## 1.Cài đặt WebvirtCloud trên CentOS 7  

### 1.1 SSH đến node WebVirtCloud
```
ssh root@192.168.95.10  
```
### 1.2 Cài đặt các packages cần thiết  
```
yum install epel-release -y
yum -y install python-virtualenv python-devel libvirt-devel glibc gcc nginx supervisor python-lxml git python-libguestfs
```  
### 1.3 Tạo thư mục và clone source code từ trang chủ về  
```
cd /srv
git clone https://github.com/retspen/webvirtcloud && cd webvirtcloud
cp webvirtcloud/settings.py.template webvirtcloud/settings.py
```
### 1.4 Thay thế secret key  
```
[root@webvirtcloud webvirtcloud]# pwd
/srv/webvirtcloud
```
Thay đổi chuỗi secret key trong file `settings.py` bằng một đoạn string ngẫu nhiên mà chỉ mỗi bạn sở hữu
```
vi webvirtcloud/settings.py
```  
Như sau  
```
SECRET_KEY = 'hongnga'
```  
### 1.5 Cài đặt webvirtcloud  
```
[root@webvirtcloud webvirtcloud]# pwd
/srv/webvirtcloud
```  
```
virtualenv venv
source venv/bin/activate
venv/bin/pip install -r conf/requirements.txt
cp conf/nginx/webvirtcloud.conf /etc/nginx/conf.d/
venv/bin/python manage.py migrate
```  
### 1.6 Cấu hình supervisor  
Thêm các cấu hình sau vào cuối file `/etc/supervisord.conf`  
```
cp /etc/supervisord.conf /etc/supervisord.conf.bk
vi /etc/supervisord.conf
```
Như sau  
```
[program:webvirtcloud]
command=/srv/webvirtcloud/venv/bin/gunicorn webvirtcloud.wsgi:application -c /srv/webvirtcloud/gunicorn.conf.py
directory=/srv/webvirtcloud
user=nginx
autostart=true
autorestart=true
redirect_stderr=true

[program:novncd]
command=/srv/webvirtcloud/venv/bin/python /srv/webvirtcloud/console/novncd
directory=/srv/webvirtcloud
user=nginx
autostart=true
autorestart=true
redirect_stderr=true  
```  
### 1.7 Cấu hình nginx  

Comment lại block server trong file `/etc/nginx/nginx.conf`  
```
vi /etc/nginx/nginx.conf 
```  
Như sau  
```
#    server {
#        listen       80 default_server;
#        listen       [::]:80 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }
```  
Sau đó chỉnh sửa file `/etc/nginx/conf.d/webvirtcloud.conf`  
```
vi /etc/nginx/conf.d/webvirtcloud.conf  
```
Như sau  
```
upstream gunicorn_server {
    #server unix:/srv/webvirtcloud/venv/wvcloud.socket fail_timeout=0;
    server 127.0.0.1:8000 fail_timeout=0;
}
server {
    listen 80;

    server_name $hostname;
    access_log /var/log/nginx/webvirtcloud-access_log; 

    location /static/ {
        root /srv/webvirtcloud;
        expires max;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Proto $remote_addr;
        proxy_connect_timeout 600;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
        client_max_body_size 1024M;
    }
}
```
Restart supervisord  
```
systemctl restart supervisord
```  
### 1.8 Phần quyền cho các thư mục  
Phân quyền cho user nginx có thể đọc được file trong thư mục chứa code
```
chown -R nginx:nginx /srv/webvirtcloud
```  
Phần quyền cho selinux  
```
yum install policycoreutils-python -y
setenforce 0
semanage fcontext -a -t httpd_sys_content_t "/srv/webvirtcloud(/.*)"  
```  
### 1.9 Cấu hình firewalld  
```
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=6080/tcp
firewall-cmd --reload
```  
### 1.10 Restart và Enable services  
```
systemctl restart nginx && systemctl restart supervisord
systemctl enable nginx && systemctl enable supervisord
```  

## 2. Cấu hình trên host KVM  
Để WebvirtCloud có thể kết nối đến Host KVM và quản lý được các VM trong host KVM ta cần cấu hình một số thông tin sau trên host KVM

Trên môi trường lab nên tôi cũng tiến hành tắt firewalld. 
```
systemctl stop firewalld  
```  

Hoặc nếu không muốn tắt firewall bạn có thể mở port `16509` để WebvirtCloud có thể kết nối đến.  

### 2.1 Cài đặt gói `libvirt`  
```
yum install libvirt  
```
### 2.2 Chỉnh sửa cấu hình libvirt  

- Vào file `/etc/libvirt/libvirtd.conf` sửa các dòng thành nội dung như sau  
```
listen_tls = 0
listen_tcp = 1
tcp_port = "16509"
listen_addr = "0.0.0.0"
auth_tcp = "none"  
```  
- Sau đó chỉnh sửa trên file `/etc/sysconfig/libvirtd`  
```
LIBVIRTD_ARGS="--listen"  
```  
- Kiểm tra lại cài đặt  
```
systemctl restart libvirtd  
ps ax | grep libvirtd  
ss -antup | grep libvirtd  
virsh -c qemu+tcp://127.0.0.1/system  
```  

## 3. Truy cập Web và add các node KVM  
- Truy cập đường dẫn  
```
http://192.168.95.10 
```  
- Login với user mặc định `admin/admin`. Sau đó add nodes  

<img src="https://i.imgur.com/ElQTUaO.png">  

- Click `Computes` và tích vào dấu "+" để add   

<img src="https://i.imgur.com/4L8Nobh.png">  

- Khai báo thông tin TCP connect tới `libvirt`  

<img src="">  




