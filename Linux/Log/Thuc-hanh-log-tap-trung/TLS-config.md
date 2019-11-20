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
Giao tiếp giữa các host lưu trữ và cơ quan cấp chứng chỉ cũng được mã hóa rõ ràng, vì vậy trước tiên chúng ta cần tạo cặp khóa (key-pair) cho cơ quan cấp chứng chỉ (Certificate Authority).  

<img src="https://i.imgur.com/sB3xAuK.png">

Nếu bạn chỉ đang làm trên mô hình thử nghiệm, bạn có thể sử dụng Rsyslog server như một `Certificate Authority`.  

> Lưu ý ta cần kiểm tra thời gian của client và server. Chúng cần được đồng bộ hóa.  

```
[root@server ~]# date
Wed Nov 20 11:17:06 +07 2019
```
```
[root@client ~]# date
Wed Nov 20 11:17:11 +07 2019
```
**2.1 Cấu hình Certificate Authority**  

Để tạo `self-signed certificate` ta sẽ sử dụng `certool`, là một phần của GnuTLS. Vì vậy ta cần cài đặt GnuTLS rpm.    
- Cài đặt `gnutls`
```
[root@client ~]# yum -y install gnutls-utils
``` 
> Lưu ý: Cổng TCP 6514 cần có thể truy cập được trên log server và client cũng cần có khả năng đi ra khỏi cổng đó.  

- Tạo `private key`  
```
[root@client ~]# certtool --generate-privkey --outfile ca-key.pem
Generating a 2048 bit RSA private key...
```
- Kiểm tra key vừa tạo  

<img src="https://i.imgur.com/MYd1Z5v.png">  

- Thay đổi quyền chỉ user root có thể đọc cho file  
```
[root@client ~]# chmod 400 ca-key.pem
```  
Bây giờ ta sẽ tạo (self-signed) CA Certificate (Chứng chỉ CA). Bạn cần phải điền thông tin và lựa chọn một vài thông số. Lưu ý bạn nên thiết lập hiệu lực của Certificate lâu dài. Ở đây tôi chọn 3650 ngày (10 năm). Bạn cần xác định rằng các chứng chỉ thuộc về một cơ quan có thẩm quyền. Giấy chứng nhận được sử dụng để ký các chứng chỉ khác.  
```
[root@client ~]# certtool --generate-privkey --outfile ca-key.pem
Generating a 2048 bit RSA private key...
[root@client ~]# ls -l
total 12
-rw-------. 1 root root 1412 Nov 19 09:25 anaconda-ks.cfg
-rw-------. 1 root root 5823 Nov 20 11:25 ca-key.pem
[root@client ~]#
[root@client ~]#
[root@client ~]# chmod 400 ca-key.pem
[root@client ~]# certtool --generate-self-signed --load-privkey ca-key.pem --outfile ca.pem
Generating a self signed certificate...
Please enter the details of the certificate's distinguished name. Just press enter to ignore a field.
Common name: client.example.com
UID:
Organizational unit name:
Organization name:
Locality name:
State or province name:
Country name (2 chars):
Enter the subject's domain component (DC):
This field should not be used in new certificates.
E-mail:
Enter the certificate's serial number in decimal (default: 6761243706684884463):


Activation/Expiration time.
The certificate will expire in (days): 3650


Extensions.
Does the certificate belong to an authority? (y/N): y
Path length constraint (decimal, -1 for no constraint): -1
Is this a TLS web client certificate? (y/N): n
Will the certificate be used for IPsec IKE operations? (y/N): n
Is this a TLS web server certificate? (y/N): n
Enter a dnsName of the subject of the certificate: client.example.com
Enter a dnsName of the subject of the certificate:
Enter a URI of the subject of the certificate:
Enter the IP address of the subject of the certificate:
Enter the e-mail of the subject of the certificate:
Will the certificate be used to sign OCSP requests? (y/N): n
Will the certificate be used to sign code? (y/N): n
Will the certificate be used for time stamping? (y/N): n
Will the certificate be used to sign other certificates? (y/N): y
Will the certificate be used to sign CRLs? (y/N): y
Enter the URI of the CRL distribution point:
X.509 Certificate Information:
        Version: 3
        Serial Number (hex): 5dd4c35420cc45ef
        Validity:
                Not Before: Wed Nov 20 04:38:46 UTC 2019
                Not After: Sat Nov 17 04:39:28 UTC 2029
        Subject: CN=client.example.com
        Subject Public Key Algorithm: RSA
        Algorithm Security Level: Medium (2048 bits)
                Modulus (bits 2048):
                        00:d2:40:42:a9:5a:57:32:7d:c4:8d:50:20:d0:91:e4
                        92:fa:f4:97:5a:09:7c:b3:a1:88:fb:7d:8f:ef:aa:5a
                        fe:91:b5:f5:42:ab:7c:a7:fd:f5:e1:38:22:46:1c:05
                        1e:96:01:38:79:a8:05:69:54:e3:e2:e5:ee:77:bd:52
                        bd:6e:fe:bd:59:ec:35:2e:86:de:ba:75:f5:52:eb:af
                        5f:36:78:4c:7b:28:6a:15:3e:d1:53:ba:5c:a9:f3:f8
                        90:f0:90:52:2b:31:7d:b7:fc:21:b1:d1:d6:a5:ec:d4
                        5a:12:da:77:01:97:6d:90:55:61:ad:64:f4:b6:5b:4e
                        80:d3:43:21:ba:c1:cc:85:0d:ca:c3:63:66:5e:db:be
                        fb:a2:54:b9:a4:ad:6d:84:68:0d:b6:67:52:fc:32:41
                        1f:53:b4:0e:cd:29:c7:74:d8:00:e7:56:29:bd:2d:ff
                        a3:30:49:1d:78:5c:33:8c:d0:3f:1b:71:4d:51:7d:b5
                        e5:49:b6:e1:90:56:74:c4:ea:2a:46:4c:fe:94:0a:cd
                        24:bf:87:fa:7c:cb:e3:c6:a6:00:e0:de:2a:aa:bd:de
                        49:1d:10:c8:e2:34:f5:40:22:3a:eb:47:39:b5:25:66
                        c0:06:38:cd:40:6e:14:28:d2:c2:17:72:66:2f:f8:5e
                        6b
                Exponent (bits 24):
                        01:00:01
        Extensions:
                Basic Constraints (critical):
                        Certificate Authority (CA): TRUE
                Subject Alternative Name (not critical):
                        DNSname: client.example.com
                Key Usage (critical):
                        Certificate signing.
                        CRL signing.
                Subject Key Identifier (not critical):
                        2e25978013ea54e3db296b3bca3b1dae1152d56c
Other Information:
        Public Key ID:
                2e25978013ea54e3db296b3bca3b1dae1152d56c
        Public key's random art:
                +--[ RSA 2048]----+
                |    =o           |
                |   = +E          |
                |  + +..          |
                | +   + o .       |
                |. o o + S        |
                | . ..o =         |
                |  .oo.. .        |
                | ..o+. .         |
                |  ==..           |
                +-----------------+

Is the above information ok? (y/N): y


Signing certificate...
```  

- Xác thực khóa vừa tạo  

<img src="https://i.imgur.com/oRyIJZb.png">  

**Lưu ý**:  
-  Không để bên thứ 3 biết CA của bạn. Nếu bị lộ, bảo mật sẽ thất bại.  
- `ca-key.pem` là private key để dành riêng cho CA certificate và `ca.pem` là public key có thể gửi đến các node khác. Bạn cũng có thể thu hồi chứng chỉ bằng cách sử dụng `openssl`. Tham khảo tại [đây](https://www.golinuxcloud.com/revoke-certificate-generate-crl-openssl/).  

**2.2 Tạo certificate cho các máy**  
Trong bước này, ta sẽ tạo certificate cho mỗi máy. Xin lưu ý rằng cả client và server đều cần chứng chỉ. Chứng chỉ xác định mỗi máy để ngang hàng từ xa.  
- Tiếp theo ánh xạ tên máy server để việc xác định khóa và tên node được dễ dàng hơn.  


