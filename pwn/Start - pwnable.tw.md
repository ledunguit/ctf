# Start - pwnable.tw
## Challenge

<p align="center">
  <img src="https://user-images.githubusercontent.com/64201705/138208384-1665452f-2327-4edc-a780-b3f2a8938387.png">
</p>

### File
<p align="center">
  <img src="https://user-images.githubusercontent.com/64201705/138210568-1c2f54c4-72d1-4192-ad45-9e8699325d50.png">
</p>

### Checksec
<p align="center">
  <img src="https://user-images.githubusercontent.com/64201705/138210943-fd316314-5f7c-4ce3-8d89-ef94e4917aa6.png">
</p>

### Run
<p align="center">
  <img src="https://user-images.githubusercontent.com/64201705/138211025-40281b14-c94f-462a-96cb-4951d2e5a82f.png">
</p>

### Disassemble
<p align="center">
  <img src="https://user-images.githubusercontent.com/64201705/138210873-5415e47e-b060-4ab5-b029-6f4657dd8cb6.png">
</p>

### Phân tích
Luồng thực thi của chương trình:
- Push esp: đẩy giá trị esp vào stack
- Push ```0x804809d```: đẩy địa chỉ đang trỏ tới hàm thoát vào stack để kết thúc chương trình khi gọi return (Khi phân tích bằng IDA hoặc X96DBG sẽ thấy rõ hơn)
- Khởi tạo giá trị cho các thanh ghi (```xor eax, eax``` nghĩa là ```eax = 0```, tương tự với các thanh ghi ```ebx, ecx, edx```)
- Thực hiện ```5``` lệnh push để đưa dòng chữ ```Let's start the CTF``` vào stack.
- Chương trình dùng ```0x80``` để gọi các lệnh hệ thống (Linux Syscall)
- Giá trị ```0x80``` đầu tiên, ```eax = 4```, chương trình gọi lệnh ```sys_write()``` để in ra stdout (```ebx = 1```) ```20``` kí tự (```edx = 0x14```) tại địa chỉ esp (```ecx = esp```), mục đích là in dòng ```Let's start the CTF``` ra màn hình.
- Giá trị ```0x80``` thứ hai, ```eax = 3```, chương trình gọi lệnh ```sys_read()``` đọc tối đa 60 ký tự (```edx = 0x3c```) từ stdin (```ebx = 0```), lưu vào stack tại vị trí esp (```ecx = esp```).
- Tăng giá trị ```esp``` lên 20 và return.

<p align="center">
  <img src="hmm">
</p>

