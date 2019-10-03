### Thực hành tạo VLAN trên Packet Tracer

#### Công việc:
Cho các PC từ 0-4
Thiết lập 2 VLAN tương ứng theo yêu cầu sau:

VLAN 90: PC0, PC2, PC4
VLAN 99: PC1, PC3

Sau khi cấu hình VLAN, thực hiện lệnh Ping để kiểm tra các kết nối.

#### Thực hiện:

1. Mở Cisco Packet Tracer và thực hiện thiết lập hệ thống gồm 1 switch và 5 PC.

*Lưu ý*: Dây đấu nối giữa switch và các PC là loại cable thẳng
<img src="https://i.imgur.com/GOn3hJP.png">

2. Tiến hành cấu hình địa chỉ IP cho từng PC
    PC0: 192.168.1.1/24

    PC1: 192.168.1.2/24

    PC2: 192.168.1.3/24

    PC3: 192.168.1.4/24

    PC4: 192.168.1.5/24

<img src="https://i.imgur.com/rbotX9S.png">

3.	Cấu hình VLAN trên switch
Mở cửa sổ CLI, thiết lập VLAN-id và VLAN-name  sau đó gán các cổng
<img src="https://i.imgur.com/bqN3ap5.png">

Sau khi đặt id và tên vlan thì tiếp tục gán các cổng

<img src="https://i.imgur.com/62KvoBd.png">

4.	Thực hiện lệnh Ping để kiểm tra các kết nối

<img src="https://i.imgur.com/oDfILMJ.png">

Do PC0 và PC3 ở cùng 1 VLAN nên có thể ping được với nhau còn PC2 nằm khác VLAN nên kết quả ping trả về là *Request timed out*.