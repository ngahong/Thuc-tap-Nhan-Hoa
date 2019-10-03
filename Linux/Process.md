## Tiến trình trong Linux
Mục lục
#### 1. [Tiến trình là gì?](#1)
#### 2. [Phân loại tiến trình](#2)

<a name="1"></a>
**1. Tiến trình là gì?**  
Tiến trình (Process) đơn giản là một thể hiện của một hoặc nhiều tác vụ (luồng) liên quan thực thi trên cùng một máy. Nó không giống như một chương trình (program) hoặc một lệnh. Một chương trình có thể có 1 hoặc nhiều tiến trình. Một số tiến trình có thể độc lập hoặc liên quan với nhau. Vì vậy khi một tiến trình bị lỗi, nó có thể (hoặc không thể) ảnh hưởng đến các tiến trình khác đang chạy trên hệ thống.  
 Các tiến trình sử dụng nhiều tài nguyên hệ thống như bộ nhớ, CPU và các thiết bị ngoại vi như máy in và màn hình. Hệ điều hành (đặc biệt là kernel) chịu trách nhiệm phân bổ một phần thích hợp các tài nguyên này cho từng tiến trình và đảm bảo tổng thể sử dụng tối ưu.  

 Tiến trình có 3 trạng thái:
 - Đang chạy (running): Tiến trình chiếm quyền xử lý CPU, dùng tính toán hoặc thực hiện các công việc của mình.
 - Chờ (waiting): tiến trình bị Hệ điều hành tước quyền xử lý CPU và chờ đến lượt cấp phát khác.    
 - Tạm dừng (suspend): Hệ điều hành tạm dừng tiến trình. Tiến trình được đưa vào trạng thái ngủ (sleep). Khi cần thiết hay có nhu cầu, Hệ điều hành sẽ đánh thức (wake up) hay nạp lại mã lệnh của tiến trình vào bộ nhớ. Cấp phát tài nguyên CPU để tiến trình tiếp tục hoạt động.  
 Tại bất kì thời điểm nào luôn có nhiều tiến trình hoạt động. Hệ điều hành theo dõi chúng bằng cách gán cho mỗi tiến trình một ID gọi là PID (Process ID). PID là số duy nhất, được sử dụng để theo dõi trạng thái tiến trình, sử dụng CPU, sử dụng bộ nhớ...  

<a name ="2"></a>
 **2. Phân loại tiến trình**  

 Các PID mới thường được chỉ định theo thứ tự tăng dần khi các tiến trình được sinh ra. Do đó PID 1 biểu thị tiến trình **init** (tiến trình khởi tạo). Các tiến trình sau đó sẽ dần dần được đánh số PID cao hơn. 

 **2.1 Init Process**    

 *Init process* là tiến trình đầu tiên được khởi động sau khi bạn lựa chọn hệ điều hành trong boot loader. Trong cây tiến trình, *init process* là tiến trình cha của các tiến trình khác. Init process có đặc điểm sau:  
 - PID = 1;
 - Không thể kill init process.

**2.2 Parent process - Child process**

Trong hệ điều hành linux, các tiến trình được phân thành parent process và child process. Một tiến trình khi thực hiện lệnh fork() để tạo ra một tiến trình mới thì được gọi là *parent process* , các tiến trình mới được tạo gọi là child process.

<img src="https://i.imgur.com/s1mjoZ1.png">

Một parent process có thể có nhiều child process nhưng mỗi child process chỉ có một parent process. Thông tin của tiến trình ngoài PID (Process ID) còn có PPID (Parent Process ID).   

**2.3 Orphan process - Zombie Process**  
Khi parent process - child process hoạt động sẽ xảy ra một số trường hợp đặc biệt. Lúc đó Orphan process - Zombie process sẽ được hình thành.  
Khi một parent process bị tắt trước khi child process được tắt, tiến trình con đó sẽ trở thành một orphan process. Lúc này init process sẽ trở thành parent process của orphan process và thực hiện tắt chúng.    

Khi một child process được kết thúc, mọi trạng thái của child process sẽ được thông báo bởi lời gọi hàm wait() của parent process. Vì vậy kernel sẽ đợi parent process trả về hàm wait() trước khi tắt chilc process. Tuy nhiên vì một vài lý do mà parent process không thể trả về hàm wait(), khi đó child process sẽ trở thành một zombie process. Khi ở trạng thái này, tiến trình gần như giải phóng hoàn toàn bộ nhớ, chỉ lưu trữ một vài thông tin như PID, lượng tài nguyên sử dụng,... trên bảng danh sách tiến trình.
Câu lệnh tìm các zombie process:
```
# ps aux | grep Z
```
hoặc  
```
# ps aux | grep "defunct"
```
<img src="https://i.imgur.com/FvUf3WJ.png">  

**2.4 Deamon Process**  
Một Daemon Process là một tiến trình chạy nền. Nó sẽ luôn trong trạng thái hoạt động và sẽ được kích hoạt bởi một điều kiện hoặc câu lệnh nào đó. Trong Unix, các daemon thường được kết thúc bằng "d". Ví dụ như httpd, sshd, crond, mysqld...  






