## Hướng dẫn cài đặt MariaDB trên CentOS 8  

CentOS 8 sử dụng `dnf` để quản lý gói mặc định thay cho `yum` của các phiên bản trước đó. Trong bài viết này tôi sẽ tiến hành cài đặt `MariaDB 10.3` bằng lệnh `dnf module -y install mariadb:10.3`.   
```
[root@centos8srv01 ~]# dnf module -y install mariadb:10.3
```  
Mặc định charaset trong file cấu hình `mariadb-server.cnf` là `latin1`. Ta sẽ thiết lập lại bằng cách thêm mã `character-set-server=utf8` vào dòng 21 trong `/etc/my.cnf.d/mariadb-server.cnf`  
```
vi /etc/my.cnf.d/mariadb-server.cnf
character-set-server=utf8  
```  
Kích hoạt dịch vụ MariaDB tự chạy khi khởi động hệ thống  
```
systemctl enable --now mariadb
```
Nếu hệ thống đang chạy firewall, thêm dòng sau để cho phép chạy dịch vụ MariaDB 
```
[root@centos8srv01 ~]# firewall-cmd --add-service=mysql --permanent
success
[root@centos8srv01 ~]# firewall-cmd --reload
success
```
### Bảo mật MariaDB trên CentOS 8  

Sử dụng lệnh `mysql_secure_installation` để thiết lập mật khẩu root, vô hiệu hóa đăng nhập root từ xa, xóa database test, xóa anonymous user và cuối cùng là reload các table liên quan đến quyền hạn. 
```
[root@centos8srv01 ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
### Thao tác cơ bản với MariaDB  

**Đăng nhập MariaDB với tài khoản root**  
```
[root@centos8srv01 ~]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 16
Server version: 10.3.11-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```
**Hiển thị danh sách user**  
```
MariaDB [(none)]> select user,host,password from mysql.user;
+------+-----------+-------------------------------------------+
| user | host      | password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | *xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx |
| root | 127.0.0.1 | *xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx |
| root | ::1       | *xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx |
+------+-----------+-------------------------------------------+
3 rows in set (0.001 sec)
```
**Hiển thị danh sách cơ sở dữ liệu**  
```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.001 sec)
```

**Tạo cơ sở dữ liệu `test_database`**  
```
MariaDB [(none)]> create database test_database;
Query OK, 1 row affected (0.001 sec)
```

**Tạo bảng `test_table`**  
Trong cơ sở dữ liệu `test_database` tạo bảng `test_table` gồm các trường `id`, `name`, `address`, `primary key` tương ứng với các kiểu dữ liệu int, varchar(50), varchar(50), id.    
```
MariaDB [(none)]> create table test_database.test_table (id int, name varchar(50), address varchar(50), primary key (id));
Query OK, 0 rows affected (0.004 sec)
```
**Chèn thông tin các trường trong bảng**  
```
MariaDB [(none)]> insert into test_database.test_table(id, name, address) values("001", "CentOS", "Cloud365");
Query OK, 1 row affected (0.001 sec)
```
**Hiển thị thông tin của bảng**  
```
MariaDB [(none)]> select * from test_database.test_table;
+----+--------+----------+
| id | name   | address  |
+----+--------+----------+
|  1 | CentOS | Cloud365 |
+----+--------+----------+
1 row in set (0.001 sec)
```  
**Xóa cơ sở dữ liệu**  
```
MariaDB [(none)]> drop database test_database;   
Query OK, 1 row affected (0.111 sec)
```
**Thoát khỏi MariaDB**  
```
MariaDB [(none)]> exit
Bye
```  

