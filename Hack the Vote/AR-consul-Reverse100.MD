## Arkansas - Consul - Reverse - 100
The file simply outputs “Poor Bernie.” and exits when run.

```
$ file consul
consul: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.26, BuildID[sha1]=897c070c53ceb5b58080d891a68b96a197816773, not stripped
```

running strings reveals some interesting function names:
-    dont_call_me
-    fake_help
-    real_help


All of them just segfault when jumped to from main, except real_help which also prints
>    Leonardo De Pisa? Who's that–The next president?


Since this doesn't show up when running strings, I looked at the assembly and register values at different places throughout the real_help function to see if I could find where it was being pulled from.

```
(gdb) disas real_help
    ...
    0x0000000000400ae6 <+13>:    mov    $0x6012a0,%edi
    0x0000000000400aeb <+18>:    callq  0x40064c <sub_43E8>
    0x0000000000400af0 <+23>:    mov    %rax,-0x8(%rbp)
    ...


(gdb) x/s 0x6012a0
0x6012a0 <b0>:  "?XbaTeWb\023\067X\023C\\fT2\023J[b\032f\023g[Tg\325s\206G[X\023aXkg\023ceXf\\WXag2"


(gdb) b *main+31
(gdb) b *real_help+23
(gdb) r
Breakpoint 1, 0x0000000000400b5c in main ()
(gdb) jump *real_help
Breakpoint 2, 0x0000000000400af0 in real_help ()
(gdb) x/s $rax
0x602420:	"Leonardo De Pisa? Who's that–The next president?"
```

So the sub_43E8 function is decoding it. However, the function doesn't seem to be too complex as they line up pretty nicely

```
?XbaTeWb\023\067X\023C\\fT2\023J[b\032f\023g[Tg\325s\206G[X\023aXkg\023ceXf\\WXag2
Leonardo[SP]D   e[SP]Pi sa?[SP]Who'   s[SP]that-   T     he[SP]next[SP]Presi dent?
```

```
>>> ord('?') - ord('L')
13
>>> ord('X') - ord('e')
13
```

The offset is 13. But rotating the entire binary by 13 doesn't reveal anything else, so I tried some other rotation values
```
12
We didn't deserve Bernie.

18
The end is forever. But after that, you're good to go.
```



Finally, I automated it to perform all possible rotations...

```python
#!/usr/bin/env python
with open("consul", "rb") as file:
    b = file.read()
    for i in range(256):
        decoded = ""
        for ch in b:
            decoded += chr((ord(ch)+i) % 256)

        print(decoded)
```



... and grepped for flag.

```
    $ ./rotator | strings | grep flag
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@flag{write_in_bernie!}@@@@@@@@@@
```

flag{write_in_bernie!}
