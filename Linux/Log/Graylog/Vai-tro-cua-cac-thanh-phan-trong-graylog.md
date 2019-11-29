## Tìm hiểu các thành phần trong Graylog  

### Stream  
 Graylog Stream (luồng) là một cơ chế để định tuyến các message thành các danh mục trong thời gian thực trong khi chúng được xử lý. 

Luồng là một tính năng cốt lõi của Graylog và có thể được coi là một hình thức gắn thẻ cho các tin nhắn đến. Luồng là một cơ chế được sử dụng để định tuyến tin nhắn thành các danh mục trong thời gian thực. Quy tắc truyền phát hướng dẫn Graylog thông báo nào định tuyến vào luồng nào.

Luồng có nhiều công dụng. Đầu tiên, chúng được sử dụng để định tuyến dữ liệu để lưu trữ vào một chỉ mục. Chúng cũng được sử dụng để kiểm soát truy cập dữ liệu, định tuyến tin nhắn để phân tích cú pháp, làm giàu hoặc sửa đổi khác và xác định tin nhắn nào sẽ được lưu trữ.

Các luồng có thể được sử dụng cùng với Cảnh báo để thông báo cho người dùng hoặc trả lời khác khi tin nhắn đáp ứng một tập hợp các điều kiện.  

### Search  
`Graylog Search` là giao diện được sử dụng để tìm kiếm nhật ký trực tiếp. Graylog dử dụng cú pháp đơn giản hóa, rất giống với Lucene. Phạm vi thời gian tương đối hoặc tuyệt đối có thể được cấu hình từ menu thả xuống.   
Các tìm kiếm có thể được lưu hoặc hiển thị dưới dạng các tiện ích bảng điều khiển có thể được thêm trực tiếp vào bảng điều khiển từ trong màn hình tìm kiếm.

Người dùng có thể định cấu hình chế độ xem của riêng mình và có thể chọn xem dữ liệu tóm tắt hoặc hoàn chỉnh từ các thông báo sự kiện.  

### Dashboards (Bảng điều khiển)  
`Graylog Dashboards` là trực quan hóa hoặc tóm tắt thông tin có trong các sự kiện nhật ký. Mỗi bảng điều khiển được điền bởi một hoặc nhiều tiện ích, Widgets trực quan hóa hoặc tóm tắt dữ liệu nhật ký sự kiện với dữ liệu được lấy từ các giá trị trường như đếm, trung bình hoặc tổng. Các chỉ số xu hướng, biểu đồ, đồ thị và bản đồ dễ dàng được tạo để trực quan hóa dữ liệu.  
Các widget bảng điều khiển và bố trí bảng điều khiển có thể cấu hình. Truy cập bảng điều khiển được kiểm soát thông qua kiểm soát truy cập dựa trên vai trò của Graylog. Bảng điều khiển có thể được nhập và xuất qua gói nội dung.  

### Alert  
Cảnh báo có 2 thành phần đó là điều kiện cảnh báo và thông báo cảnh báo. Điều kiện cảnh báo được gắn với luồng và có thể dựa trên nội dung của một trường, giá trị tổng hợp của một trường hoặc ngưỡng đếm message.  
Một thông báo cảnh báo kích hoạt khi một điều kiện được đáp ứng, phổ biến là gửi một email hoặc gọi HTTP phân tích hoặc một hệ thống khác.  
Kiểu output truyền thống có thể được tạo bằng plusgins. Cảnh báo có thể được nhập và xuất bằng content packs.  

### System  

**Overview**  
