# SGX

## Overview <a id="overview"></a>

* Goal 1: allow a client to interact with a secure-side computation
  * We assume that this is a data-centre scenario
  * We assume that the OS and data-centre operator are untrusted with respect to the client that wants to interact with some server-side computation
  * We want to prevent both the OS on the data-centre machine, and the data-centre operator themselves, from reading or tampering with the state of the server-side computation without being detected
* Goal 2: Allow the secure computation to execute atop the highly-optimsed circuits which execute traditional computations
  * The architects at Intel took a lot of time to make the microarchitecture fast, so ideally we would like this secure server-side computation to run the greatest extent possible on those highly-optimised circuits that do things like speculative execution and out-of-order execution etc
  * Hmmm, this raises the problem of side-channels. If we're not going to have a separate chip that's going to execute this secure computation then presumably this means there's going to be some sharing, if only at the microarchitecture level
  * There's also this big challenge of adding a minimal set of SGX hardware components to a complicated legacy microarchitecture?
  * ​

## Enclave <a id="enclave"></a>

So if we look at a 64 bit virtual address space, we have the normal things. However, we're going to add an _enclave_ to the user-mode state which is going to be the secure computation

![](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MHb6U7NeWpnm_jLsYd9%2F-MHbA0P_i99Miq1jFzzt%2Fsgx_enclave.png?alt=media&token=f37cd72d-c90c-416e-be1b-e38a36fbcfba)

Enclave

In the enclave, we're going to have:

* stack
* heap
* static data
* code
* Entry table
  * help to coordinate how external code jumps or invokes enclave functionality

As the diagram suggests, there's going to be this untrusted host process which embeds an enclave. When code in the untrusted process is executing \(in the user-mode state\), it can't access the enclave pages. If you try to read, you're going to get the value of `-1`, regardless of the state of the enclave pages. If you try to write to the enclave pages, the hardware just drops those writes.

The untrusted host interacts with the enclave code using an `EENTER` instruction, which basically says "hey, I'm the untrusted host, I want to start enclave code."

The enclave code is going to run at ring-3, the least-privileged level. However, the enclave code can access the entire user-mode address space of the host. This is helpful, as it turns out, the enclave code can't issue system calls instructions, such as `syscall` or `int`. If the enclave code wants to issue I/Os, it has to rely on the untrusted host.

For example, if the if the enclave wants to write data to the disk or network, it has to place that data somewhere in the memory that's accessible to the untrusted host, and hope that the untrusted hist will perform the write. Similarly, if the enclave wants to read data in, then it has to put that data request in somewhere in the memory accessible to the untrusted host, then ask the untrusted host to issue the I/O, then read that result in from the untrusted memory.

The enclave can return the control of the CPU back to the untrusted host by executing this new `EEXIT` instruction.

## Microarchitecture <a id="microarchitecture"></a>

![](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MHb6U7NeWpnm_jLsYd9%2F-MHef5fuO4ktqKv0tbse%2Fsgx_microarch.png?alt=media&token=11bc9a92-de3b-4eeb-bbb1-e53a0c80b6bb)

CPU microarchitecture

So when we look at normal microarchitecture, we see that we've got a core alongside some L1, L2 and L3 cache. We've got the page table register \(`%cr3` which we looked at in [NX + page tables](nx-and-page-tables.md)\). It's going to have a bunch of Translation Lookaside Buffers \(TLBs\) that control the mappings from virtual mappings to physical ones.

We're then going to add this special `isEnclave` bit. This can be thought of as being similar to the logical `isPrivelleged` bit that the CPU has, which is set for whether the code is running in the kernel-mode or in user-mode. So therefore, we can say that the `isEnclave` bit is asking whether were running enclave code or not.

In between a core and it's memory holding and the physical RAM, there's a Memory Encryption Engine. Using this, we can separate physical RAM into two separate regions: normal system memory and enclave page cache \(EPC\), which is where all the enclave state lives.

What the MEE is going to do is, when enclave code is running, and an enclave own cache line has to be evicted from the L3, it's going to transparently encrypt that thing and add some max counters, before sending that memory write to the physical RAM. This means that for all of the EPC state, all the enclave page state, it's all encrypted. Even if you were to use EE tricks and try and read the RAM directly, it's all encrypted by the Memory Encryption Engine, before it even hits the bus which goes to physical RAM. So it's safe against physical tampering.

When enclave code is executing, and it needs to bring in enclave data from physical RAM, from L1 through to L3 cache, then as that cache line is coming from physical RAM, it's going to go through the Memory Encryption Engine, which is going to decrypt the data and check the counters and the MAC to look for integrity and freshness.

What's not in the diagram is this new hardware base data structure called the Enclave Page Cache Map \(EPCM\). This is an SGX structure that's going to contain 1 entry for each page in the EPC. The EPCM can only be modified by SGX instructions, which means that the EPC is going to contain a lot of important metadata like which particular EPC belongs to the EPCM. The EPCM is also consulted during memory accesses to prevent EPC pages from being read or written to by people who shouldn't be able to write there.

What bits do we have in the Enclave Page Cache Map?

```c
+---+--------+---+---------+-----+-----+
| 1 |   2    |   |    3    |  4  |  5  |
+---+--------+---+---------+-----+-----+
```

1.  Is the enclave page code ID valid?
2. Which enclave owns this page?
3. What is the base virtual address that the page should be mapped to in the untrusted host?
4. Read/ Write/ eXecute perms
5. Is the EPC blocked?

## Building an enclave <a id="building-an-enclave"></a>

In normal system data in the RAM, there's the plain text code and data for the enclaves initial state - this'll be sitting somewhere in regular system memory, before `EECREATE` happens. This is an instruction which starts building up an enclave.

_Note: all these instructions are executed by the OS. All the instructions that set up the enclave are privileged instructions that only the kernel can execute. It is true that the OS can launch Denial of Service by refusing to load an enclave, or loading it improperly_ \(`EADD` _was called too many times etc._\). _However, that's fine, as the client can detect this remote attestation._

_Note: an enclave can be multi-threaded_

What the OS will do is it'll call `ECREATE`once. It'll generate a new SGX page \(basically a metadata page for the enclave which'll contain book-keeping information\).

The OS is going to do an `EADD` instruction. This'll take the page of code or data from normal system memory, copy it it into an EPC page - called the _reg page_ \(regular page\). This'll be used to start initialising the code data for the enclave.

After, an `EEXTEND` will be called. This'll hash some of the data that was added into the EPC. As it turns out, `EEXTEND` only hashes 256 bytes of data. If you have a 4kb page, then the OS has to call `EEXTEND` 16 times to update this cumulative patch. Why is this useful? It''s helpful because of remote access station. Essentially, at a high level, once an enclave is up and running, an remote client might want to verify the identity of the remote enclave before sharing sensitive data. What the client can do is ask the SGX hardware, which is assumed to be trusted, to generate a signed message, which'll say "here is the cumulative hash of all of the data that was used to initialise the enclave, including both static data and the code".

You may ask why the `EEXTEND` only covers 256 bytes instead of a full 4kb page? \(hashing the page 16 times\). Well, that's because hashing an entire page would take longer. This is irritating, as if you have a long-running instruction, you have to disable interrupts, which seems bad because it means the system becomes less response to external stimuli, such as I/O events and timers. Or, you could allow interrupts, but fail the instructions with an error code. However, that's a pain if the instruction is fairly complicated, such as hashing. What SGX does, is it makes the `EEXTEND` only cover 256 bytes, then the kernel has to invoke multiple times to cover the full 4kb page.

After another `EADD` and `EEXTEND`, the Enclave Page Cache Map is updated - more and more entries are being added to setup the enclave.

Eventually, we create a Thread Control Structure \(TCS\) page. If we look back up at the notes, an enclave can be multi-threaded. So this TCP page keeps track of how many threads that are active and where they all are in terms of their execution status.

Finally, the kernel calls the `EINIT` and the enclave is finalised. There can't be any additional updates to the enclave at this point.

## Enter into an enclave <a id="enter-into-an-enclave"></a>

_Only callable by Ring 3 code and you can't be in the enclave already_

If you want to enter into an enclave, you can use this `EENTER` instruction, which'll takes in the address of the thread control structure that you want to jump into.

* It'll check the TCS to make sure that the untrusted host isn't executing this thread.
*  It'll then flush the TLB entry, as we don't want the malicious OS to somehow fill up the TLB with mappings that aren't correct for the enclave.
* We're then going to flip the `isEnclave` bit to `1`
* Now executing the enclave, we're going to save `rip`, `rsp`, `rbp` to a scratch area inside the enclave's address space \(useful for returning out of the enclave\).
* Then it's going to save the Asynchronous Exit Pointer \(AEP\) \(which points to a location in the untrusted host\) to scratch area inside the enclave's address space
* Then the address of the instruction after `EENTER` is going to be written to `rcx` which'll be saved by the enclave code \(again, useful for exiting the enclave\)
* Jump to the enclave start address described by the Thread Control Structure

## Exit an enclave <a id="exit-an-enclave"></a>

_Only called by ring 3 when inside the enclave_

The kernel is going to execute the `EEXIT` instruction which has a parameter of the `jmpTarget`

* Flip the `isENclave` bit to `0`
* Flush the TLB entries - don't want to reveal anything about the mapping address entries
* Clear the `TCS.isBusy` bit to say that the thread isn't used
* Jump to the `jmpTarget` in the untrusted host; should be saved buy `EENTER`.

{% hint style="info" %}
Note: none of these instructions change the `cr3` register.
{% endhint %}

## _Memory access check_ <a id="memory-access-check"></a>

SGX is going to add extra hardware to the memory controller, to ensure that the EPC pages can only be accessed by enclave code, in particular, the enclave that is associated with each EPC page. The extra hardware is going to rely on the EPCM

![](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MHf0aHVuj0YGU8c-p9-%2F-MHf6Id1HfCLG1klsE6C%2Fmem_access_checks_sgx.png?alt=media&token=dd5abbfc-8daf-473b-9b16-713e3336f673)

Memory access checks in SGX

This is basically showing how virtual address is translated and checked to see if it's an enclave page.

## Hyperthreading side-channel  <a id="hyperthreading-side-channel"></a>

Before we get into this **attack**, we need to deep dive into what a processor architecture looks like

![](https://gblobscdn.gitbook.com/assets%2F-MGOhxJbNhi10jg9Cv-U%2F-MHf0aHVuj0YGU8c-p9-%2F-MHf7FCrgXxgqCprQofX%2Fhyperthreaded_processor.png?alt=media&token=d94dcfc5-5a2a-4afd-ae54-3a5e7d8bff63)

What a hyperthreaded processor looks like

The basic idea is that we have software that has two separate pipelines, even though there's a single physical core - two logical cores, one physical core. Why does the software think there are two pipelines? This is because we have two sets of registers that are exposed by this single physical pipelines. So we have two sets of registers; two sets of register state that can be scheduled by the OS.

In this case, we have a blue thread that is operating on the top logical thread, and yellow being the other thread.

So the instructions get queued up, they go through register rename, holding the rewritten instructions in the queue. We then get to instruction scheduling, and what we see the core of pipelining. This part is actually agnostic as to which logical core instructions come from. What this out-of-order execution core sees is dependencies between various instructions and various registers and various memory values. There isn't any notion of the blue instruction coming from the blue registers or a yellow instruction coming from a yellow register. At the end, where we retire stuff, we have to care about the notion of these logical cores. We see the reorder buffer, where the stuff in the middle is out-of-order execution, but to enable precise interrupts, we have to retire each one of the instructions through each logical core in logical orders

Now imagine that the blue code belongs to enclave code and that the yellow code belongs to the non-enclave code. So what might happen? There's actually contention in the shared core of the pipeline. It's conventional as there are functional units and other resources that are shared between enclave code and non-enclave code.

So as a trivial example, suppose that the non-enclave code wanted to determine whether the enclave was executing integer instructions or not. What the non-enclave code could do is time how long the integer instructions take. Whenever the non-enclave code sees the completion time of those instructions go up, for example, if it reads the time stamp counter, then the non-enclave code can infer that the enclave code is using integer functions.

So that's a side channel, and that can leak a lot of information. For example, if the enclave code was performing cryptography, there's been a lot of work that shows that if the non-enclave code knows the crypto library the enclave is using, then based on contention for functional units, the non-enclave code can guess what keys are being manipulated for example.

## Cache based side-channel <a id="cache-based-side-channel"></a>

If you would like to know a bit more about this, you can check out this paper:

​[https://www.usenix.org/system/files/conference/usenixsecurity18/sec18-van\_bulck.pdf](https://www.usenix.org/system/files/conference/usenixsecurity18/sec18-van_bulck.pdf)​

At a high level, the enclave and the untrusted host share the same L1 cache. Why is that? Because they're in the same address space. They're on the same core, so they share the same L1 cache.

The untrusted host can load the enclave and evict an oracle array's cache lines. An oracle array is essentially an array that is living in the untrusted host. The host is going to evict all the cache lines that belong to the array, such as using the instruction `CLFLUSH` \(cache line flush\) and then load the enclave.

The untrusted host process is going to wait for the enclave to bring a clear text secret value into the L1 cache, and the untrusted host is going to do, at a high level:

* `unint8_t v = *secret_ptr; // dereferences the pointer`
  * _Note: the_ `secret_ptr` _is pointing into enclave memory - memory that the untrusted host cannot read or write_
* `unint64_t o = oracle[v*4096]; // try to read the oracle array based on v`
  * Going to attempt to see which oracle addresses are being read

Remember in the first step, the untrusted host has evicted all the cache lines belonging to the oracle array. But this isn't going to work, due to SGX? Well, that's kind of true. This is going to fail at the Instruction Set Architecture \(ISA\) level \(i.e. if `v` is kept in a register, then `*secret_ptr` is going to be `-1` or the dummy value\), however, at the microarchitecture level, there's going to be out-of-order execution and so there can be a speculative load of this secret pointer value. The CPU can speculatively execute that load, bring a value into `v`, and then try and read the relevant lines from oracle into cache. This can all happen speculatively as the processor in parallel is doing page table protection, and whether this load should actually be allowed to take place.

This means that, even at the ISA level read, the value `v` will fail and get `-1`, depending on how some of these microarchitecture race conditions resolve themselves, it could be that the speculative code gets as far as executing the `oracle[v*4096]` read, bringing in a particular cache line in the oracle array where that particular line is chosen based on this value, `v`. It's never revealed at the ISA level, but it may have side effects at the microarchitecture level, via the cache in particular.

The untrusted host can then read how long it takes to read each of the cache lines in the oracle array, in which it can then determine the value of `v`, as remember the host evicted all the oracle cache line. At the very end, the untrusted code reads each of the cache lines. Some of those reads are going to be very slow, in fact, all of them will be very slow, except for the one that's been brought into the L1 cache via the microarchitecture level speculation.

