# Ret2dlresolve

As we saw in [Static vs Dynamic Linking](../../theory/static-vs-dynamic-linking.md) , dynamically linked binaries are linked with a libc file when they're executed. So when a program has ASLR turned on, it won't know the location of where the functions are inside the libc. It has to go through some checks to see where everything is. We can target this functionality.

As a way to see this: We don't need to find the libc version or have any leaks, woooooo!

## Background knowledge <a id="background-knowledge"></a>

The `.dynamic` section of a binary contains multiple information used by `ld.so` to resolve the symbols at runtime.

```c
$ readelf -d ./binary
Dynamic section at offset 0xec8 contains 27 entries:
Tag        Type                         Name/Value
0x00000001 (NEEDED)                     Shared library: [libc.so.6]
0x0000000c (INIT)                       0x43c
0x0000000d (FINI)                       0x744 
0x00000019 (INIT_ARRAY)                 0x1ec0 
0x0000001b (INIT_ARRAYSZ)               4 (bytes) 
0x0000001a (FINI_ARRAY)                 0x1ec4 
0x0000001c (FINI_ARRAYSZ)               4 (bytes) 
0x6ffffef5 (GNU_HASH)                   0x1ac 
0x00000005 (STRTAB)                     0x2ac 
0x00000006 (SYMTAB)                     0x1cc 
0x0000000a (STRSZ)                      195 (bytes) 
0x0000000b (SYMENT)                     16 (bytes) 
0x00000015 (DEBUG)                      0x0 
0x00000003 (PLTGOT)                     0x1fc0 
0x00000002 (PLTRELSZ)                   48 (bytes) 
0x00000014 (PLTREL)                     REL 
0x00000017 (JMPREL)                     0x40c 
0x00000011 (REL)                        0x3bc 
0x00000012 (RELSZ)                      80 (bytes) 
0x00000013 (RELENT)                     8 (bytes) 
0x0000001e (FLAGS)                      BIND_NOW 
0x6ffffffb (FLAGS_1)                    Flags: NOW PIE 
0x6ffffffe (VERNEED)                    0x38c 
0x6fffffff (VERNEEDNUM)                 1 
0x6ffffff0 (VERSYM)                     0x370 
0x6ffffffa (RELCOUNT)                   4 
0x00000000 (NULL)                       0x0
```

We will focus on `SYMTAB`, `STRTAB`, `JMPREL`.

### JMPREL <a id="jmprel"></a>

The `JMPREL` segment \(corresponding the `rel.plt`\) stores a table called _Relocation table_. Each entry maps to a symbol.

```c
$ readelf -r ./binary​Relocation
section '.rel.dyn' at offset 0x3bc contains 10 entries: 
Offset     Info    Type            Sym.Value  Sym. Name
00001ec0  00000008 R_386_RELATIVE   
00001ec4  00000008 R_386_RELATIVE   
00001ff8  00000008 R_386_RELATIVE   
00002004  00000008 R_386_RELATIVE   
00001fe4  00000106 R_386_GLOB_DAT  00000000   _ITM_deregisterTMClone
00001fec  00000706 R_386_GLOB_DAT    00000000   __gmon_start__
00001ff0  00000906 R_386_GLOB_DAT    00000000   stdin@GLIBC_2.
000001ff4  00000b06 R_386_GLOB_DAT    00000000   stdout@GLIBC_2.
000001ffc  00000c06 R_386_GLOB_DAT    00000000   _ITM_registerTMCloneTa​

Relocation section '.rel.plt' at offset 0x40c contains 6 entries: 
Offset     Info    Type            Sym.Value  Sym. Name
00001fcc  00000207 R_386_JUMP_SLOT   00000000   printf@GLIBC_2.0
00001fd0  00000307 R_386_JUMP_SLOT   00000000   fflush@GLIBC_2.0
00001fd4  00000407 R_386_JUMP_SLOT   00000000   fgets@GLIBC_2.0
00001fd8  00000607 R_386_JUMP_SLOT   00000000   puts@GLIBC_2.0
00001fdc  00000807 R_386_JUMP_SLOT   00000000   __libc_start_main@GLIBC_2.0
00001fe0  00000a07 R_386_JUMP_SLOT   00000000   memset@GLIBC_2.0
```

The type of these entries is `ELF\_REL`, which is defined as followed. The size of one entry is 8 bytes

```c
typedef uint32_t Elf32_Addr ; 
typedef uint32_t Elf32_Word ; 
typedef struct 
{
   Elf32_Addr r_offset ; /* Address */ 
   Elf32_Word r_info ; /* Relocation type and symbol index */ 
} Elf32_Rel ; 
#define ELF32_R_SYM(val) ((val) >> 8) 
#define ELF32_R_TYPE(val) ((val) & 0xff)
```

Let's take a look at the first entry in our `rel.plt` table.

```c
Offset     Info    Type            Sym.Value  Sym. Name
00001fcc  00000207 R_386_JUMP_SLOT   00000000   printf@GLIBC_2.0
```

* The _Offset_ is the address of the GOT entry in our table, in this case `0x00001fcc`.
* The _Info_ stores additional metadata such as `ELF_R_SYM` or `ELF_R_TYPE`.
* The _Sym. Name_ gives us the name of our symbol, in this case `printf@GLIBC_2.0`
* According to the MACROS in the `#define` section

  * `ELF32_R_SYM(r_info) == 1`
  * `ELF32_R_TYPE(r_info) == 7 (R_386_JUMP_SLOT)`
  * `R_SYM = 1`

  `​`

### STRTAB <a id="strtab"></a>

STRTAB is simply a table which contains the strings for symbols names \(look at the value of STRTAB when we did looked at the `.dynamic` section\).

```c
gdb-peda$ x/10s 0x2ac
0x2ac:	""
0x2ad:	"libc.so.6"
0x2b7:	"_IO_stdin_used"
0x2c6:	"fflush"
0x2cd:	"puts"
0x2d2:	"stdin"
0x2d8:	"printf"
0x2df:	"fgets"
0x2e5:	"memset"
0x2ec:	"stdout"
```

### SYMTAB <a id="symtab"></a>

This table holds relevant symbols information. Each entry is a `ELF32_Sym` structure and is 16 bytes

```c
typedef struct 
{ 
   Elf32_Word st_name ; /* Symbol name (string tbl index) */
   Elf32_Addr st_value ; /* Symbol value */ 
   Elf32_Word st_size ; /* Symbol size */ 
   unsigned char st_info ; /* Symbol type and binding */ 
   unsigned char st_other ; /* Symbol visibility under glibc>=2.2 */ 
   Elf32_Section st_shndx ; /* Section index */ 
} Elf32_Sym ;
```

The firtst field,`st_name`, gives the offset in _STRTAB_ where the name of the symbol begins. The other fields aren't used in the exploit, so they won't be covered here. However, they can be found heree: [https://www.cs.princeton.edu/courses/archive/fall01/cs217/assignment5/2.1.html](https://www.cs.princeton.edu/courses/archive/fall01/cs217/assignment5/2.1.html)​

The `ELF_R_SYM(r_info) == 1` variable \(from _JMPREL_ table\) gives the index of the `ELF32_SYM` in _SYMTAB_ for the specified symbol, in this case it's set to 1

```c
// (SYMTAB + index*sizeof(entry)) where index = ELF32_R_SYM(r_info)
gdb-peda$ x/4w 0x1cc + 1*16
0x1dc:	U"~"
0x1e4:	U""
0x1e8:	U" ,"
0x1f4:	U""

// (STRTAB + st_name)
gdb-peda$ x/s 0x2ac + 0x1dc
0x488 <fflush@plt+8>:	""
gdb-peda$ x/s 0x2ac + 0x1e4
0x490 <fgets@plt>:	"\377\243\024"
gdb-peda$ x/s 0x2ac + 0x1e8
0x494 <fgets@plt+4>:	""
gdb-peda$ x/s 0x2ac + 0x1f4
0x4a0 <puts@plt>:	"\377\243\030"
```

Adding the first 4 bytes from ELF32\_SYM to STRTAB gives the address of the symbol name.

## \_dl\_runtime\_resolve <a id="_dl_runtime_resolve"></a>

With the concepts of the three parts of the `.dynamic section` now we can see how resolving symbols works \(which can be seen in the GOT and PLT **pratical analysis** section\)

As a reminder:

```c
gdb-peda$ x/3i $eip
=> 0x8048300 <read@plt>:	jmp    DWORD PTR ds:0x804a00c
   0x8048306 <read@plt+6>:	push   0x0
   0x804830b <read@plt+11>:	jmp    0x80482f0
gdb-peda$ x/1xw 0x804a00c
0x804a00c:	0x08048306
gdb-peda$ x/2i 0x80482f0
   0x80482f0:	push   DWORD PTR ds:0x804a004
   0x80482f6:	jmp    DWORD PTR ds:0x804a008
```

1. The program reads the GOT value from `0x804a00c` and jumps back into the PLT section
2. Push the parameter `0x0` to the stack
3. Push any extra parameter on to the stack and jump to the resolver

This proccess is equivalent to the following

```c
_dl_runtime_resolve(link_map, rel_offset)
```

* `rel_offset` gives the offset of the `ELF32_REL` in JMPREL table
* `link_map(0x804a004)` is essentially a list with all the loaded libraries
* `_dl_runtime_resolve` uses this list to resolve the symbol

After relocating the symbol and its entry in SYMTAB populated, the initial call of `read` will be called. The pseudocode below summarises the process described

```c
// call of unresolved read(0, buf, 0x100)
_dl_runtime_resolve(link_map, rel_offset) {
    Elf32_REL * rel_entry = JMPREL + rel_offset ;
    Elf32_SYM * sym_entry = &SYMTAB [ ELF32_R_SYM ( rel_entry -> r_info )];
    char * sym_name = STRTAB + sym_entry -> st_name ;
    _search_for_symbol_(link_map, sym_name);
    // invoke initial read call now that symbol is resolved
    read(0, buf, 0x100);
}
```

## Exploit <a id="exploit"></a>

As you may or may not have seen, there aren't any bound checks to validate our data. Therefore, the main idea is to provide a massive `rel_offset` , such that the`rel_entry` can be found in our controllable area. We can then craft forged structures for `ELF32_REL` and `ELF32_SYM`, which'll force `dl_runtime_reoslve()` to bind the `system()` function call.

The key is that the index of the orresponding pseudo-entry should be calculated correctly

It's also important to not forget that our function will be called after resolving, so the parameter should be on the stack _before_ calling the resolver

### Demonstration <a id="demonstration"></a>

Suppose that

* JMPREL @ `0x0`
* SYMTAB @ `0x100`
* STRTAB @ `0x200`
* controllable area @ `0x300`

We need to craft our `ELF32_REL` and `ELF32_SYM` somwhere within the controllable area and provide a `rel_offset` in which the resolver reads our specially forged structure. Let's suppose that the controllable area has the following layout \(could generate this via stack pivoting\)

```c
             +--------+
r_offset     |GOT     |  0x300     
r_info       |0x2100  |  0x304
alignment    |AAAAAAAA|  0x308
st_name      |0x120   |  0x310
st_value     |0x0     |
st_size      |0x0     |
others       |0x12    |
sym_string   |"syst   |  0x320
             |em\x00" |
             +--------+
```

When `dl_runtime_resolver(link_map, 0x300)` is called, the `0x300` offset is used to get the `ELF32_REL * rel_entry = JMPREL + 0x300 == 0x300`.

The `ELF32_SYM` is access using the `r_info` field from `0x304`. `ELF32_SYM * sym = &SYMTAB[(0x2100 >> 8)] == 0x310`

The last step is to compute the address of the symbol string. This is done bby adding `st_name` to `STRTAB` : `const char *name = STRTAB + 0x120 == 0x320`

{% hint style="info" %}
Note: the SYMTAB accesses its entries as an array, therefore `ELF32_SYM` should be aligned to 0x10 bytes
{% endhint %}

Now that we control `st_name`, we can force the resolver to relocate `system` and call `system("sh")` to pwn the system.

