# RELRO

## Background <a id="background"></a>

This requires a bit of knowledge about the[ GOT and PLT tables](https://tango37645.gitbook.io/binexp/bin-exp/theory/got-and-plt). If you're not sure what they are, go take a look at the page.

But if we need a simple refresher, we know that the GOT \(Global Offset Table\) is used to dynamically find functions that are located in shared libraries, such as libc and ld. These calls, one the program has been run at least once, will point to the PLT \(Procedure Linkage Table\), which stores the x86 instructions to call the specific function from the GOT.

There are a few implications of this:

* The PLT needs to be located at a fixed offset from the `.text` section
* As GOT contains data used by different parts of the program, it needs to be allocated at a known static address in memory
* As the GOT is lazily bound, it needs to be writable

As GOT exists at a predefined place in memory, an attacker could exploit a vulnerability to write 4 bytes at a controlled place in memory \(such as integer overflows leading to out-of-bounds write\), may be exploited to allow arbitrary code execution.

## RELRO <a id="relro"></a>

To prevent this, the compiler needs to ensure that the linker resolves all dynamically linked functions at the beginning of executing the binary, to make the GOT read-only. This is known as RELRO \(Relocation Read-Only\). Having the GOT being read-only stops the overwriting of its entries.

RELRO can be turned on via

```c
gcc -g -O0 -Wall -z,relro,-z,now -o out file.c
```

* `-g` : Produces debugging information in the OS native format
* `-O`: Optimisation level \(0 = compilation time\)
* `-W` : Warning level \(all = turn on all warnings\)
* `-z,relro,-z,now` : Turn on full RELRO
* `-o` : output filename

### Partial RELRO <a id="partial-relro"></a>

You can also turn on partial RELRO by using `-z,relro` and not having the `-z,now` option.

In partial RELRO, the non-PLT part of the GOT section \(`.got` from readelf output\) is read-only, however, the `.got.plt` is still writable. In full RELRO, both the `.got` and `.got.plt` are marked as read-only.

Both partial and full RELRO reorder the ELF internal data sections to prevent them from being overwritten in the event of a BOF, however, only full RELRO mitigates the technique of overwriting the GOT entry to get control of program execution.

