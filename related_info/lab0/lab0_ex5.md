#README
analysis lab0_ex5
```
echo "compile and analysis lab0_ex5"
echo "====================================="
gcc -m32 -g -o lab0_ex5.exe lab0_ex5.c
echo "====================================="
echo "using objdump to decompile lab0_ex5"
echo "====================================="
objdump -S lab0_ex5.exe
echo "====================================="
echo "using readelf to analyze lab0_ex5"
echo "====================================="
readelf -a lab0_ex5.exe
echo "====================================="
echo "using nm to analyze lab0_ex5"
echo "====================================="
nm lab0_ex5.exe
```
.参数压栈传递，并且是从右向左依次压栈。
.ebp总是指向当前栈帧的栈底。
.返回值通过eax寄存器传递。

objdump -S lab0_ex5.exe
```
0804841d <X>:
void X(int b) {
 804841d:	55                   	push   %ebp
 [push %ebp指令把ebp寄存器的值压栈，同时把esp的值减4，esp的值现在是0xffffce78]

 804841e:	89 e5                	mov    %esp,%ebp
 ［把这个值0xffffce78传送给ebp寄存器］

 8048420:	83 ec 18             	sub    $0x18,%esp
 ［编译器计算好函数需要的空间］

  if (b==1){
 8048423:	83 7d 08 01          	cmpl   $0x1,0x8(%ebp)
 8048427:	75 0e                	jne    8048437 <X+0x1a>

    printf("X:: b is 1!\n");
 8048429:	c7 04 24 00 85 04 08 	movl   $0x8048500,(%esp)
 8048430:	e8 bb fe ff ff       	call   80482f0 <puts@plt>

 8048435:	eb 0c                	jmp    8048443 <X+0x26>
 ［跳转到结尾 leave］

  }else{
    printf("X:: b is not 1!\n");
 8048437:	c7 04 24 0c 85 04 08 	movl   $0x804850c,(%esp)
 804843e:	e8 ad fe ff ff       	call   80482f0 <puts@plt>
  }
}
 8048443:	c9                   	leave  
 8048444:	c3                   	ret    

08048445 <main>:

int main(int argc, char * argv){
 8048445:	55                   	push   %ebp
 8048446:	89 e5                	mov    %esp,%ebp
 8048448:	83 e4 f0             	and    $0xfffffff0,%esp
 804844b:	83 ec 20             	sub    $0x20,%esp
 [0x20:在调用一个函数时, 编译器就计算好函数需要的空间, 然后esp = ebp-需要的空间, 通过ebp+偏移量来访问. 在函数里调用另外一个函数时, 原来fun的ebp值压栈]

    int a=2;
 804844e:	c7 44 24 1c 02 00 00 	movl   $0x2,0x1c(%esp)
 ［局部变量放在0x1c(%esp)地址中］

 8048455:	00
    X(a);
 8048456:	8b 44 24 1c          	mov    0x1c(%esp),%eax
 804845a:	89 04 24             	mov    %eax,(%esp)
 804845d:	e8 bb ff ff ff       	call   804841d <X>
 ［call由两个作用：
1 X(a)函数调用完之后要返回到call的下一条指令继续执行，所以把call的下一条指令的地址134513762［0x08048462］压栈，同时把esp的值减4，esp的值现在是0xffffce7c
2 修改程序计数器eip，跳转到X(a)函数的开头执行
 ］
}
```
具体的gdb调试：
```
(gdb) disas
Dump of assembler code for function main:
   0x08048445 <+0>:	push   %ebp
   0x08048446 <+1>:	mov    %esp,%ebp
   0x08048448 <+3>:	and    $0xfffffff0,%esp
   0x0804844b <+6>:	sub    $0x20,%esp
   0x0804844e <+9>:	movl   $0x2,0x1c(%esp)
   0x08048456 <+17>:	mov    0x1c(%esp),%eax
   0x0804845a <+21>:	mov    %eax,(%esp)
=> 0x0804845d <+24>:	call   0x804841d <X>
   0x08048462 <+29>:	leave  
   0x08048463 <+30>:	ret    
End of assembler dump.
(gdb) i r
eax            0x2	2
ecx            0xa09e48f9	-1600239367
edx            0xffffced4	-12588
ebx            0xf7fbe000	-134488064
esp            0xffffce80	0xffffce80
ebp            0xffffcea8	0xffffcea8
esi            0x0	0
edi            0x0	0
eip            0x804845d	0x804845d <main+24>
eflags         0x282	[ SF IF ]
cs             0x23	35
ss             0x2b	43
ds             0x2b	43
es             0x2b	43
fs             0x0	0
gs             0x63	99
(gdb) si
X (b=2) at lab0_ex5.c:1
1	void X(int b) {
(gdb) i r
eax            0x2	2
ecx            0xa09e48f9	-1600239367
edx            0xffffced4	-12588
ebx            0xf7fbe000	-134488064
esp            0xffffce7c	0xffffce7c
ebp            0xffffcea8	0xffffcea8
esi            0x0	0
edi            0x0	0
eip            0x804841d	0x804841d <X>
eflags         0x282	[ SF IF ]
cs             0x23	35
ss             0x2b	43
ds             0x2b	43
es             0x2b	43
fs             0x0	0
gs             0x63	99
(gdb) x/20 $esp
0xffffce7c:	134513762	2	-12476	-12468
0xffffce8c:	-136015747	-134487100	-134230016	134513787
0xffffce9c:	2	134513776	0	0
0xffffceac:	-136119709	1	-12476	-12468
0xffffcebc:	-134304534	1	-12476	-12572
(gdb) p/x $esp
$1 = 0xffffce7c
(gdb) x/20 $esp
0xffffce7c:	0x08048462	0x00000002	0xffffcf44	0xffffcf4c
0xffffce8c:	0xf7e4907d	0xf7fbe3c4	0xf7ffd000	0x0804847b
0xffffce9c:	0x00000002	0x08048470	0x00000000	0x00000000
0xffffceac:	0xf7e2fa63	0x00000001	0xffffcf44	0xffffcf4c
0xffffcebc:	0xf7feacea	0x00000001	0xffffcf44	0xffffcee4
(gdb)

跳转到x函数

(gdb) si
0x0804841e	1	void X(int b) {
(gdb) i r
eax            0x2	2
ecx            0xa09e48f9	-1600239367
edx            0xffffced4	-12588
ebx            0xf7fbe000	-134488064
esp            0xffffce78	0xffffce78
ebp            0xffffcea8	0xffffcea8
esi            0x0	0
edi            0x0	0
eip            0x804841e	0x804841e <X+1>
eflags         0x282	[ SF IF ]
cs             0x23	35
ss             0x2b	43
ds             0x2b	43
es             0x2b	43
fs             0x0	0
gs             0x63	99
(gdb)
```
readelf -a lab0_ex5.exe
```
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Intel 80386
  Version:                           0x1
  Entry point address:               0x8048320
  Start of program headers:          52 (bytes into file)
  Start of section headers:          5220 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         9
  Size of section headers:           40 (bytes)
  Number of section headers:         35
  Section header string table index: 32

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        08048154 000154 000013 00   A  0   0  1
  [ 2] .note.ABI-tag     NOTE            08048168 000168 000020 00   A  0   0  4
  [ 3] .note.gnu.build-i NOTE            08048188 000188 000024 00   A  0   0  4
  [ 4] .gnu.hash         GNU_HASH        080481ac 0001ac 000020 04   A  5   0  4
  [ 5] .dynsym           DYNSYM          080481cc 0001cc 000050 10   A  6   1  4
  [ 6] .dynstr           STRTAB          0804821c 00021c 00004a 00   A  0   0  1
  [ 7] .gnu.version      VERSYM          08048266 000266 00000a 02   A  5   0  2
  [ 8] .gnu.version_r    VERNEED         08048270 000270 000020 00   A  6   1  4
  [ 9] .rel.dyn          REL             08048290 000290 000008 08   A  5   0  4
  [10] .rel.plt          REL             08048298 000298 000018 08   A  5  12  4
  [11] .init             PROGBITS        080482b0 0002b0 000023 00  AX  0   0  4
  [12] .plt              PROGBITS        080482e0 0002e0 000040 04  AX  0   0 16
  [13] .text             PROGBITS        08048320 000320 0001c2 00  AX  0   0 16
  ［存放程序执行代码的一块内存区域］
  [14] .fini             PROGBITS        080484e4 0004e4 000014 00  AX  0   0  4
  [15] .rodata           PROGBITS        080484f8 0004f8 000024 00   A  0   0  4
  [16] .eh_frame_hdr     PROGBITS        0804851c 00051c 000034 00   A  0   0  4
  [17] .eh_frame         PROGBITS        08048550 000550 0000d0 00   A  0   0  4
  [18] .init_array       INIT_ARRAY      08049f08 000f08 000004 00  WA  0   0  4
  [19] .fini_array       FINI_ARRAY      08049f0c 000f0c 000004 00  WA  0   0  4
  [20] .jcr              PROGBITS        08049f10 000f10 000004 00  WA  0   0  4
  [21] .dynamic          DYNAMIC         08049f14 000f14 0000e8 08  WA  6   0  4
  [22] .got              PROGBITS        08049ffc 000ffc 000004 04  WA  0   0  4
  [23] .got.plt          PROGBITS        0804a000 001000 000018 04  WA  0   0  4
  [24] .data             PROGBITS        0804a018 001018 000008 00  WA  0   0  4
  ［存放程序中已初始化的全局变量的一块内存区域，静态分配］
  [25] .bss              NOBITS          0804a020 001020 000004 00  WA  0   0  1
  [存放程序中未初始化的全局变量的一块内存区域，静态分配]
  [26] .comment          PROGBITS        00000000 001020 000024 01  MS  0   0  1
  [27] .debug_aranges    PROGBITS        00000000 001044 000020 00      0   0  1
  [28] .debug_info       PROGBITS        00000000 001064 0000e3 00      0   0  1
  [29] .debug_abbrev     PROGBITS        00000000 001147 0000d0 00      0   0  1
  [30] .debug_line       PROGBITS        00000000 001217 00004e 00      0   0  1
  [31] .debug_str        PROGBITS        00000000 001265 0000b6 01  MS  0   0  1
  [32] .shstrtab         STRTAB          00000000 00131b 000146 00      0   0  1
  [33] .symtab           SYMTAB          00000000 0019dc 000490 10     34  50  4
  [34] .strtab           STRTAB          00000000 001e6c 000255 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)

There are no section groups in this file.

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  PHDR           0x000034 0x08048034 0x08048034 0x00120 0x00120 R E 0x4
  INTERP         0x000154 0x08048154 0x08048154 0x00013 0x00013 R   0x1
      [Requesting program interpreter: /lib/ld-linux.so.2]
  LOAD           0x000000 0x08048000 0x08048000 0x00620 0x00620 R E 0x1000
  LOAD           0x000f08 0x08049f08 0x08049f08 0x00118 0x0011c RW  0x1000
  DYNAMIC        0x000f14 0x08049f14 0x08049f14 0x000e8 0x000e8 RW  0x4
  NOTE           0x000168 0x08048168 0x08048168 0x00044 0x00044 R   0x4
  GNU_EH_FRAME   0x00051c 0x0804851c 0x0804851c 0x00034 0x00034 R   0x4
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x10
  GNU_RELRO      0x000f08 0x08049f08 0x08049f08 0x000f8 0x000f8 R   0x1

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .interp
   02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rel.dyn .rel.plt .init .plt .text .fini .rodata .eh_frame_hdr .eh_frame
   03     .init_array .fini_array .jcr .dynamic .got .got.plt .data .bss
   04     .dynamic
   05     .note.ABI-tag .note.gnu.build-id
   06     .eh_frame_hdr
   07     
   08     .init_array .fini_array .jcr .dynamic .got

Dynamic section at offset 0xf14 contains 24 entries:
  Tag        Type                         Name/Value
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
 0x0000000c (INIT)                       0x80482b0
 0x0000000d (FINI)                       0x80484e4
 0x00000019 (INIT_ARRAY)                 0x8049f08
 0x0000001b (INIT_ARRAYSZ)               4 (bytes)
 0x0000001a (FINI_ARRAY)                 0x8049f0c
 0x0000001c (FINI_ARRAYSZ)               4 (bytes)
 0x6ffffef5 (GNU_HASH)                   0x80481ac
 0x00000005 (STRTAB)                     0x804821c
 0x00000006 (SYMTAB)                     0x80481cc
 0x0000000a (STRSZ)                      74 (bytes)
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000015 (DEBUG)                      0x0
 0x00000003 (PLTGOT)                     0x804a000
 0x00000002 (PLTRELSZ)                   24 (bytes)
 0x00000014 (PLTREL)                     REL
 0x00000017 (JMPREL)                     0x8048298
 0x00000011 (REL)                        0x8048290
 0x00000012 (RELSZ)                      8 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x6ffffffe (VERNEED)                    0x8048270
 0x6fffffff (VERNEEDNUM)                 1
 0x6ffffff0 (VERSYM)                     0x8048266
 0x00000000 (NULL)                       0x0

Relocation section '.rel.dyn' at offset 0x290 contains 1 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
08049ffc  00000206 R_386_GLOB_DAT    00000000   __gmon_start__

Relocation section '.rel.plt' at offset 0x298 contains 3 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
0804a00c  00000107 R_386_JUMP_SLOT   00000000   puts
0804a010  00000207 R_386_JUMP_SLOT   00000000   __gmon_start__
0804a014  00000307 R_386_JUMP_SLOT   00000000   __libc_start_main

The decoding of unwind sections for machine type Intel 80386 is not currently supported.

Symbol table '.dynsym' contains 5 entries:
   Num:    Value  Size Type    Bind   Vis      Ndx Name
     0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.0 (2)
     2: 00000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
     3: 00000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main@GLIBC_2.0 (2)
     4: 080484fc     4 OBJECT  GLOBAL DEFAULT   15 _IO_stdin_used

Symbol table '.symtab' contains 73 entries:
   Num:    Value  Size Type    Bind   Vis      Ndx Name
     0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 08048154     0 SECTION LOCAL  DEFAULT    1
     2: 08048168     0 SECTION LOCAL  DEFAULT    2
     3: 08048188     0 SECTION LOCAL  DEFAULT    3
     4: 080481ac     0 SECTION LOCAL  DEFAULT    4
     5: 080481cc     0 SECTION LOCAL  DEFAULT    5
     6: 0804821c     0 SECTION LOCAL  DEFAULT    6
     7: 08048266     0 SECTION LOCAL  DEFAULT    7
     8: 08048270     0 SECTION LOCAL  DEFAULT    8
     9: 08048290     0 SECTION LOCAL  DEFAULT    9
    10: 08048298     0 SECTION LOCAL  DEFAULT   10
    11: 080482b0     0 SECTION LOCAL  DEFAULT   11
    12: 080482e0     0 SECTION LOCAL  DEFAULT   12
    13: 08048320     0 SECTION LOCAL  DEFAULT   13
    14: 080484e4     0 SECTION LOCAL  DEFAULT   14
    15: 080484f8     0 SECTION LOCAL  DEFAULT   15
    16: 0804851c     0 SECTION LOCAL  DEFAULT   16
    17: 08048550     0 SECTION LOCAL  DEFAULT   17
    18: 08049f08     0 SECTION LOCAL  DEFAULT   18
    19: 08049f0c     0 SECTION LOCAL  DEFAULT   19
    20: 08049f10     0 SECTION LOCAL  DEFAULT   20
    21: 08049f14     0 SECTION LOCAL  DEFAULT   21
    22: 08049ffc     0 SECTION LOCAL  DEFAULT   22
    23: 0804a000     0 SECTION LOCAL  DEFAULT   23
    24: 0804a018     0 SECTION LOCAL  DEFAULT   24
    25: 0804a020     0 SECTION LOCAL  DEFAULT   25
    26: 00000000     0 SECTION LOCAL  DEFAULT   26
    27: 00000000     0 SECTION LOCAL  DEFAULT   27
    28: 00000000     0 SECTION LOCAL  DEFAULT   28
    29: 00000000     0 SECTION LOCAL  DEFAULT   29
    30: 00000000     0 SECTION LOCAL  DEFAULT   30
    31: 00000000     0 SECTION LOCAL  DEFAULT   31
    32: 00000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    33: 08049f10     0 OBJECT  LOCAL  DEFAULT   20 __JCR_LIST__
    34: 08048360     0 FUNC    LOCAL  DEFAULT   13 deregister_tm_clones
    35: 08048390     0 FUNC    LOCAL  DEFAULT   13 register_tm_clones
    36: 080483d0     0 FUNC    LOCAL  DEFAULT   13 __do_global_dtors_aux
    37: 0804a020     1 OBJECT  LOCAL  DEFAULT   25 completed.6590
    38: 08049f0c     0 OBJECT  LOCAL  DEFAULT   19 __do_global_dtors_aux_fin
    39: 080483f0     0 FUNC    LOCAL  DEFAULT   13 frame_dummy
    40: 08049f08     0 OBJECT  LOCAL  DEFAULT   18 __frame_dummy_init_array_
    41: 00000000     0 FILE    LOCAL  DEFAULT  ABS lab0_ex5.c
    42: 00000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    43: 0804861c     0 OBJECT  LOCAL  DEFAULT   17 __FRAME_END__
    44: 08049f10     0 OBJECT  LOCAL  DEFAULT   20 __JCR_END__
    45: 00000000     0 FILE    LOCAL  DEFAULT  ABS
    46: 08049f0c     0 NOTYPE  LOCAL  DEFAULT   18 __init_array_end
    47: 08049f14     0 OBJECT  LOCAL  DEFAULT   21 _DYNAMIC
    48: 08049f08     0 NOTYPE  LOCAL  DEFAULT   18 __init_array_start
    49: 0804a000     0 OBJECT  LOCAL  DEFAULT   23 _GLOBAL_OFFSET_TABLE_
    50: 080484e0     2 FUNC    GLOBAL DEFAULT   13 __libc_csu_fini
    51: 00000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTab
    52: 08048350     4 FUNC    GLOBAL HIDDEN    13 __x86.get_pc_thunk.bx
    53: 0804a018     0 NOTYPE  WEAK   DEFAULT   24 data_start
    54: 0804841d    40 FUNC    GLOBAL DEFAULT   13 X
    55: 0804a020     0 NOTYPE  GLOBAL DEFAULT   24 _edata
    56: 080484e4     0 FUNC    GLOBAL DEFAULT   14 _fini
    57: 0804a018     0 NOTYPE  GLOBAL DEFAULT   24 __data_start
    58: 00000000     0 FUNC    GLOBAL DEFAULT  UND puts@@GLIBC_2.0
    59: 00000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    60: 0804a01c     0 OBJECT  GLOBAL HIDDEN    24 __dso_handle
    61: 080484fc     4 OBJECT  GLOBAL DEFAULT   15 _IO_stdin_used
    62: 00000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main@@GLIBC_
    63: 08048470    97 FUNC    GLOBAL DEFAULT   13 __libc_csu_init
    64: 0804a024     0 NOTYPE  GLOBAL DEFAULT   25 _end
    65: 08048320     0 FUNC    GLOBAL DEFAULT   13 _start
    66: 080484f8     4 OBJECT  GLOBAL DEFAULT   15 _fp_hw
    67: 0804a020     0 NOTYPE  GLOBAL DEFAULT   25 __bss_start
    68: 08048445    31 FUNC    GLOBAL DEFAULT   13 main
    69: 00000000     0 NOTYPE  WEAK   DEFAULT  UND _Jv_RegisterClasses
    70: 0804a020     0 OBJECT  GLOBAL HIDDEN    24 __TMC_END__
    71: 00000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
    72: 080482b0     0 FUNC    GLOBAL DEFAULT   11 _init

Histogram for `.gnu.hash' bucket list length (total of 2 buckets):
 Length  Number     % of total  Coverage
      0  1          ( 50.0%)
      1  1          ( 50.0%)    100.0%

Version symbols section '.gnu.version' contains 5 entries:
 Addr: 0000000008048266  Offset: 0x000266  Link: 5 (.dynsym)
  000:   0 (*local*)       2 (GLIBC_2.0)     0 (*local*)       2 (GLIBC_2.0)  
  004:   1 (*global*)   

Version needs section '.gnu.version_r' contains 1 entries:
 Addr: 0x0000000008048270  Offset: 0x000270  Link: 6 (.dynstr)
  000000: Version: 1  File: libc.so.6  Cnt: 1
  0x0010:   Name: GLIBC_2.0  Flags: none  Version: 2

Displaying notes found at file offset 0x00000168 with length 0x00000020:
  Owner                 Data size	Description
  GNU                  0x00000010	NT_GNU_ABI_TAG (ABI version tag)
    OS: Linux, ABI: 2.6.24

Displaying notes found at file offset 0x00000188 with length 0x00000024:
  Owner                 Data size	Description
  GNU                  0x00000014	NT_GNU_BUILD_ID (unique build ID bitstring)
    Build ID: 478290bdd1c70c1b3f9d2d02a8897fbb46968f2d
```

nm lab0_ex5.exe
```
0804a020 B __bss_start
0804a020 b completed.6590
0804a018 D __data_start
0804a018 W data_start
08048360 t deregister_tm_clones
080483d0 t __do_global_dtors_aux
08049f0c t __do_global_dtors_aux_fini_array_entry
0804a01c D __dso_handle
08049f14 d _DYNAMIC
0804a020 D _edata
0804a024 B _end
080484e4 T _fini
080484f8 R _fp_hw
080483f0 t frame_dummy
08049f08 t __frame_dummy_init_array_entry
0804861c r __FRAME_END__
0804a000 d _GLOBAL_OFFSET_TABLE_
         w __gmon_start__
080482b0 T _init
08049f0c t __init_array_end
08049f08 t __init_array_start
080484fc R _IO_stdin_used
         w _ITM_deregisterTMCloneTable
         w _ITM_registerTMCloneTable
08049f10 d __JCR_END__
08049f10 d __JCR_LIST__
         w _Jv_RegisterClasses
080484e0 T __libc_csu_fini
08048470 T __libc_csu_init
         U __libc_start_main@@GLIBC_2.0
08048445 T main
         U puts@@GLIBC_2.0
08048390 t register_tm_clones
08048320 T _start
0804a020 D __TMC_END__
0804841d T X
08048350 T __x86.get_pc_thunk.bx

```
