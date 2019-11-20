## Cấu hình TLS   

### 1. Đặt vấn đề

Ở những phần trước ta đã tìm hiểu cơ bản về log trong Linux. Tuy nhiên khi vận chuyển gói tin từ client đến server qua TCP hoặc UDP thì các message không được mã hóa, do đó, bất kỳ hacker nào đánh hơi các gói tin của bạn đều có thể xem được các log, theo dõi nội dung, tìm ra lỗ hổng và thực hiện các cuộc tấn công vào máy chủ của bạn.    
Ví dụ ta dùng Wireshark để xem các gói tin và IP rất cụ thể.  

<img src="https://i.imgur.com/XiMXbIt.png"> 

Quan sát ảnh trên bạn sẽ thấy nội dung một message rất rõ ràng "Hi this is a test message". Thử tượng tượng nếu tin nhắn chứa thông tin nhạy cảm từ máy chủ của bạn bị lộ ra ngoài thì nó có thể trở thành một thảm họa an ninh.  
Đó là lý do tại sao chúng ta phải thiết tập `TLS`, cho phép mã hóa bản tin truyền đi giữa máy khách và máy chủ.  

### 2. Mã hóa Rsyslog messages với TLS  

Để mã hóa messages giữa client và server ta sẽ sử dụng TLS và chứng chỉ tin cậy (trusted certificates).  
Cần đảm bảo rằng máy client và server đang giao tiếp  chính xác với nhau sau đó sẽ tiến hành mã hóa messages của chúng.  
Giống như sự trao đổi keys giữa client và server, chúng có thể giải mã messsage một cách nhanh chóng.  
Trong phần cài đặt, ta sẽ sử dụng một chứng chỉ chứng thực chữ ký khóa (**a certificate authority signing the keys**).  
Giao tiếp giữa các host lưu trữ và cơ quan cấp chứng chỉ cũng được mã hóa rõ ràng, vì vậy trước tiên chúng ta cần tạo cặp khóa (key-pair) cho cơ quan cấp chứng chỉ (Certificate Authority - CA).  

<img src="https://i.imgur.com/sB3xAuK.png">

Nếu bạn chỉ đang làm trên mô hình thử nghiệm, bạn có thể sử dụng Rsyslog server như một `Certificate Authority`.  

Mô hình lab  
- Dựng 2 máy ảo CentOS7, 1 máy đóng vài trò là client và máy còn lại là server.  

### Tạo chứng chỉ CA  
- Trên server cài các gói cần thiết  
```
yum install -y gnutls-utils
(or)
yum install -y gnutls-bin
```  
- Tạo private key cho CA
```
certtool --generate-privkey --outfile CA-key.pem
```
- Tạo public key cho CA
```
certtool --generate-self-signed --load-privkey CA-key.pem --outfile CA.pem

Common name: ngahd
The certificate will expire in (days): 3650
Does the certificate belong to an authority? (y/N): y
Will the certificate be used to sign other certificates? (y/N): y
Is the above information ok? (Y/N): y
```  
> Ta chỉ cần trả lời một số câu hỏi như trên, các câu hỏi khác có thể enter để bỏ trống.  

### Tạo private/public key cho server  

```
certtool --generate-privkey --outfile server-key.pem --bits 2048
```  
```
certtool --generate-request --load-privkey server-key.pem --outfile server-request.pem  

Common name: server.com.vn
Enter a dnsName of the subject of the certificate: server.com.vn
Does the certificate belong to an authority? (y/N): n
Is this a TLS web client certificate? (y/N): y
Is this a TLS web server certificate? (y/N): y
```  

```
certtool --generate-certificate --load-request server-request.pem --outfile server-cert.pem --load-ca-certificate CA.pem --load-ca-privkey CA-key.pem
```
```
The certificate will expire in (days): 3650
Does the certificate belong to an authority? (y/N): n
Is this a TLS web client certificate? (y/N): y
Is this also a TLS web server certificate? (y/N): y
Enter a dnsName of the subject of the certificate: server.com.vn
Is the above information ok? (Y/N): y
```
- Xóa file `server-request.pem`.  
```
rm -rf server-request.pem
```  

### Tạo private/public key cho client
```
certtool --generate-privkey --outfile client-key.pem --bits 2048
```
```
certtool --generate-request --load-privkey client-key.pem --outfile client-request.pem
```

```
Common name: client.com.vn
Enter a dnsName of the subject of the certificate: client.com.vn
Does the certificate belong to an authority? (y/N): n
Is this a TLS web client certificate? (y/N): y
Is this a TLS web server certificate? (y/N): y
```
```
certtool --generate-certificate --load-request client-request.pem --outfile client-cert.pem --load-ca-certificate CA.pem --load-ca-privkey CA-key.pem
```
