# Stack Pivoting

## What is a stack frame? <a id="what-is-a-stack-frame"></a>

We have "the stack", which is in the userland and kernel code, and it gets linked into a binary when it's run. It's real name is _the call stack_, also known as _the execution stack, program stack, control stack etc._ The call stack stores information about active subroutines and functions, including where a function or subroutine will return to once it finishes.

This call stack is split up into multiple pieces, known as _stack frames_, or _frames._ Each frame is directly related to a subroutine - it holds the arguments for execution of the subroutine, local variables and the address of where the function is executing.

When the binary starts, it has only one frame, that being `main()`. This is known as the _initial frame_ or the _outermost frame_. Each time a function is called, a new frame will be created, and the program control-flow will jump to that address, jumping back to `main()` once it has returned.

## Stack pivoting <a id="stack-pivoting"></a>

We have a code snippet:

```c
void scanInput(int iParm1)​{
  undefined input [256];
  input[0] = 0;
  printf("Ok, what do you have to say for yourself?");
  fgets(0,input,(long)iParm1);
  printf("Interesting thought \"%s\", I\'ll take it into consideration.\n",input);
  return;
}
```

Where it simply scans in our input and stores it into a variable. Our `stack frame` would look something like this

```java
+------------+
| stack data |
|    input   |
+------------+
|  base ptr  |
| instr ptr  |
+------------+
```

Where we have

* `stack data` being the variables that are kept on the stack \(so our `input`\)
* After, you have a `base ptr` for the stack and a `instr ptr` for the instructions

When a `call` instruction is made to call the `scanInput()` function, both the `base ptr` and the `instr ptr` are placed onto the top of the call stack from `main()`, so therefore when `scanInput()` returns, it knows where to return to.

So how can we abuse this?

Well, if we take a look at how this gets executed in the assembly, we see this:

```c
00400c3f e8 2f ff        CALL       scanInput
0400c44  c9              LEAVE
00400c45 c3              RET
```

So as we can see, once the program does the instruction `call scanInput` it'll leave the main function, and return \(back to `_start`as main gets executed through `__libc_start_main`, located in `_start`\). So, what if we could perform a buffer overflow that gets us up to the `LEAVE` instruction. We could then try and _pivot to a new stack frame_ . This'll allow us to either set up a new, fake stack frame, which means that we could execute anything we wanted, such as a ROP chain of `system("/bin/sh")` or `execve("/bin/sh")`. This is known as "stack pivoting" - we're pivoting to a new stack frame.

This is sometimes seen in CTFs when there is what's known as a one-byte overflow. Say that a program got `257` bytes of data, storing it in a buffer that is `256`. This'll allow us to overwrite the last byte of the address, allowing us to pivot anywhere within a `0x00-0xff` byte range, or a 256 bytes range.

