# Format strings

This exploit is caused when a function that requires format strings, isn't properly formatted. This is the case when either these functions are properly formatted and attacker controlled:

* `fprintf(FILE *stream, const char *format);` writes the printf to a file
* `printf(const char *format);` outputs a formatted string
* `sprintf(char *str, const char *format);` prints into a string
* `snprintf(char *str, size_t size, const char *format);` prints into a string, checking length
* `vfprintf(FILE *stream, const char *format, va_list arg);` prints the va\_arg structure to a file
* `vsprintf(char *str, const char *format, va_list arg);` prints the va\_arg to a string
* `vnsprintf(const char *str, size_t size, const char *format, va_list arg);` prints the va\_arg to a string checking its length

What this means is that the program doesn't properly validate user input.

## Format parameter? <a id="format-parameter"></a>

These are ways in which data can be outputted in a specific format. This format may be either a string, an integer, a pointer etc. The full list of all format parameters can be found here: [https://codeforwin.org/2015/05/list-of-all-format-specifiers-in-c-programming.html](https://codeforwin.org/2015/05/list-of-all-format-specifiers-in-c-programming.html) For format string attacks, these are the common ones to try:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Format specifier</th>
      <th style="text-align:left">Output</th>
      <th style="text-align:left">Passed as</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>%%</code>
      </td>
      <td style="text-align:left">% Character (literal)</td>
      <td style="text-align:left">Reference</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>%p</code>
      </td>
      <td style="text-align:left">
        <p>Pointer</p>
        <p>Reads 4 bytes in 32 bit, 8 bytes on 64 bit</p>
      </td>
      <td style="text-align:left">Reference</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>%d</code>
      </td>
      <td style="text-align:left">Decimal</td>
      <td style="text-align:left">Value</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>%c</code>
      </td>
      <td style="text-align:left">Character</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><code>%u</code>
      </td>
      <td style="text-align:left">Unsigned decimal</td>
      <td style="text-align:left">Value</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>%x</code>
      </td>
      <td style="text-align:left">Hexadecimal integer (4 bytes on most systems)</td>
      <td style="text-align:left">Value</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>%s</code>
      </td>
      <td style="text-align:left">Reads data as a pointer to a string</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><code>%n</code>
      </td>
      <td style="text-align:left">Writes a number of characters to a pointer</td>
      <td style="text-align:left">Reference</td>
    </tr>
  </tbody>
</table>

### Example <a id="example"></a>

```c
#include <stdio.h>

​void main(int argc, char **argv){
    printf("%s\n", argv[1]);
    printf(argv[1]);
    printf("Exit");
}
```

So as well can see, we have unsafe and safe code. The safe code is properly formatted, and we can't take a format string vulnerability to it. However, in the second example, it just prints our first argument \(`argv[0]` is the filename\) without any formatting. This is susceptible to a format strings attack.

If we compile the program, letting it be 32 bit, with `gcc -m32 -o helloWorld example.c` and then we can execute the program doing `./helloWorld`. We can specify anything we want to the. We could specify something like

```c
$ ./helloWorld "Hello, World!"
Hello, World!
Hello, World!
Exit
```

Which would work nicely. But what if we specified a format string as our argument? What happens?

```c
$ ./helloWorld "%p:%p:%p"
%p:%p:%p
0xf7fa580:0x565ba000:0x256
Exit
```

What happened there? Our first `printf()` was properly formatted, so it did nothing. But in our second `printf()`, it gave us a hexadecimal number, more specifically a pointer to an address somewhere in memory. How did it do this?

Remember that we're on 32 bit. We can take a look at how 32 bit and 64 bit binaries get arguments here: System V Calling Conventions . As a reminder, 32 bit gets arguments off of the stack, whereas 64 bit gets arguments from registers.

If we look back at the arguments that `printf()` takes, we see

```c
printf(const char *format);
```

So the first argument that the CPU expects is this formatted parameter. If nothing is specified after the format string, then it'll just get the top value from the stack, and put that as its argument. This allows us to _leak_ addresses, which then allows us to do a ton of things, such as Defeating ASLR and PIE or Defeating Stack Canaries , leading to a Ret2libc attack. We can also do a lot more things with this.

When we're attacker a binary, we might need to leak multiple addresses to form an attack. But what if we need to provide multiple format strings, say 100, to get our value? We can take a shortcut with this, using the format of `%x$p` where `x` is an integer and `p` is a format specifier.

Let's use our previous example:

```c
$ ./helloWorld "%p:%p:%p"
%p:%p:%p
0xf7fa580:0x565ba000:0x256
Exit

​$ ./helloWorld "%3$p"
%3$p
0x256
Exit
```

So as we can see, we've leaked the 3rd value from the stack. This is just a shorthand of writing `%p%p%p`and is useful when fuzzing values, which can be seen with this fuzzer tool that I've written.

Did you know that web applications may be susceptible to this? This website explains it nicely: [https://www.netsparker.com/blog/web-security/string-concatenation-format-string-vulnerabilities/](https://www.netsparker.com/blog/web-security/string-concatenation-format-string-vulnerabilities/)​

