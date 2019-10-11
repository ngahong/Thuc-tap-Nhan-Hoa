### Logical Volume Management(LVM)  
1. Tìm hiểu về LVM  
`Logical Volume Management` được sử dụng để tạo nhiều ổ đĩa logic. Nó cho phép các bộ phận logic được thay đổi kích thước (giảm hoặc tăng) theo ý muốn của người quản trị. 
Logical Volume Manager (LVM) là phương pháp cho phép ấn định không gian đĩa cứng (Physical Volume) thành những Logical Volume khiến cho việc thay đổi kích thước trở nên dễ dàng hơn. Với kỹ thuật LVM ta có thể thay đổi kích thước mà không phải sửa lại bảng table của OS. Điều này rất hữu ích khi ta sử dụng hết phần bộ nhớ còn trống của partition (phân vùng) và muốn mở rộng dung lượng của nó. `Physical Volume` là những đĩa vật lý như `sda`,`sdb`,... `Group Volume` là một nhóm tập hợp nhiều `Physica Volume`, dung lượng tổng được sử dụng để tạo ra các `Logical Volume`. `Logical Volume` có thể xem là các "phân vùng ảo" trên "ổ đĩa ảo" ta có thể thêm vào, gỡ bỏ và thay đổi kích thước một cách nhanh chóng.  
<img src="https://i.imgur.com/sotY3mn.png">  
**Ưu điểm**:  
- Có thể gom nhiều đĩa cứng vật lý thành 1 đĩa ảo có dung lượng lớn hơn.
- Có thể tạo ra các phân vùng có dung lượng lớn nhỏ tùy ý.  
- Có thể thay đổi kích thước các vùng dễ dàng và linh hoạt.  
**Nhược điểm**:  
- Các bước thiết lập phức tạp.  
- Càng gắn nhiều ổ đĩa và thiết lập LVM thì hệ thống khởi động càng lâu.  
- Có thể bị mất dữ liệu khi 1 trong các đĩa cứng vật lý bị hỏng.  
