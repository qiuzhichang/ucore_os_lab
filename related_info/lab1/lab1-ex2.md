# lab1-ex2
Try below command (stack)

```
gcc -m32 -o lab1-ex2.exe lab1-ex2.c
```

Try to analysis the means of these output log.

```
which has no line number information.
 [ebp-12] --?? = 0xffffce5c --- -134488064
 [ebp-08] --?? = 0xffffce60 --- 0
 [ebp-04] --?? = 0xffffce64 --- 0
 [ebp+00] -oebp= 0xffffce68 --- 0xffffce98
 [ebp+04] -ret = 0xffffce6c --- 0x8048463
 [ebp+08] -- d = 0xffffce70 --- 1
 [ebp+12] -- e = 0xffffce74 --- 2
 [ebp+16] -- f = 0xffffce78 --- 3
0xf7e2fa63 in __libc_start_main () from /lib32/libc.so.6
```
