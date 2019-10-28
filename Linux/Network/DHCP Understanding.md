### Tìm hiểu về DHCP 

DHCP (Dynamic Host Configuration Protocol) - Giao thức cấu hình máy chủ động.  
Lịch sử phát triển của nó phát triển ban đầu từ giao thức RARP, phát triển lên BOOTP và cuối cùng được cải tiến lên DHCP do nhu cầu cải tiến kĩ thuật mạng.  
- `RARP` là một giao thức mạng sơ khai được dùng bởi các máy client để yêu cầu có một địa chỉ Ipv4 để sử dụng cho mục đích liên lạc với các máy khác trong trạm. (thời kì đầu, khi mới có địa chỉ IP và các máy bắt đầu dùng địa chỉ IP để liên lạc thay cho địa chỉ MAC). Hạn chế của RARP là giao thức lớp 2, chỉ cấp địa chỉ IP và không cấp thêm một thông tin nào khác, là giao thức cấp địa chỉ tĩnh sơ khai và còn cần nhiều sự tương tác của người quản trị mạng khi phải cấu hình thông tin trên RARP server trước.  
- BOOTP: RARP rõ ràng là không đủ sức cung cấp thông tin cấu hình TCP/IP cho các máy tính. Để hổ trợ vừa cho các máy tính không có đĩa cứng vừa cho việc cấu hình TCP/IP tự động, vì thế mà BOOTP (Bootstrap) được tạo ra. BOOTP được tạo ra để giải quyết các hạn chế của RARP.  
    - Nó vẫn còn dựa vào quan hệ client/server, nhưng nó được triển khai ở tầng cao hơn, dùng UDP cho việc vận chuyển. Nó không còn phụ thuộc vào phần cứng đặc biệt nào của nhà sản xuất như là RARP.  
    - Hỗ trợ gởi thêm thông tin tới máy client ngoài địa chỉ IP.Thông tin thêm này thường được gởi trong một thông điệp duy nhất.  
    - Nó có thể sử dụng trong môi trường client và server ở trong những hệ thống mạng gồm nhiều NetID khác nhau. Điều này cho phép quản lý địa chỉ IP tập trung ở một server. 
    - Sử dụng IP/UDP (port 67 cho địa chỉ đích của server, và port 68 cho địa máy client) và do đó, có thể di chuyển qua router. Đây là một lợi thế vì có thể dùng giao thức phân giải địa chỉ qua môi trường liên mạng LAN.  
    - Nhưng bên cạnh đó, nó vẫn còn một số hạn chế là không cấp được địa chỉ IP động khi mà nhu cầu cấp địa chỉ động trở thành rõ rệt hơn bao giờ hết khi Internet thực sự khởi đầu cất cánh vào cuối thập niên 1990 và việc gán địa chỉ IP tĩnh và dùng mãi cho mỗi máy để chúng kết nối vào mạng là một phí phạm.

=> Cải tiến BOOTP thành DHCP.