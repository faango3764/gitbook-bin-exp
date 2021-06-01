# Static vs Dynamic Linking

A file can either be statically or dynamically linked. It is common nowadays to see programs dynamically linked.

## Statically linked <a id="statically-linked"></a>

_Statically linked binaries_ are ones where they contain the necessary features to run, which are linked at compile time, ie. they live inside the binary. This is an ideal way to compile programs if they're rather small so that they're portable.

### Advantage <a id="advantage"></a>

The advantage of this is that the program can run as soon as it's started, it doesn't need to wait for any libraries to be included.

Another advantage is that it removes the external dependency on the libraries. If the library that you're using changes, then the program won't need to worry about it, as it has the library already imported into the program, meaning that hopefully, your program won't break due to the new function.

The binary is portable across all platforms. Due to the binary having the functions inside the code, it again doesn't need any external dependencies, so it would be able to work on all versions of Linux, or all versions of Windows for example.

### Disadvantage <a id="disadvantage"></a>

However, due to the functions living inside the binary, this increases the size of the executable, so it's not the best for disk-space usage.

Another disadvantage is that, contrary to the second advantage, if there's a change to the library, the program won't be able to use that feature, unless the binary is recompiled with the extra function in the library.

Similarly, if a bug is in the library, then you'll need to recompile the binary with the bug fix to keep your code up to date and bug-free, whether that be from a vulnerability point of view or a dependency error.

## Dynamically linked libraries <a id="dynamically-linked-libraries"></a>

_Dynamically linking binaries_ are ones where they contain a small statically linked function which is calls once the program starts. This static function will map the link library into memory and run the code that the functions contained. The link library determines where all the dynamic libraries are located, then mapping the libraries into the middle of the virtual memory and resolving the references to the symbols contained in those libraries.

### Working with the GOT <a id="working-with-the-got"></a>

When a dynamically linked binary calls a function inside a linkable library \(say `puts()` from `libc`\), the call actually points to the PLT \(located, funnily enough, in the `.plt` section\). This will then, due to how the [GOT and PLT](https://euanb26.gitbook.io/resources/cybersec/binary-exploitation/theory/got-and-plt) work, point to the GOT, which will then point to the shared library function \(the actual address of `puts()`\)

### Advantages <a id="advantages"></a>

Due to not storing the functions within the program, the disk-space usage isn't as large. A Dynamically Linked Library \(DLL\) is loaded into memory only once, in which multiple programs can then access the DLL \(these programs can also be written in different languages and still access the same DLL\)

### Disadvantage <a id="disadvantage-1"></a>

If the DLL isn't present, then the program will terminate and provide an error message, saying that it can't find the DLL

This is also the case if the library is updated, and it doesn't provide backwards compatibility. So if the programmer can't upgrade their program to allow for this update on the library, and can't find a DLL that'll still fit the requirements for their program, then that program won't be able to run.

## Relocations <a id="relocations"></a>

When a compiler is looking at where the addresses are for functions in libraries, it doesn't know where they are. A method calling _relocation_ was introduced - the hard work of doing the lookup at runtime is performed by the linker \(`ld-linux.so`\), which is run before any code from the program is run \(this is done by the kernel, so the user doesn't need to worry about it\)

{% hint style="info" %}
Note: every dynamically linked program will be linked against the linker, which is set in an ELF section called the `.interp`
{% endhint %}

