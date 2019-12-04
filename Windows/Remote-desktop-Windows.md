# Remote desktop trên Windows 10  

Ý tưởng là ta đứng từ laptop và truy nhập vào PC bằng phương thức RDP (Remote Desktop Protocol) dưới quyền truy cập tài khoản Administrator hoặc user
<img src="https://i.imgur.com/hmeeFSr.png">

## Remote vào tài khoản Administrator  
1. Ấn đồng thời phím `Windows+R` để mở cửa sổ run. Tiếp theo gõ `lusrmgr.msc` để hiển thị cửa sổ quản lý Users và Groups. Ta chọn vào mục `Users` sẽ thấy danh sách các user hiện có (bao gồm cả Administrator)

<img src="https://i.imgur.com/CC7LQDb.png"> 

2. Chuột phải vào `Administrator` và chọn `Properties` sau đó bỏ dấu tích ở phần `Account is disable`. Đặt mật khẩu cho Admin sau đó ấn `OK`.  

<img src="https://i.imgur.com/cfKdUSu.png">

3. Bật chế độ Remote desktop connection  
Vào mục Search trên Windows và gõ Remote desktop để thiết lập kết nối từ xa. Bật chế độ Enable Remote Desktop  

<img src="https://i.imgur.com/VdIFODa.png">  

4. Từ một máy khác (Laptop) ta sẽ mở cửa sổ run (Windows+R) và gõ `mstsc`. 
Cửa sổ Remote Desktop Connection hiện ra, bạn hãy nhập địa chỉ IP của máy muốn remote (PC), điền tên user (Administrator) và chọn `Connect`. 

<img src="https://i.imgur.com/RvS8hxq.png">  

Tiếp theo bạn nhập password của Admin bên phía PC và chọn OK

<img src="https://i.imgur.com/FllIFCr.png">  

Và chọn `Yes` để tiếp tục kết nối.  
Vậy là bạn đã thành công trong việc Remote Desktop đến một host dưới tài khoản Administrator.  

## Remote vào tài khoản User   

Trên PC ta tạo một tài khoản user có tên là `user1`
và đặt mật khẩu cho user đó. 

<img src="https://i.imgur.com/aDG9meC.png">  

Vào cửa sổ `Remote Desktop Connection` và chọn `Select users that can remotely access this PC`

<img src="https://i.imgur.com/ISTcBF4.png">  

Một cửa sổ hiện ra để bạn Add thêm user trong danh sách remote desktop. Bạn điền tên user và chọn `Check Name`. Sau đó ấn `OK`  

<img src="https://i.imgur.com/soRxuFW.png">  

Tương tự như bước remote vào tài khoản Admin, từ một máy khác (Laptop) bạn có thể remote vào tài khoản user trên PC. 
