# GOT and PLT

There are many functions when programming in C: `fgets`, `system`, `fopen`, `exit`, `dup2` etc. All these functions are stored in the C standard library, which we call libc.

Libc is essentially a shared object file that stores all sorts of variables and functions a binary will need. Almost every binary imports this library at an address, usually this address is different every execution due to ASLR.

{% hint style="info" %}
Note: not all binaries need libc. Some don't use libc functions at all, some binaries are static and carry all of the libc functions they need along with them.
{% endhint %}

{% hint style="info" %}
Note 2: We're talking about libc here, but this applies to all dynamic libraries.
{% endhint %}

The problem arises: how can the binary know where the function it needs is located if the address at which libc is imported is constantly changing? That's where the **GOT** and the **PLT** come in.

## GOT and PLT <a id="got-and-plt"></a>

GOT stands for `Global Offset Table`. It is a section existing in the binary that holds the real addresses of functions that are in dynamic libraries.  For example, `printf@got` holds the address of `printf`.

PLT stands for `Process Linkage Table`. It holds PLT "stubs" - small pieces of code, of which their purpose is to jump to the correct function, or set things up.

When dealing with dynamic functions, the program will call upon stubs in the PLT, not the functions directly.

At the beginning of the program, the GOT may hold the addresses of the PLT instead of the actual addresses, or it may hold the addresses of resolution or identifier functions.

The first time running a dynamically linked function, the PLT will be jumped to. The PLT will fall into it's "default" nature, resolving the address of the actual function, updating the GOT to hold this address, then jump to the function.

Afterwards, anytime the PLT is called again, it will simply read the GOT to get the address of the function, then jump there.

This is known as "lazy binding", as it is unlikely that the location of the function called has changed, whilst also saving some CPU cycles.

## Practical analysis <a id="practical-analysis"></a>

If we jump into a program and set a breakpoint before the program is going to call the function, we can see how it gets accessed \(my binary has been compiled with PIE, so I have to run it before accessing the location\)

```c
$ gdb -q  ./vuln
...
gdb-peda$ b *main
Breakpoint 1 at 0x61d
gdb-peda$ r
^C
gdb-peda$ disas main
......
gdb-peda$ b *0x804845f
Breakpoint 2 at 0x804845f
gdb-peda$ c
Continuing
....
Breakpoint 2, 0x804845f in main ()
gdb-peda$ x/i $pc
```

So we're just about to call `puts@plt`. Let's step through until we get to the actual `puts` function.

```c
gdb-peda$ si // step into
gdb-peda$ x/i $pc
```

We're in the PLT, and we see that we're performing a normal jump. However, if we analyse what `0x804a00c` is

{% hint style="info" %}
Note: This is a jump to a function pointer. The processor will dereference the pointer, resulting in jumping to the address.
{% endhint %}

{% hint style="info" %}
Note 2: The pointer is in the `.got.plt` section
{% endhint %}

```c
gdb-peda$ x/wx 0x804a00c
gdb-peda$ si
gdb-peda$ x/2i $pc
```

That's weird, we just jumped to the next instruction. Ohhh, it's because we haven't called `puts` before, so we need to trigger the first lookup. It pushes the slot number \(0x0\) on to the stack, then calling the routine to lookup the symbol name.

This happens to be the beginning of the `.plt` section. What does this stub do?

```c
gdb-peda$ si
gdb-peda$ si
gdb-peda$ x/2i $pc
=> 0x80482f0: push   DWORD PTR ds:0x804a004
   0x80482f6: jmp    DWORD PTR ds:0x804a008
```

Now, we push the value of the second entry in `.got.plt`, then jump to the address stored in `0x804a008`. What are those values?

```c
gdb-peda$ x/2wx 0x804a004
0x804a004:  0xf7ffd918  0xf7fedf40
gdb-peda$ vmmap
...
0xf7fd9000 0xf7ffb000 r-xp    22000 0      /lib/i386-linux-gnu/ld-2.24.so
0xf7ffc000 0xf7ffd000 r--p     1000 22000  /lib/i386-linux-gnu/ld-2.24.so
0xf7ffd000 0xf7ffe000 rw-p     1000 23000  /lib/i386-linux-gnu/ld-2.24.so
...
```

So it seems as though the first value is pointing to the `rw` page of `ld.so`, and the second value pointing to the `rx` page of `ld.so`.

So finally, we got to the information that we were originally looking for. The two addresses in the `.got.plt` section are populated by the linker.loader `ld.so` at the time it is loading the binary.

Eventually, in the `ld.so` we will reach a `ret` instruction which will point to the symbol. Let's take a look at the stack layout

```c
gdb-peda$ x/i $pc
=> 0xf7fedf5b:  ret    0xc
gdb-peda$ ni
gdb-peda$ info symbol $pc
puts in section .text of /lib/i386-linux-gnu/libc.so.6
gdb-peda$ x/4wx $esp
0xffffcc2c: 0x08048464  0x08048500  0xffffccf4  0xffffccfc
gdb-peda$ x/s *(int *)($esp+4)
0x8048500:  "Hello world!"
```

So that was rather a long trip just to get from `main` to `puts`. But now, we can just take a look in the `.got.plt` section, disassembling `puts@plt` to verify the address

```c
gdb-peda$ disass 'puts@plt'
0x08048300 <+0>:	jmp    DWORD PTR ds:0x804a00c
0x08048306 <+6>:	push   0x0   
0x0804830b <+11>:	jmp    0x80482f0
End of assembler dump.
gdb-peda$ x/wx 0x804a00c
0x804a00c:	0xf7e4b870
gdb-peda$ info symbol 0xf7e4b870
puts in section .text of /lib/i386-linux-gnu/libc.so.6
```

So now, if the program were to do `call puts@plt`, it'd result in a `jmp`. At this point, the overhead of the relocation is one extra jump and derefrencing the pointer, however, the GOT is very often in L1 or L2 cache so that deference wouldn't be significant.

How did the `.got.plt` get updated? That's the reason that a pointer to the beginning of the GOT was passed as an argument back to `ld.so`, which that inserted the proper address in the GOT to replace the previous address which pointed to the next instruction in the PLT.

## Small bit on attacks <a id="small-bit-on-attacks"></a>

If full [RELRO](https://euanb26.gitbook.io/resources/cybersec/binary-exploitation/untitled/relro) is not on, the GOT is not read-only. This can lead to a GOT overwrite attack, in which we overwrite the GOT entry of a function with another address. Say, for example, puts is called on our input after a vulnerability that gave us an arbitrary write. We would be able to overwrite puts in the GOT with the address of system in the PLT perhaps, then input /bin/sh.

When `puts@plt` would run, it would check `puts@got`, which would hold the address of system, and then jump there, allowing us to control the program's execution.

