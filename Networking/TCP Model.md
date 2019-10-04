### MÔ HÌNH TCP/IP 5 LỚP
Mục lục  
#### I. [Tổng quan về mô hình TCP/IP](#1)  
1 [Mô hình TCP/IP là gì?](#1)  
2 [Lịch sử hình thành và phát triển mô hình TCP/IP](#2) 
3 [Cách thức hoạt động của mô hình TCP/IP](#3) 
#### II. [Chức năng của các tầng trong mô hình TCP/IP](#4)  
1 [Tầng ứng dụng](#5)  
2 [Tầng vận chuyển](#6)
3 [Tầng internet](#7)
4 [Tầng liên kết dữ liệu](#8)
5 [Tầng vật lý](#9)  

<a name ="1"></a>
**I. Tổng quan về mô hình TCP/IP**  

**1. Mô hình TCP/IP là gì?**  
    TCP/IP (Transmission Control Protocol - Giao thức điều khiển truyền nhận/ Giao thức liên mạng) là một bộ giao thức trao đổi thông tin được sử dụng để truyền tải và kết nối các thiết bị trong mạng Internet. TCP/IP được phát triển để đáp ứng nhu cầu truyền tải dữ liệu tin cậy hơn cùng với khả năng phục hồi tự động.  
**2. Sự phát triển và hình thành mô hình TCP/IP**  
    Ý tưởng hình thành mô hình TCP/IP được bắt nguồn từ Bộ giao thức liên mạng trong công trình DARPA vào năm 1970. Trải qua vô sô năm nghiên cứu và phát triển của 2 kỹ sư Robert E. Kahn và Vinton Cert cùng sự hỗ trợ của không ít các nhóm nghiên cứu. Đầu năm 1978, giao thức TCP/IP được ổn định hóa với giao thức tiêu chuẩn được dùng hiện này của Internet đó là mô hình TCP/IP Version 4. 
       
<img src="https://i.imgur.com/0qhVYki.png">    

Vào năm 1975, cuộc thử nghiệm thông nối giữa 2 mô hình TCP/IP được diễn ra thành công. Cũng bắt đầu từ đây, cuộc thử nghiệm thông nối giữa các mô hình TCP/IP được diễn ra nhiều hơn và đều đạt được kết quả tốt. Cũng chính vì điều này, một cuộc hội thảo được Internet Architecture Broad mở ra, với sự tham dự của hơn 250 đại biểu của các công ty thương mại, từ đây giao thức và mô hình TCP/IP được phổ biến rộng rãi trên khắp thế giới.

**3. Cách thức hoạt động của mô hình TCP/IP**  
    Phân tích từ tên gọi, TCP/IP là sự kết hợp của 2 giao thức TCP và IP. Trong đó *IP* (Giao thức liên mạng) cho phép các gói tin được gửi đến đích đã định sẵn ban đầu. Giao thức *TCP* (Giao thức truyền vận) đóng vai trò kiểm tra và đảm bảo sự an toàn cho mỗi gói tin khi đi qua mỗi trạm. Trong quá trình này, nếu giao thức TCP nhận thấy gói tin bị lỗi, 1 tín hiệu sẽ được truyền đi và yêu cầu hệ thống gửi lại một gói tin khác. Quá trình này sẽ được làm rõ hơn ở mục chức năng của mỗi tầng trong mô hình TCP/IP.  

<a name="4"></a>
II. Chức năng của từng tầng trong mô hình TCP/IP 
Một mô hình TCP/IP tiêu chuẩn bao gồm 4 lớp được chồng lên nhau, bắt đầu từ tầng thấp nhất là tầng Vật lý (Physical)-> Tầng mạng (Network)-> Tầng giao vận (Transport) và cuối cùng là Tầng ứng dụng (Application).  
<img src="https://i.imgur.com/D3wf0AM.png">  
Tuy nhiên một số ý kiến cho rằng mô hình TCP/IP nên được chia làm 5 tầng, tức là tầng 2,3,4 được giữ nguyên và phân chia tầng cuối cùng thành 2 tầng Datalink và Physical. Đó cũng là xu hướng nghiên cứu hiện nay, phù hợp với tốc độ phát triển Internet và bóc tách vấn đề nghiên cứu rõ hơn.
<img src="https://i.imgur.com/kNEu7u4.png">  

**1. Application Layer (Tầng ứng dụng)**  
Tầng trên cùng (hoặc tầng 5) được gọi là tầng ứng dụng (Application Layer). Đây là nơi có hầu hết các ứng dụng TCP/IP. Phần mềm bạn tạo cho ứng dụng cuối của bạn thường sẽ tương tác với một số ứng dụng này. Ứng dụng TCP/IP được sử dụng phổ biến nhất là HTTP (Giao thức truyền tải siêu văn bản) sử dụng để lướt internet.  
<img src="https://i.imgur.com/FP4x6Ks.png">  

**2. Transport Layer (Tầng giao vận)**  

Tầng 4 được gọi là tầng Giao vận (Transport Layer). Tầng này có chức năng xử lý dữ liệu truyền, phân chia thành các gói nhỏ (segment), điều khiển các gói dữ liệu đi theo đúng với từng ứng dụng của nó và lựa chọn 1 giao thức phù hợp (TCP hoặc UDP).  

 <img src="https://i.imgur.com/wrmLKoO.png">
 Chúng ta sẽ nghiên cứu rõ 2 giao thức TCP và UDP ở một bài riêng.

 **3. Internet Layer (Tầng Mạng)  




