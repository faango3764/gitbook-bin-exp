# Why move from 32 to 64 bit?

At a first glance, it'll seem like moving from 32 bit to 64 bit, in terms of ASLR, the benefit is the number of bits that we can randomise objects with \(as we saw in [ASLR ](https://tango37645.gitbook.io/binexp/bin-exp/defences/aslr)we get 11 bits of randomisation in 32-bit and 22 bits in 64-bit\). However, there are other important reasons that have a massive benefit, not just for [ASLR](aslr.md), like the Stack Smashing Protector, or [Canary](stack-canaries.md).

## ABI <a id="abi"></a>

One of these is the Application Binary Interface \(ABI\), which is the interface between two program modules. Often, one of these modules is a library or OS facility, and the other being a program run by the user. The ABI is also used to define how data structures are accessed in machine code.

If we note back in [System V Calling Conventions](https://tango37645.gitbook.io/binexp/bin-exp/theory/system-v-calling-conventions) , we have 32-bit functions passing their parameters on the stack, \(useful when bypassing ASLR and NX to perform a [ret2libc](../attacks/stack/ret2libc.md) attack\)

Whereas, x86-64 passes the parameters to the function via the registers, which makes it harder for attackers to do a [ret2libc](../attacks/stack/ret2libc.md) attack, as the library functions are expecting the parameters through the registers, not through the stack.

_Note that this is an ABI change. However, it is not related to the increase of addresses that can be used, but related to the fact that there are twice the number of registers \(introduced the_ _`r`_ _family instead of the 32-bit version of_ _`e`\)_

Having a different Application Binary Interface completely changes the attack vector, as we can see in the table below.

| Parameter | Linux 32-bit \(i386\) | Linux 64-bit \(x86-64\) |
| :--- | :--- | :--- |
| ASLR Entropy | Very low \(8 bits\) | High \(28 bits\) |
| ABI/Call parameters | Stack | Registers |
| Direct Attack To ret2libc | Yes | No |
| Offset2lib | Partial | Partial |
| Brute Force In Practise | Yes | No |
| Native PIE CPU support | Yes | Yes \(`rip`\) |

## IP relative address <a id="ip-relative-address"></a>

There is another improvement in the 64-bit executables: Instruction Pointer \(IP\) relative addressing. Relative addressing basically means the "base plus offset", where the IP is the base. This addressing mode was present in most processors _but_ the x86 family. It isn't easy \(rather efficient\) to generate [Position Independent Code/Executable \(PIC/PIE\)](pie.md) on a processor without IP relative addressing.

Now, most executables are PIE compiled - it's been more than a decade to make the move from EXEC \(non-pie ELF\) to DYN \(pie/pic ELF\).

The ASLR in 64-bit systems is not only better because it prevents some attacks, but it is faster than the PIE/PIC CPU support. Therefore, when attackers are designing new methods to bypass ASLR on 64-bit binaries, they need to overcome these.

_Note: Although the architecture is 64-bit, the addresses aren't. The actual virtual addresses are only 47 bits, which greatly reduces the number of bits to be randomised by ASLR._

_In 2017, Intel announced a 57-bit virtual address spaces, but there's no processor in the market yet. Other processors, like the IBM s360, has a real full 64-bit addressing._

