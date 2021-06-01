# NX + page tables

So how can we defend against shellcode? This is where NX comes in \(or as it's shown in the presentation: W XOR X\)

## What is it? <a id="what-is-it"></a>

In the old days, a page table that was marked `writable` implied `executable`. Attackers saw this, and thought "hey, let's try and abuse this to get a shell". The defenders saw that this was a new attack vector, and so had to mitigate it somehow. So, they changed to implication of a page being `writable AND executable` to EITHER `writable` or `executable`. This would put a stop to shellcode, as:

* The attacker would be able to write to a page, however, won't be able to execute anything there
* A page would be executable, however, it couldn't be written to

If an attacker _were_ to try and execute a ret2shellcode attack, and tried jumping to a no-execute page, then the hardware would throw a fit and would generate an exception. This would result in the shellcode failing to execute.

A no-execute page can consist of any of the pages mapped to the binary, including:

* Stack
* Heap
* Libc: a file which holds all the functions that are used in a C program, such as `strcpy`
* Ld: a library that links other libraries

Using NX also has some nice properties:

* You don't have to modify the application
* Protections are implemented by the hardware \(Memory Mapping Unit\), so it's fast

As it is a very important mitigation, it's becoming standard on popular chips, such as the x86 range, AMD and ARM.

## Page tables for NX bits <a id="page-tables-for-nx-bits"></a>

![Page tables for x86-64](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MGPT_9Mfw1vUlMDkAdR%2F-MGPXyF-21w6R3821cdM%2Fnx_explanation.PNG?alt=media&token=b5548d85-9bcd-4f6a-ad9b-cfc67991e1ad)

This is what the page tables look like for an x86-64 application.

Over on the left, we have the `%cr3` register, which is a special control register which can be thought of as a page table pointer. So whenever the OS does a context switch to a new process, the OS is going to set the `cr3` register to point to the page table for the current address space.

The `cr3` register points to the `Page map level 4 (Entry)`, which also takes some bits from the virtual address \(L4 at the top\) to select a specific entry in the `PML4E`. The entry will then point to the `Page directory pointer (Entry)` which will again get some bits from the virtual address and get a specific entry in the `PDPTE`. This will work it's way up all the way until the virtual address has been mapped to a physical address.

If we take a look at what one of the entries in the `PTE` entries are, we can see quite a couple things which are labelled.

![PTE entry with no NX bit](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MGPT_9Mfw1vUlMDkAdR%2F-MGPbDtGKcewPEHaxhO5%2FPTE_entry.PNG?alt=media&token=d19d7d69-0136-48f4-adda-f8a8c08cce9b)

This is what it'll look like when there's no NX bit. The numbers at the bottom correlate to the numbers at the top of the previous image, the virtual address offsets. Now let's see what it'll look like when there is an NX bit set.

![PTE entry with NX bit](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MGPT_9Mfw1vUlMDkAdR%2F-MGPc6T1b7Di9FORdtGA%2FPTE_entry_nx_enabled.PNG?alt=media&token=abea47ae-9c12-423a-b192-f69ef0ffe8b5)

So we can see here that an NX bit has been added to the 63 bit in the entry

