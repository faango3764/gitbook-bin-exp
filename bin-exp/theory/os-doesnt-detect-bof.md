# Why doesn't the OS detect BOF?

We've always been taught that the OS is this paternalistic character; it's always checking if someone has tampered with it's state, and if so it'll punish them.

A common question when learning binary exploitation is this question.

{% hint style="warning" %}
Why doesn't the OS detect buffer overflows?
{% endhint %}

Roughly speaking, the kernel only executes code when:

* User-level code issues a system call
* An external event occurs \(eg. a timer interrupt fires, a storage device indicates an IO read has been completed\).

We can say here, roughly, that the kernel is mostly passive and it relies on page tables and the MMU \(the hardware which performs the translation from virtual memory addresses to physical addresses\) to isolate one process from another. However, the kernel isn't actively inspecting everything that a process does.

What this means is that these attacks can still occur, such as buffer overflows. The page-table-based isolation doesn't protect a process from itself. Therefore, the OS can't do anything special with the process, as the kernel is just sitting around, waiting for something to happen.

