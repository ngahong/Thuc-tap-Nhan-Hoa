### Hướng dẫn tạo 1 hard disk trên Linux  

**1. Cấu trúc LVM**  
`Logical Volume Management` được sử dụng để tạo nhiều ổ đĩa logic. Nó cho phép các bộ phận logic được thay đổi kích thước (giảm hoặc tăng) theo ý muốn của người quản trị.  

<img src="https://i.imgur.com/fNCFJRO.png">

Kiểm tra `Hard Drivers` có trên hệ thống bằng lệnh `lsblk`   

<img src="https://i.imgur.com/eUhBMre.png">

Ta thấy hệ thống đang có 1 hard disk nằm trong `/dev/sda` vậy nên ta sẽ tiếp tục add thêm 1 hard disk `/dev/sdb` nữa.  

Mở `Edit visual machine setting` ở cửa sổ VMWare. Chọn `Add` và click vào phần `Harddisk`.  

<img src="https://i.imgur.com/MTi0zij.png">  

Tiếp tục thiết lập thông số cho hard disk. Ở đây chọn dung lượng đĩa size = 10G. Sau đó chọn `Next` và `Finish`.  

<img src="https://i.imgur.com/mK6xhmO.png">

Sau đó ta kiểm tra lại `/sdb` đã được tạo chưa:

<img src="https://i.imgur.com/zttgS3i.png">  

**2. Phân vùng hard disk**  

Bây giờ chúng ta sẽ phân vùng cho hard disk `/sdb`. Để thao tác phân vùng đĩa, hãy mở nó và bắt đầu bằng lệnh dưới đây:   
```
# fdisk /dev/sdb
```
Tiếp theo ấn phím `m` để xem các tùy chọn.  

<img src="https://i.imgur.com/EdnlHLf.png">

Muốn phân vùng ta sử dụng lệnh `p`.   
Để tạo một phân vùng mới, hãy dùng lệnh `n` và sau đó chọn `p` cho `primary` và depending 1-4 vào phân vùng trên ổ đĩa này.  
Ta để các thông số cài đặt theo chế độ mặc định (default) ngoại trừ `Last sector` ta để `+1G`.  
- `n` (tạo một phân vùng mới)  
- `p` (tạo phân vùng chính)  
- `1` (số `1` biểu thị phân vùng sẽ là `/dev/sdb1`)  
Sau đó ta dùng lệnh `w` để lưu và thoát.

<img src="https://i.imgur.com/BWszzba.png">

**3. Định dạng lại partition thành LVM**  

 Tiếp tục gõ lệnh `fdisk /dev/sdb` và sử dụng lệnh `t` để thay đổi định dạng partition. Chọn `8e` để thay đổi thành `LVM` rồi chọn `w` để lưu.  

<img src="https://i.imgur.com/ejguw40.png">

Tương tự tạo các partition khác của `/sdb` là `sdb2`, `sdb3`.  
Và tạo thêm hard disk `/sdc` kèm theo các partition `sdc1`, `sdc2`.

<img src="https://i.imgur.com/qiUVPL4.png">  

**4. Tạo Physical Volume (PV)**  

Sử dụng lệnh `pvcreate` để tạo các Physical Volume.
```
# pvcreate /dev/sdb1
# pvcreate /dev/sdc1
```
Nếu bạn không thực hiện được lệnh `pvcreate` thì tiến hành cài đặt `yum install -y lvm2`.  

<img src="https://i.imgur.com/cJtwF5v.png">

Kiểm tra danh sách các Physical Volume được tạo bằng lệnh `pvs` hoặc `pvdisplay`.  

<img src="https://i.imgur.com/M3uJaFj.png">

**5. Tạo Volume Group**  
Tiếp theo ta sẽ nhóm các Physical Volume vào Volume Group. Cú pháp lệnh:  
```
# vgcreate vgcreate tên_group_volume Physical_volume1 Physical_volume2 Physical_volume3 ...
```
<img src="https://i.imgur.com/wDY1I6x.png">

Ta sử dụng lệnh `vgs` hoặc `vgdislay` để xem kết quả Volume Group vừa tạo. 

**6. Tạo Logical Volume**  
Từ 1 `Volume Group` ta có thể tạo ra các `Logical Volume` bằng cách sử dụng lệnh theo cú pháp:  
```
# lvcreate -L size_volume -n tên_logical_volume tên_group_volume
```
<img src="https://i.imgur.com/taXsOHU.png">  

Sử dụng lệnh `lvs` để kiểm tra kết quả.  

**7. Định dạng Logical Volume**  
Để format `Logical Volume` thành các định dạng như `ext2`, `ext3`, `ext4` ta thực hiện như sau:  
<img src="https://i.imgur.com/fHN8nif.png">  

**8. Mount và sử dụng**  

Ta tạo 1 thư mục và mount Logical Volume vào đó.

<img src="https://i.imgur.com/fQ7Kc9z.png">  









