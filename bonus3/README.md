For the last time we use `radade2` and check the main:

```bash
[0x08048440]> pdf @ sub.main_80484f4
[...]
│           0x08048533      83bc249c00..   cmp dword [var_9ch], 0
│       ┌─< 0x0804853b      7406           je 0x8048543
[...]
│      ││   0x080485eb      c744240407..   mov dword [var_4h], 0x8048707 ; [0x8048707:4]=0x2f006873
│      ││   0x080485f3      c704240a87..   mov dword [esp], str._bin_sh ; [0x804870a:4]=0x6e69622f ; "/bin/sh"
│      ││   0x080485fa      e821feffff     call sym.imp.execl
```

We see a simple comparison that just needs an empty string to work so we try it when executing our bonus3:

```bash
bonus3@RainFall:~$ ./bonus3 ""
$ cat /home/user/end/.pass
3321b6f81659f9a71c76616f606e4b50189cecfea611393d5d649f75e157353c
```

And finally we finished all levels !
