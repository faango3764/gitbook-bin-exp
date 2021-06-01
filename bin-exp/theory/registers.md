# Registers

Registers play a key part in binary exploitation, as well as running a program. So what are they?

Registers are essentially temporary data storage devices that can hold different sizes of memory addresses on different machines and programs. On 64 bit machines and programs, these are 64 bits.

So what registers do we have?

In general, we have 9 main registers

* ax
  * Holds the return value of a function
* bx
  * General purpose
* cx
  * Genreal purpose
* dx
  * General purpose
* sp
  * Stack pointer - Points to the top of the stack
* bp
  * Base pointer - Points to the bottom of the stack
* ip
  * Instruction pointer - Points to the instruction that will be executed next
* si
  * Source index - general purpose
* di
  * Destination index - general purpose

We can actually get 64 bit version of these registers, as well as 32, as well as 16, and 8 bits. Any register prefixed with `r` is 64 bit \(Standing for register\), any register prefixed with `e` is 32 bit \(standing for extended\), the registers above are 16 bits, and then the lower and higher bits are viewed via `l` and `h` respectively.

Where a visual is better than a paragraph:

```java
rax: ==================================================== (64 bit)
eax:                         ============================ (32 bit)
ax:                                        ============== (16 bit)
al:                                                ====== (8 bits (lower))
ah:                                        ======         (8 bits (upper))
```

​

