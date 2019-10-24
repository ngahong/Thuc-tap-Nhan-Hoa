### Tìm hiểu DHCP và cấu hình DHCP trên server  

### Chuẩn bị:  
Trên VMWare Workstation dựng 2 máy, 1 máy chủ server CentOS 7_1 và 1 máy client CentOS 7_2. Ta sẽ tiến hành cấu hình DHCP server trên máy chủ và máy client sẽ nhận địa chỉ IP động do máy chủ cấp phát.  

### Tiến hành:  

**Phía máy chủ server**  

1. Tắt chế độ cấp phát địa chỉ IP động trên VMWare Workstation. 

<img src="https://i.imgur.com/VZVn0ox.png">

2. Đặt địa chỉ IP tĩnh cho server   

*Cách 1*: Sửa file cấu hình trong `/etc/sysconfig/network-scripts/ifcfg-ens33`.   
Ở đây tôi đặt địa chỉ IP tĩnh nằm trong dải `192.168.152.0` gắn với NIC `ens33` trên máy chủ.   

```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="a1131155-80f0-4942-b712-130f300fbe1a"
DEVICE="ens33"
ONBOOT="yes"
IPADDR=192.168.152.3
GATEWAY=192.168.152.2
PREFIX=24
DNS=8.8.8.8
```  
*Cách 2*: sử dụng lệnh `nmcli` để cấu hình trên command line. 

3. Cài đặt dhcp trên máy chủ  

```
# yum install -y dhcp
```  
4. Cấu hình dhcp server bằng cách chỉnh sửa file `/etc/dhcp/dhcpd.conf`.  

```
option domain-name "mydomain.org";
option domain-name-servers ns1.mydomain.org, ns2.mydomain.org;
lease-file-name "/var/lib/dhcpd/dhcpd.leases";
default-lease-time 600;
max-lease-time 7200;

subnet 192.168.152.0 netmask 255.255.255.0 {
  range 192.168.152.10 192.168.152.100;
  option domain-name-servers ns1.internal.mydomain.org;
  option domain-name "internal.mydomain.org";
  option routers 192.168.152.1;
  option broadcast-address 192.168.152.255;
  default-lease-time 600;
  max-lease-time 7200;
}

```
Ở đây: 
- Cấu hình toàn cục (global) cho DHCP server
    - option domain-name: khai báo tên miền lớp mạng chung.
    - option domain-name-servers: khai báo tên server của domain bạn đã cung cấp ở trên.
    - lease-file-name: chỉ định file chứa thông tin IP đã được cấp phát qua DHCP.
    - default-lease-time: thời gian tồn tại mặc định của IP DHCP cấp phát cho người dùng.
    - max-lease-time: thời gian tồn tại tối đa mà địa chỉ IP DHCP cấp phát cho người dùng. 

- Cấu hình cục bộ (local) cho DHCP server  
    - range <ip_1> <ip_2>: dải ip mà máy chủ cấp phát động.  
    - option domain-name-servers: cung cấp thông tin DNS của server cho client.
    - option domain-name: tên miền của server
    - option routers: thông tin địa chỉ IP của router gateway mà client sẽ sử dụng khi nhận IP DHCP. 
    - option broadcast-address: thông tin broadcast của lớp mạng mà client sẽ nhận IP sử dụng.
    - default-lease-time:  
    - max-lease-time:  

Sau khi chỉnh sửa file cấu hình, cần khởi động lại bằng lệnh restart  
```
# systemctl restart dhcpd
```  
5. Lease Database  

Khi DHCP gán IP cho 1 client nào đó, DHCP Server sẽ ghi nhận lại các IP mà nó đã gán cho các host vào lease database file, ta có thể xem chi tiết tại:
```
# cat /var/lib/dhcpd/dhcpd.leases
```
*Lưu ý chung:*  
- Lease database sẽ được tái tạo lại thường xuyên để kích thước file không quá lớn.  
- Không tự ý xóa và tạo file mới. trước khi thay đổi nên tạo backup. 
- Nếu khởi động dịch vụ DHCP thất bại, nguyên nhân có thể do file dhcpd.lease không tồn tại. Tạo file này bằng lệnh sau:  
```
# touch /var/lib/dhcpd/dhcpd.lease
```  
- Trường hợp Server có nhiều network interfaces, một interface được cấu hình làm dhcp client nhận Public IP address để ra Internet, interface còn lại ta cấu hình (cho mạng nội bộ đứng sau firewall) nhận request từ local client bằng cách thêm dòng sau vào file `/etc/sysconfig/dhcpd`:
```
DHCPDARGS=interface_name
```
Ở đây tôi chọn gắn vào `ens33` trên máy của tôi. 

<img src="https://i.imgur.com/xCS3MkW.png">

6.  Khởi động DHCP server và set rule firewall.  

- Bây giờ ta sẽ khởi động dịch vụ dhcp trên server:  
```
# systemctl start dhcpd
```  
Nếu đang chạy Firewalld, hãy cho phép dịch vụ DHCP. DHCP Server sử dụng port 67/UDP
```
# firewall-cmd --add-service=dhcp --permanent 
success
# firewall-cmd --reload 
success
```

Kiểm tra xem tiến trình dịch vụ dhcpd có đang chạy trên hệ thống không bằng lệnh `ps`. 

<img src="https://i.imgur.com/0Ws7rrh.png">

**Phía máy client**  

Trước tiên bạn cần xác định chính xác card mạng nào đang nằm cùng lớp mạng thiết bị hay VLAN với server cấp IP DHCP. Giả sử tôi sử dụng card mạng ens33 trong máy client và chỉnh sửa trong file cấu hình. Đặt giá trị **BOOTPROTO="dhcp"** và khởi động lại dịch vụ mạng bằng lệnh `systemctl restart network.service` và xem kết quả. 

<img src="https://i.imgur.com/fPcCzJt.png">

Như vậy máy client đã nhận được địa chỉ IP do máy server cấp phát. Đối chiếu với IP của máy client là `192.168.152.11` đã nằm trong dải `192.168.152.10` - `192.168.152.100` của range IP bên phía server đặt.

**Mutilhomed DHCP**   
1. Cấu hình 1 host vào nhiều mạng 

Đây là trường hợp DHCP có 2 subnet nhưng chỉ có 1 host. Host này được cấu hình địa chỉ IP nằm trong miền IP của cả 2 subnet. Host mang IP của subnet nào tùy thuộc vào việc nó được kết nối vật lý tới mạng nào. Cấu hình này hữu ích trong trường hợp ta muốn thay đổi nhanh địa chỉ của 1 host về 1 subnet dự phòng. 

```
default-lease-time 600;
max-lease-time 7200;
subnet 10.0.0.0 netmask 255.255.255.0 { 
option subnet- mask 255.255.255.0
option routers 10.0.0.1;
range 10.0.0.5 10.0.0.15;
}
subnet 172.16.0.0 netmask 255.255.255.0 {
option subnet-mask 255.255.255.0
option routers 172.16.0.1;
range 172.16.0.5 172.16.0.15;

}
host example0 {
hardware ethernet 00: 1A: 6B: 6A: 2E: 0B;
fixed- address 10. 0. 0. 20;
}
host example1 {
hardware ethernet 00: 1A: 6B: 6A: 2E: 0B;
fixed- address 172. 16. 0. 20;
}
```
2. Cấu hình nhiều host vào 1 mạng  

Trong một trường hợp khác. Ta có 2 network interface khác nhau của 2 host và được gán cùng 1 địa chỉ IP. Lưu ý là cấu hình sau không hiệu quả nếu cả 2 host cùng kết nối tới cùng một network.  
Cấu hình này ứng dụng trong trường hợp 1 host bị down thì ta chỉ việc kết nối host dự phòng vào mạng là host dự phòng sẽ mang địa chỉ IP của host bị down và thay thế nó.

```
host interface0 {
hardware ethernet 00: 1a: 6b: 6a: 2e: 0b;
fixed- address 10. 0. 0.18;
}

host interface1 {
hardware ethernet 00: 1A: 6B: 6A: 27: 3A;
fixed- address 10. 0. 0.18;
}
```