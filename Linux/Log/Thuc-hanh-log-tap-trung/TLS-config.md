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

Mô hình lab  
- Dựng 2 máy ảo CentOS7, máy `node2` đóng vài trò là client và `node3` là server.  


> Lưu ý ta cần kiểm tra thời gian của client và server. Chúng cần được đồng bộ hóa.  

```
[root@node2 ~]# date
Wed Nov 20 11:17:06 +07 2019
```
```
[root@node3 ~]# date
Wed Nov 20 11:17:11 +07 2019
```
**2.1 Cấu hình Certificate Authority**  

Để tạo `self-signed certificate` ta sẽ sử dụng `certool`, là một phần của GnuTLS. Vì vậy ta cần cài đặt GnuTLS rpm.    
- Cài đặt `gnutls`
```
[root@node2 ~]# yum -y install gnutls-utils
``` 
> Lưu ý: Cổng TCP 6514 cần có thể truy cập được trên log server và client cũng cần có khả năng đi ra khỏi cổng đó.  
```
[root@node2 ~]# firewall-cmd --zone=public --add-port=6514/tcp --permanent
success
[root@node2 ~]# firewall-cmd --reload
```
- Tạo `private key`  
```
[root@node2 ~]# certtool --generate-privkey --outfile ca-key.pem
Generating a 2048 bit RSA private key...
```
- Kiểm tra key vừa tạo  
```
[root@node2 ~]# ls -l
total 16
-rw-------. 1 root root 1412 Nov 19 09:25 anaconda-ks.cfg
-r--------. 1 root root 5813 Nov 20 14:32 ca-key.pem
```
- Thay đổi quyền chỉ user root có thể đọc cho file  
```
[root@node2 ~]# chmod 400 ca-key.pem
``` 

Bây giờ ta sẽ tạo (self-signed) CA Certificate (Chứng chỉ CA). Bạn cần phải điền thông tin và lựa chọn một vài thông số. Lưu ý bạn nên thiết lập hiệu lực của Certificate lâu dài. Ở đây tôi chọn 3650 ngày (10 năm). Bạn cần xác định rằng các chứng chỉ thuộc về một cơ quan có thẩm quyền. Giấy chứng nhận được sử dụng để ký các chứng chỉ khác.  
```
[root@node2 ~]# certtool --generate-privkey --outfile ca-key.pem
Generating a 2048 bit RSA private key...
[root@node2 ~]# ls -l
total 12
-rw-------. 1 root root 1412 Nov 19 09:25 anaconda-ks.cfg
-rw-------. 1 root root 5813 Nov 20 14:32 ca-key.pem
[root@node2 ~]# chmod 400 ca-key.pem
[root@node2 ~]# certtool --generate-self-signed --load-privkey ca-key.pem --outfile ca.pem
Generating a self signed certificate...
Please enter the details of the certificate's distinguished name. Just press enter to ignore a field.
Common name: node2.example.com
UID:
Organizational unit name:
Organization name:
Locality name:
State or province name:
Country name (2 chars):
Enter the subject's domain component (DC):
This field should not be used in new certificates.
E-mail:
Enter the certificate's serial number in decimal (default: 6761289147011052346):


Activation/Expiration time.
The certificate will expire in (days): 3650


Extensions.
Does the certificate belong to an authority? (y/N): y
Path length constraint (decimal, -1 for no constraint): -1
Is this a TLS web client certificate? (y/N): n
Will the certificate be used for IPsec IKE operations? (y/N): n
Is this a TLS web server certificate? (y/N): n
Enter a dnsName of the subject of the certificate: node2.example.com
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
        Serial Number (hex): 5dd4eca8074c333a
        Validity:
                Not Before: Wed Nov 20 07:35:08 UTC 2019
                Not After: Sat Nov 17 07:35:16 UTC 2029
        Subject: CN=node2.example.com
        Subject Public Key Algorithm: RSA
        Algorithm Security Level: Medium (2048 bits)
                Modulus (bits 2048):
                        00:a6:59:e7:49:da:de:1e:fe:d7:4b:33:31:f6:c0:af
                        ee:19:63:e2:bf:f1:d3:49:0a:0b:4d:0c:1e:23:3a:41
                        bb:37:bd:a5:8a:16:2c:eb:4a:9d:57:39:4d:e6:03:51
                        72:7d:99:64:9d:98:4b:18:47:de:c0:d5:de:5a:35:8a
                        97:a6:a6:34:42:ab:16:02:42:b7:40:cd:31:03:5f:1e
                        2c:a4:f4:e6:0f:36:b4:1f:2c:2e:60:76:82:69:f3:46
                        cd:e8:7b:82:be:ef:9b:9e:0d:5e:75:28:0f:86:e6:0c
                        a4:ff:3b:56:88:82:0d:d2:e6:8a:18:d2:43:a1:37:e8
                        23:43:b3:08:8f:30:19:7a:88:cd:72:41:8c:02:96:40
                        69:e4:23:38:80:5d:1f:30:50:80:9f:56:89:d4:cb:ed
                        79:48:3e:05:c6:44:da:65:b6:42:d8:e3:97:6b:1e:36
                        7b:3b:1b:4d:a9:f1:7c:78:b4:93:b8:41:ca:17:a8:d2
                        73:84:cf:b1:32:a5:ef:93:69:f5:2d:d9:1f:ad:a2:2b
                        58:42:db:ed:d8:61:1f:ad:25:ef:76:08:6b:48:8e:6b
                        6d:d4:89:2d:0d:3d:68:b8:66:7a:bb:11:34:c7:61:e6
                        f3:4c:db:00:e1:85:b2:e5:a4:68:d7:ba:7e:f3:b7:ff
                        87
                Exponent (bits 24):
                        01:00:01
        Extensions:
                Basic Constraints (critical):
                        Certificate Authority (CA): TRUE
                Subject Alternative Name (not critical):
                        DNSname: node2.example.com
                Key Usage (critical):
                        Certificate signing.
                        CRL signing.
                Subject Key Identifier (not critical):
                        35179cb6bf1238dd2b3eb6873f8714bcc716de14
Other Information:
        Public Key ID:
                35179cb6bf1238dd2b3eb6873f8714bcc716de14
        Public key's random art:
                +--[ RSA 2048]----+
                |           ...   |
                |            +. E |
                |          o..o  .|
                |         . o. o..|
                |        S  o o.=o|
                |          o o =.=|
                |           . + * |
                |            * * .|
                |           ooB.o |
                +-----------------+

Is the above information ok? (y/N): y


Signing certificate...
```  

- Xác thực khóa vừa tạo  
```
[root@node2 ~]# ls -l
total 16
-rw-------. 1 root root 1412 Nov 19 09:25 anaconda-ks.cfg
-r--------. 1 root root 5813 Nov 20 14:32 ca-key.pem
-rw-r--r--. 1 root root 1139 Nov 20 14:37 ca.pem
```

**Lưu ý**:  
-  Không để bên thứ 3 biết CA của bạn. Nếu bị lộ, bảo mật sẽ thất bại.  
- `ca-key.pem` là private key để dành riêng cho CA certificate và `ca.pem` là public key có thể gửi đến các node khác. Bạn cũng có thể thu hồi chứng chỉ bằng cách sử dụng `openssl`. Tham khảo tại [đây](https://www.golinuxcloud.com/revoke-certificate-generate-crl-openssl/).  

**2.2 Tạo certificate cho các máy**  
Trong bước này, ta sẽ tạo certificate cho mỗi máy. Xin lưu ý rằng cả client và server đều cần chứng chỉ. Chứng chỉ xác định mỗi máy để ngang hàng từ xa.  
- Tiếp theo ánh xạ tên máy server để việc xác định khóa và tên node được dễ dàng hơn.  
```
[root@node2 ~]# certtool --generate-privkey --outfile node3-key.pem --bits 2048
** Note: Please use the --sec-param instead of --bits
Generating a 2048 bit RSA private key...
```  
> Thực tế private key là chưa đủ nên cần phải ký từ certificate authority. Ở đây, ta đang đưa ra một yêu cầu bằng certtool để tải khóa riêng của `node3-key.pem` và ký khóa riêng này vào `outfile`, tức là `node3-request.pem`.   
- Tiếp tục điền các thông tin   
```
[root@node2 ~]# certtool --generate-request --load-privkey node3-key.pem --outfile node3-request.pem
Generating a PKCS #10 certificate request...
Common name: node3.example.com
Organizational unit name:
Organization name:
Locality name:
State or province name:
Country name (2 chars):
Enter the subject's domain component (DC):
UID:
Enter a dnsName of the subject of the certificate: node3.example.com
Enter a dnsName of the subject of the certificate:
Enter a URI of the subject of the certificate:
Enter the IP address of the subject of the certificate:
Enter the e-mail of the subject of the certificate:
Enter a challenge password:
Does the certificate belong to an authority? (y/N):
Will the certificate be used for signing (DHE and RSA-EXPORT ciphersuites)? (Y/n): n
Will the certificate be used for encryption (RSA ciphersuites)? (Y/n): n
Will the certificate be used to sign code? (y/N): n
Will the certificate be used for time stamping? (y/N): n
Will the certificate be used for IPsec IKE operations? (y/N): n
Will the certificate be used to sign OCSP requests? (y/N): n
Is this a TLS web client certificate? (y/N): n
Is this a TLS web server certificate? (y/N): n
```  
- Kiểm tra `server-request.pem` đã tồn tại chưa  
```
[root@node2 ~]# ls -l
total 28
-rw-------. 1 root root 1412 Nov 19 09:25 anaconda-ks.cfg
-r--------. 1 root root 5813 Nov 20 14:32 ca-key.pem
-rw-r--r--. 1 root root 1139 Nov 20 14:37 ca.pem
-rw-------. 1 root root 5823 Nov 20 14:37 node3-key.pem
-rw-------. 1 root root 2388 Nov 20 14:40 node3-request.pem
```
- Và sau khi thực hiện tất cả điều này, quy trình tạo tài liệu chính cho máy chủ nhật ký RSA, cũng như máy khách, sẽ hoàn tất. Ở đây, private key của cơ quan cấp chứng chỉ được sử dụng để ký các chứng chỉ sẽ được sử dụng bởi `node3` và đó là điều sẽ đảm bảo rằng `node3` sẽ được mọi người tham gia tin cậy.Tiếp tục thiết lập các thông số.  

```
[root@node2 ~]# certtool --generate-certificate --load-request node3-request.pem --outfile node3-cert.pem --load-ca-certificate ca.pem --load-ca-privkey ca-key.pem
Generating a signed certificate...
Enter the certificate's serial number in decimal (default: 6761290852408902406):


Activation/Expiration time.
The certificate will expire in (days): 1000


Extensions.
Do you want to honour the extensions from the request? (y/N):
Does the certificate belong to an authority? (y/N):
Is this a TLS web client certificate? (y/N): y
Will the certificate be used for IPsec IKE operations? (y/N):
Is this a TLS web server certificate? (y/N): y
Enter a dnsName of the subject of the certificate: node3.example.com
Enter a dnsName of the subject of the certificate:
Enter a URI of the subject of the certificate:
Enter the IP address of the subject of the certificate:
Will the certificate be used for signing (DHE and RSA-EXPORT ciphersuites)? (Y/n): n
Will the certificate be used for encryption (RSA ciphersuites)? (Y/n): n
Will the certificate be used to sign OCSP requests? (y/N): n
Will the certificate be used to sign code? (y/N): n
Will the certificate be used for time stamping? (y/N): n
X.509 Certificate Information:
        Version: 3
        Serial Number (hex): 5dd4ee3518ee4306
        Validity:
                Not Before: Wed Nov 20 07:41:54 UTC 2019
                Not After: Tue Aug 16 07:41:56 UTC 2022
        Subject: CN=node3.example.com
        Subject Public Key Algorithm: RSA
        Algorithm Security Level: Medium (2048 bits)
                Modulus (bits 2048):
                        00:d7:d6:11:6b:97:71:d1:55:b7:ab:1a:71:0f:08:ef
                        8a:3b:0f:c3:8d:a9:c1:2e:ec:b1:4f:4a:d4:d9:60:75
                        74:43:fd:c4:8b:13:e0:25:b6:d6:b0:2e:15:a6:b6:d9
                        81:c4:4c:58:c1:c2:ab:6d:ba:1a:3f:68:6b:c4:ab:c1
                        7c:48:b6:2e:28:33:57:cf:5b:a7:cd:4e:c5:bc:0b:9a
                        aa:d5:50:4d:8c:44:41:3b:17:1b:92:93:0d:be:3c:dc
                        b4:cb:44:d0:d0:92:1d:14:e8:31:35:69:16:2a:45:77
                        9d:a9:36:07:c8:67:5c:9f:48:d3:8e:71:3b:de:95:4e
                        83:24:c1:92:d8:a3:e0:08:84:31:bc:72:81:70:24:22
                        97:2d:a8:88:9c:58:c4:a0:8a:32:07:13:c2:f8:5f:d2
                        f9:8e:65:5c:59:e7:4c:25:5f:8d:65:0a:04:5d:79:c0
                        16:04:0a:e0:59:29:00:8f:d0:bb:de:c2:90:7e:94:c9
                        18:f4:ca:56:42:b2:38:a8:d0:7a:6a:49:95:4e:75:53
                        a6:0c:ba:ee:42:ab:7c:31:32:7d:6a:95:03:fd:f8:73
                        a7:7d:1e:40:dc:08:88:e6:29:ea:b3:e9:6f:1f:c6:85
                        2e:ee:8f:18:a5:f6:11:c6:8b:25:bb:57:09:5a:b7:87
                        67
                Exponent (bits 24):
                        01:00:01
        Extensions:
                Basic Constraints (critical):
                        Certificate Authority (CA): FALSE
                Key Purpose (not critical):
                        TLS WWW Client.
                        TLS WWW Server.
                Subject Alternative Name (not critical):
                        DNSname: node3.example.com
                Subject Key Identifier (not critical):
                        f7acadf85079a38280f7f7e89e534464afdf4f29
                Authority Key Identifier (not critical):
                        35179cb6bf1238dd2b3eb6873f8714bcc716de14
Other Information:
        Public Key ID:
                f7acadf85079a38280f7f7e89e534464afdf4f29
        Public key's random art:
                +--[ RSA 2048]----+
                |         .o      |
                |         ...     |
                |         .  .    |
                |   .      .o     |
                |  . o   S.= o    |
                |   . o . o.* o  .|
                |      o +.. +E...|
                |       ..B o  .o |
                |       o*o=..   .|
                +-----------------+

Is the above information ok? (y/N): y


Signing certificate...
```  
- Bây giờ ta đã có 1 signed certificates  
<img src="https://i.imgur.com/NXszcwQ.png">  

- Tiếp theo ta có thể xóa `node3-request.pem`  
```
[root@node2 ~]# rm -f node3-request.pem
```  

