## Graylog Sidecar  
Graylog Sidecar là hệ thống quản lý cấu hình nhẹ dành cho người thu thập log khác nhau, cũng được gọi là *Backends*. Node Graylog hoạt động như một trung tâm tập trung chứa các cấu hình của các trình thu thập log. Trên các thiết bị / máy chủ tạo messages được hỗ trợ, Sidecar có thể chạy dưới dạng dịch vụ (máy chủ Windows) hoặc daemon (máy chủ Linux). 

<img src="https://i.imgur.com/7iqyfve.png">  

Các cấu hình trình thu thập log được quản lý tập trung thông qua giao diện web Graylog. Theo định kỳ, trình nền Sidecar sẽ tìm nạp tất cả các cấu hình có liên quan cho mục tiêu, sử dụng API REST. Trong lần chạy đầu tiên hoặc khi phát hiện thay đổi cấu hình, Sidecar sẽ tạo (kết xuất) các tệp cấu hình phụ trợ có liên quan. Sau đó, nó sẽ bắt đầu hoặc khởi động lại, collectors log được cấu hình lại.  

## Configure  

Lưu ý cần chọn đúng gói cài đặt Sidecar theo phiên bản Graylog  

<img src="https://i.imgur.com/8jCyoSa.png">  

### Các bước cấu hình  

- Tải gói cài đặt sidecar  
```
wget https://github.com/Graylog2/collector-sidecar/releases/download/1.0.1/graylog-sidecar-1.0.1-1.x86_64.rpm
```
- Cài đặt gói `graylog-sidecar-1.0.1-1.x86_64.rpm`  
```
rpm -i graylog-sidecar-1.0.1-1.x86_64.rpm
graylog-sidecar -service install
systemctl enable graylog-sidecar
systemctl start graylog-sidecar
```
- Chỉnh sửa một vài thông số trong file cấu hình `/etc/graylog/sidecar/sidecar.yml`  
```
server_url: "http://192.168.152.134:9000/api/"
server_api_token: "15ef2382egig3cb3km1j0g014rcsbam8d6q9ro2o34e956m342h8"
```
Trong đó `192.168.152.134` là địa chỉ IP của graylog-server. Còn `15ef2382egig3cb3km1j0g014rcsbam8d6q9ro2o34e956m342h8` là api_token được tạo tự động khi ta thao tác trên trình duyệt graylog. Cách làm như sau:  
Bạn chọn `System/Sidecar` sau đó chọn `Create or reuse a token for the graylog-sidecar user`.

<img src="https://i.imgur.com/W6QRXn3.png">  

Cửa sổ `Authentication Management` hiện ra và bạn cần điền tên Token sau đó chọn `Create Token`. Tiếp theo copy chuỗi hash được sinh ra và ghi vào file cấu hình `sidecar.yml` bên phía máy client (Trong ví dụ của tôi là chuỗi `15ef2382egig3cb3km1j0g014rcsbam8d6q9ro2o34e956m342h8`). 

<img src="https://i.imgur.com/mafquV6.png">

- Tải `filebeat` về máy client và giải nén  
```
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.2-x86_64.rpm
rpm -i filebeat-7.4.2-x86_64.rpm
``` 

- Trên Graylog Web-server ta thiết lập Input log từ Client về.  

Chọn `Beats` và thiết lập các thông số. Port mặc định của Sidecar là 5044. Sau đó chọn `Save`

<img src="https://i.imgur.com/0FXApHk.png">

- Ta thực hiện cấu hình lần lượt cho các phần theo thứ tự  

<img src="https://i.imgur.com/v5rDsTv.png">  

Chọn tab `Edit` và chỉnh sửa trong mục `Default Template`. Điền địa chỉ IP của graylog-server  

<img src="https://i.imgur.com/w9Kf4vQ.png">  

Tiếp theo, chúng ta cần gán cấu hình mới được tạo (và đó là trình thu thập Filebeat) cho sidecar của chúng ta. Chuyển đến trang Quản trị Collector. 

<img src="https://i.imgur.com/4ci6k15.png">  

Sau khi thiết lập thành công, bạn có thể quay lại Overview Sidecars và nhấp vào nút `Show messages` để tìm kiếm nhật ký đã được thu thập thông qua sidecar của bạn.  
<img src="https://i.imgur.com/CWgedU4.png">  
