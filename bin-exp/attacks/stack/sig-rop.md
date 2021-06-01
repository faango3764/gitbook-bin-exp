# \(SIG\)ROP

Let's first talk about what ROP is

## ROP <a id="rop"></a>

Return Orientated Programming \(ROP\) is essentially chaining together parts of the program, in assembly, to allow us to gain a shell from the binary, or do other malicious things.

The reason that we use ROP is because most binaries, when out in the wild or even in CTFs, don't have a "win" function. Therefore, we have to create our own.

When we've managed to pwn the binary and are wondering where to go next, we need to call `execve` \(or it's family, that is `execl`,`execle`, `execlp`,`execlpe`,`execv`, `execve`, `execvp`, more can be at [https://www.geeksforgeeks.org/exec-family-of-functions-in-c/](https://www.geeksforgeeks.org/exec-family-of-functions-in-c/)\) or `system`. But we're going to need some arguments ... one common one is the string `/bin/sh` as that's what our terminals run off of - using the programming language `bash` which is spawn in via `/bin/sh`.

### x86 <a id="x86"></a>

Sow how would we provide our arguments to the function? Well the function will expect it to be called via this:

* &lt;function&gt;
* &lt;exit function / junk&gt;
* &lt;argument&gt;

This is just how the binary expects the arguments to be, with the stack layout being something along the lines of \(once the function has been called\):

```c
| return address |
|    arguments   |
```

This way, when `main()` returns, it will call `system` PLT entry, and the stack will appear as though `system` has been called normally.

_Note: we don't care where our return address points to, as we have our shell. It's there if we need to cleanly exit the program - where do we jump to if we fail._

### x86-64 <a id="x-86-64"></a>

Here, we have to work a bit harder - as we recall from the System V Calling Conventions , 64 bit machines take their arguments from the registers. When we pwn the binary, we take control of `rip`. But now, we also need to take control of `rdi`.

In this case, we need to take small parts of the binary, known as `gadgets`. These usually include a `pop` a value, or a couple, into register\(s\), and then a `ret` instruction. This allows us to chain together multiple commands to get our malicious code to execute, essentially setting up a fake stack.

```c
0x401e49    pop rdi; ret
0x401e50    pop rsi; pop r9; xor rax; ret 
```

## SIGROP <a id="sigrop"></a>

When we overflow the buffer and are able to execute a `syscall` instruction, or an `int 80` instruction, you can execute a _signal_ which'll allow you to do something \(a list of signals can be found here: [https://man7.org/linux/man-pages/man7/signal.7.html](https://man7.org/linux/man-pages/man7/signal.7.html)\). An example of a signal is a SIGSEGV for an invalid memory access, or a SIGINT as an interrupt from the keyboard. Another example of a signal is the SIGRET signal, which takes the current stack frame and writes it to the registers. So if we're able to take control of the current frame and what arguments get popped into the respective registers, we can take control of the control-flow.

### Registers that have values popped <a id="registers-that-have-values-popped"></a>

So which values get popped into which registers? The _sigcontext_ structure, which is stored on the stack, is used by `sigreturn` to pop the values into the stack \(for 64 bit binaries\)

```c
+--------------------+--------------------+
| rt_sigeturn()      | uc_flags           |
+--------------------+--------------------+
| &uc                | uc_stack.ss_sp     |
+--------------------+--------------------+
| uc_stack.ss_flags  | uc.stack.ss_size   |
+--------------------+--------------------+
| r8                 | r9                 |
+--------------------+--------------------+
| r10                | r11                |
+--------------------+--------------------+
| r12                | r13                |
+--------------------+--------------------+
| r14                | r15                |
+--------------------+--------------------+
| rdi                | rsi                |
+--------------------+--------------------+
| rbp                | rbx                |
+--------------------+--------------------+
| rdx                | rax                |
+--------------------+--------------------+
| rcx                | rsp                |
+--------------------+--------------------+
| rip                | eflags             |
+--------------------+--------------------+
| cs / gs / fs       | err                |
+--------------------+--------------------+
| trapno             | oldmask (unused)   |
+--------------------+--------------------+
| cr2 (segfault addr)| &fpstate           |
+--------------------+--------------------+
| __reserved         | sigmask            |
+--------------------+--------------------+
```

### Which registers to set? <a id="which-registers-to-set"></a>

Fortunately, pwntools has a nice feature called `SigreturnFrame()` which allows us to set registers to specific values. We could

```c
RIP:    0x1000000b (address of a syscall)
RAX:    0x1 (specify write syscall)
RDI:    0x1 (specify stdout to output flag)
RSI:    0x10000023 (address of the flag)
RDX:    0x400 (amount of bytes to print)
```

A whitepaper on SIGROP can be found here: [https://www.semanticscholar.org/paper/Sigreturn-oriented-programming-is-a-real-threat-Mabon/5c483c22bf9a761d6b900b6acdbad72b321f39ee?p2df](https://www.semanticscholar.org/paper/Sigreturn-oriented-programming-is-a-real-threat-Mabon/5c483c22bf9a761d6b900b6acdbad72b321f39ee?p2df)​

