We start this level by listing the files in the current directory:
```bash
level0@RainFall:~$ ls -la
total 737
dr-xr-x---+ 1 level0 level0     60 Mar  6  2016 .
dr-x--x--x  1 root   root      340 Sep 23  2015 ..
-rw-r--r--  1 level0 level0    220 Apr  3  2012 .bash_logout
-rw-r--r--  1 level0 level0   3530 Sep 23  2015 .bashrc
-rwsr-x---+ 1 level1 users  747441 Mar  6  2016 level0
-rw-r--r--  1 level0 level0    675 Apr  3  2012 .profile
```

We notice that the **SUID bit (s) is set**, meaning that when executed, it runs with the privileges of **level1**.

We start by running the executable:
```bash
level0@RainFall:~$ ./level0
Segmentation fault (core dumped)
level0@RainFall:~$ ./level0 1
No !
```

Let's try to find some informations using **GDB**:
```bash
(gdb) b main
Breakpoint 1 at 0x8048ec3
(gdb) r
Starting program: /home/user/level0/level0

Breakpoint 1, 0x08048ec3 in main ()
```

And when disassembling the main we notice a comparison:
```bash
0x08048ed9 <+25>:	cmp    $0x1a7,%eax
```

Converting `0x1a7` from hex to dec gives us `423`, let's try running this with our executable:
```bash
level0@RainFall:~$ ./level0 423
$ cat /home/user/level1/.pass
1fe8a524fa4bec01ca4ea2a869af2a02260d4a7d5fe7e7c24d8617e6dca12d3a
```

This way we get our first flag and move on!