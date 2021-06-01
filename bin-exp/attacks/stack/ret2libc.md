# Ret2libc

Let's decide to turn on the NX protection. How can we defeat this?

If we look back at our BOF example here, and we still have control over the `str` pointer

```c
void copy(char *str) {
    char buffer [16];
    strcpy(buffer, str);
}
```

![](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MGPEw3CJxvZDRBg_NWA%2F-MGPKt3539vxTLdzF5k0%2Fbof.PNG?alt=media&token=546e8087-34ed-4fa3-9ae7-41b8e1c4b843)

## How could we attack this? <a id="how-could-we-attack-this"></a>

Well, remember when I talked about the libc page, and what the libc holds? That's right, it holds all the functions that a C program can access, such as `strcpy()`, `gets()`, `__libc_start_main()`, `system()`, `exit()`, `malloc()`, `...`. So, what if we could call one of these functions with any parameters we want? As the libc functions live on the stack, we can find where they are, and jump to their address, providing them with their arguments.

For example, a common ret2libc attack would be to call `system("/bin/sh")`. This would emulate the shellcode, in the sense that we would be getting system to execute /bin/sh, which you guessed it, is how a bash shell gets started.

So, this is all based on the attacker being able to control the flow of execution. We overflow, then we overwrite the original return address with one of our own - this is known as _Code Pointer Corruption_

## Code Pointer Corruption <a id="code-pointer-corruption"></a>

As we said, a buffer overflow allows an attacker to divert the control flow of the program to somewhere of their choosing. So what options do they have?

* Stack
  * Return values
  * Function pointers
  * Vptrs
* Heap
  * Function pointers
  * Vptrs

