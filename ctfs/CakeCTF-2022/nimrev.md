# nimrev
## Analysis
We're given an executable file called _chall_. Running the **file** command on it gives us the following.
```
chall: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=7d489f42d4feb5f1b402f276543f3046dfed5ab4, for GNU/Linux 3.2.0, not stripped
```
Running the program takes our input and looks to compare it to something, as giving it the wrong input prints wrong.
```
> test
Wrong...
```
Since the file is not stripped, we can check its strings to see if we spot any interesting information. 
```
...
line
filename
trace
@Correct! <-- Correct message for the flag
@Wrong... <-- Flag is wrong
:*3$"
GCC: (Ubuntu 10.3.0-1ubuntu1~20.04) 10.3.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.0
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
stdlib_digitsutils.nim.c
nimCopyMem <-- nim function 
...

```
Based on the strings and the challenge name itself, we can assume that this is a nim program. We can also see the message we get back after giving the program the wrong input. We can check the disassembly to see when the ouput messages are decided and printed.
```
0000adfb  int64_t NimMainModule()

0000ae07      void* fsbase
0000ae07      int64_t rax = *(fsbase + 0x28)
0000ae22      int64_t* var_18
0000ae22      nimZeroMem(&var_18, 8)
0000ae27      int64_t var_48 = 0
0000ae39      int64_t* rax_1 = readLine_systemZio_271(stdin)
0000ae42      int64_t var_40 = 0
0000ae56      int64_t* list = newSeq(&NTIseqLcharT__lBgZ7a89beZGYPl8PiANMTA_, 0x18)
0000ae63      list[2].b = 0xbc
0000ae6b      *(list + 0x11) = 0x9e
0000ae73      *(list + 0x12) = 0x94
0000ae7b      *(list + 0x13) = 0x9a
0000ae83      *(list + 0x14) = 0xbc
0000ae8b      *(list + 0x15) = 0xab
0000ae93      *(list + 0x16) = 0xb9
0000ae9b      *(list + 0x17) = 0x84
0000aea3      list[3].b = 0x8c
0000aeab      *(list + 0x19) = 0xcf
0000aeb3      *(list + 0x1a) = 0x92
0000aebb      *(list + 0x1b) = 0xcc
0000aec3      *(list + 0x1c) = 0x8b
0000aecb      *(list + 0x1d) = 0xce
0000aed3      *(list + 0x1e) = 0x92
0000aedb      *(list + 0x1f) = 0xcc
0000aee3      list[4].b = 0x8c
0000aeeb      *(list + 0x21) = 0xa0
0000aef3      *(list + 0x22) = 0x91
0000aefb      *(list + 0x23) = 0xcf
0000af03      *(list + 0x24) = 0x8b
0000af0b      *(list + 0x25) = 0xa0
0000af13      *(list + 0x26) = 0xbc
0000af1b      *(list + 0x27) = 0x82
0000af2b      uint64_t (* var_28)(char arg1)
0000af2b      nimZeroMem(&var_28, 0x10)
0000af37      var_28 = colonanonymous__main_7
0000af43      int64_t var_38 = 0
0000af50      int64_t rsi
0000af50      if (list == 0)
0000af5b          rsi = 0
0000af56      else
0000af56          rsi = *list
0000af76      int64_t* mapped_list = map_main_11(&list[2], rsi, colonanonymous__main_7, 0)
0000af7f      int64_t var_30 = 0
0000af8c      int64_t rax_29
0000af8c      if (mapped_list == 0)
0000af97          rax_29 = 0
0000af92      else
0000af92          rax_29 = *mapped_list
0000afd0      if ((eqStrings(rax_1, join_main_42(&mapped_list[2], rax_29, nullptr)) ^ 1) != 0)
0000aff1          var_18 = copyString(&wrong_section)
0000afde      else
0000afde          var_18 = copyString(&correct_section)
0000b001      echoBinSafe(&var_18, 1)
0000b00b      int64_t rax_37 = rax - *(fsbase + 0x28)
0000b014      if (rax == *(fsbase + 0x28))
0000b01c          return rax_37
0000b016      __stack_chk_fail()
0000b016      noreturn

```
On the **NimMainModule**, we can see that the function takes user input and begins to build a new list. This list is then filled with values and then mapped a function called *colonanonymous__main_7* to create a new list. This new list (**mapped_list**) is then joined and used to compare with the user input as a string. If this compare fails, we get the wrong output. 
The program doesn't look to print the flag itself anywhere. This means that the program is checking whether the input given is the valid flag.

We know from the disassembly that the mapped_list is passed to the join_main_42 function; let's use a breakpoint in gdb-gef to see what's going on there. 
```
Starting program: /home/tripleta/ctf/cakectf-2022/nimrev/chall 
test

Breakpoint 1, 0x000055555555efaf in NimMainModule ()

[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x18              
$rbx   : 0x0055555555f070  →  <__libc_csu_init+0> endbr64 
$rcx   : 0x007ffff7d4d0a0  →  "CakeCTF{s0m3t1m3s_n0t_C}"
$rdx   : 0x0               
$rsp   : 0x007fffffffdc50  →  0x007ffff7d4c050  →  0x0000000000000004
$rbp   : 0x007fffffffdc90  →  0x007fffffffdca0  →  0x007fffffffdcc0  →  0x007fffffffdcf0  →  0x0000000000000000
$rsi   : 0x18              
$rdi   : 0x007ffff7d4d0a0  →  "CakeCTF{s0m3t1m3s_n0t_C}"
$rip   : 0x0055555555efaf  →  <NimMainModule+436> call 0x55555555e9a9 <join_main_42>
$r8    : 0x007ffff7d4c060  →  0xa0a000074736574 ("test"?)
$r9    : 0x7c              
$r10   : 0x007ffff7fa9be0  →  0x005555555766a0  →  0x0000000000000000
$r11   : 0x246             
$r12   : 0x00555555555320  →  <_start+0> endbr64 
$r13   : 0x007fffffffdde0  →  0x0000000000000001
$r14   : 0x0               
$r15   : 0x0               
$eflags: [zero carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00 
────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x007fffffffdc50│+0x0000: 0x007ffff7d4c050  →  0x0000000000000004	 ← $rsp
0x007fffffffdc58│+0x0008: 0x007ffff7d4d050  →  0x0000000000000018
0x007fffffffdc60│+0x0010: 0x007ffff7d4d090  →  0x0000000000000018
0x007fffffffdc68│+0x0018: 0x0000000000000000
0x007fffffffdc70│+0x0020: 0x0055555555eba3  →  <colonanonymous.main_7+0> endbr64 
0x007fffffffdc78│+0x0028: 0x0000000000000000
0x007fffffffdc80│+0x0030: 0x0000000000000000
0x007fffffffdc88│+0x0038: 0x1426d7380d12af00
──────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x55555555efa4 <NimMainModule+425> mov    edx, 0x0
   0x55555555efa9 <NimMainModule+430> mov    rsi, rax
   0x55555555efac <NimMainModule+433> mov    rdi, rcx
 → 0x55555555efaf <NimMainModule+436> call   0x55555555e9a9 <join_main_42>
   ↳  0x55555555e9a9 <join_main_42+0> endbr64 
      0x55555555e9ad <join_main_42+4> push   rbp
      0x55555555e9ae <join_main_42+5> mov    rbp, rsp
      0x55555555e9b1 <join_main_42+8> sub    rsp, 0x50
      0x55555555e9b5 <join_main_42+12> mov    QWORD PTR [rbp-0x38], rdi
      0x55555555e9b9 <join_main_42+16> mov    QWORD PTR [rbp-0x40], rsi
──────────────────────────────────────────────────────────────────────────────── arguments (guessed) ────
join_main_42 (
   $rdi = 0x007ffff7d4d0a0 → "CakeCTF{s0m3t1m3s_n0t_C}",
   $rsi = 0x00000000000018,
   $rdx = 0x00000000000000,
   $rcx = 0x007ffff7d4d0a0 → "CakeCTF{s0m3t1m3s_n0t_C}"
)
──────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall", stopped 0x55555555efaf in NimMainModule (), reason: BREAKPOINT
────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x55555555efaf → NimMainModule()
[#1] 0x55555555ed60 → NimMainInner()
[#2] 0x55555555eda0 → NimMain()
[#3] 0x55555555edf2 → main()

```

There's our flag, **CakeCTF{s0m3t1m3s_n0t_C}**.