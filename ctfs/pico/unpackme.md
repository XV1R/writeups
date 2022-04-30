# unpackme
## picoCTF 2022 Reverse Engineering
### Solution
Running commands such as readelf on the binary, we can see that there's virtually **no** data coming out. Furthermore, when we run *strings*, we get an interesting output where symbolic information and general headers don't seem to be found. Going off by the file name, we can assume this is packed by UPX. 

Downloading UPX and reading it's man page, we can see that we can unpack the file using a -d flag. 
Doing this, we're able to actually get output of the file 
```
tripleta@thewired:/tmp$ file unpackme-upx 
unpackme-upx: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=ce6e7d894ddeb04f318eaf921d191bfede4f925d, for GNU/Linux 3.2.0, not stripped
```
Now that we have the file in its unpacked state, we're able to reverse engineer it. 
Since it's not stripped, we can check to see if a main function exists
###### Note: I use Binary Ninja here
In the main function, we see:
```
00401ec6  66c745ec4e00       mov     word [rbp-0x14 {var_1c}], 0x4e
00401ecc  488d3d31110b00     lea     rdi, [rel data_4b3004]  {"What's my favorite number? "}
00401ed3  b800000000         mov     eax, 0x0
00401ed8  e813ef0000         call    _IO_printf
00401edd  488d45c4           lea     rax, [rbp-0x3c {var_44}]
00401ee1  4889c6             mov     rsi, rax {var_44}
00401ee4  488d3d35110b00     lea     rdi, [rel data_4b3020]
00401eeb  b800000000         mov     eax, 0x0
00401ef0  e88bf00000         call    __isoc99_scanf
00401ef5  8b45c4             mov     eax, dword [rbp-0x3c {var_44}]
00401ef8  3dcb830b00         cmp     eax, 0xb83cb
00401efd  7543               jne     0x401f42

```
In this snippet, we can see printf being called with data_4b3004 _("What's my favorite number? ")_ as its argument. A few lines below, we see call to scanf followed by a **cmp** instrunction. We can try to see if this value is the desired value.
Converting **0xb83cb** to decimal gives us **754635**
Running the program with this:
```
tripleta@thewired:/tmp$ ./unpackme-upx 
What's my favorite number? 754635
picoCTF{up><_m3_f7w_e510a27}
```
