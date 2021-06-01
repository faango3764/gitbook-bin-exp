# History

This is a long page about everything that has happened with memory corruption vulnerabilities. This page is true up to 18/09/2020

## 1972 <a id="1972"></a>

### 31/09 - First overflow <a id="31-09-first-overflow"></a>

#### The first documented overflow attack <a id="the-first-documented-overflow-attack"></a>

The Computer Security Technology Planning study compiled a paper to present computer security requirements for the US air force.

While discussing a program that handled pointers, it reads:

​[http://seclab.cs.ucdavis.edu/projects/history/papers/ande72a.pdf](http://seclab.cs.ucdavis.edu/projects/history/papers/ande72a.pdf)​

> “By supplying addresses outside the space allocated to the users programs, it is often possible to get the monitor to obtain unauthorized data for that user, or at the very least, generate a set of conditions in the monitor that causes a system crash.”

It goes on to describe another operating system where

> “the code performing this function does not check the source and destination addresses properly, permitting portions of the monitor to be overlaid by the user. This can be used to inject code into the monitor that will permit the user to seize control of the machine.”

## 1985 <a id="1985"></a>

### 17/11 <a id="17-11"></a>

#### Phrack magazine 0x01 published <a id="phrack-magazine-0x01-published"></a>

## 1988 <a id="1988"></a>

### 02/11 - Morris Worm <a id="02-11-morris-worm"></a>

#### The Morris Worm <a id="the-morris-worm"></a>

Robert Tappan Morris \(Jr.\) wrote and released the "Morris Worm" whilst being a student at Cornell University. This was the first computer worm to be distributed via the internet, which one of its attack vectors was a smash the stack against the fingerd daemon. In Eugene Spafford's analysis of the worm

> “The bug exploited to break fingerd involved overrunning the buffer the daemon used for input. The standard C library has a few routines that read input without checking for bounds on the buffer involved. In particular, the **gets** call takes input to a buffer without doing any bounds checking; this was the call exploited by the Worm.”

The fingerd program has a 512 byte buffer for `gets()`. The Morris Worm crafted an exploit of 536 bytes, which overwrote the return frame of the stack, now pointing to shellcode that would execute `execve("/bin/sh",0,0)`

### 30/11 - CERT founded <a id="30-11-cert-founded"></a>

#### CERT founded <a id="cert-founded"></a>

## 1989 <a id="1989"></a>

### 23/01 <a id="23-01"></a>

#### "ZARDOZ Security Digest" published <a id="zardoz-security-digest-published"></a>

### 31/01 <a id="31-01"></a>

#### CERT advisory for overflow in BSD v4.3 bin/password.c <a id="cert-advisory-for-overflow-in-bsd-v-4-3-bin-password-c"></a>

CERT published CA-1989-01 to document an overflow in `password.c` reported by Keith Bostic of UC-Berkley

## 1990 <a id="1990"></a>

### 23/06 <a id="23-06"></a>

#### ZARDOZ becomes the Core Sec Mail List <a id="zardoz-becomes-the-core-sec-mail-list"></a>

### 30/12 <a id="30-12"></a>

#### An empirical study of the reliability of UNIX utilities <a id="an-empirical-study-of-the-reliability-of-unix-utilities"></a>

Barton Miller published "An empirical study of the reliability of the UNIX utilities in the ACM". With relatively simple \(compared to today's\) fuzzing, they were

> "able to crash 25-33% of the utility programs on any version of UNIX that was tested"

## 1993 <a id="1993"></a>

### 05/09 <a id="05-09"></a>

#### BUGTRAQ formed <a id="bugtraq-formed"></a>

### 30/09 <a id="30-09"></a>

#### ISS Scanner released <a id="iss-scanner-released"></a>

## 1995 <a id="1995"></a>

### 13/02 <a id="13-02"></a>

#### Overflow in NCSA httpd <a id="overflow-in-ncsa-httpd"></a>

Thomas Lopatic made a posting to Bugtraq to report an overflow vulnerability in NCSA httpd v1.3. He showed the steps to reproduce to exploit and included an exploit which creates a file called "GOTCHA" in the `/tmp` directory

### 04/03 <a id="04-03"></a>

#### SATAN released <a id="satan-released"></a>

### 19/10 <a id="19-10"></a>

#### CERT publishes vulnerability in syslogd <a id="cert-publishes-vulnerability-in-syslogd"></a>

### 20/10 <a id="20-10"></a>

#### "How to write buffer overflows" <a id="how-to-write-buffer-overflows"></a>

This was a document which was mainly for Peter Zatko \(Mudge\) for his own notes. It covers an introduction into the basics of overflows and writing shellcode. It included the syslogd bug which was founded the day before.

### 03/12 <a id="03-12"></a>

#### Splitvt exploit published <a id="splitvt-exploit-published"></a>

DaveG and VicM of Avalon Research published an advisory \(and exploit\) for splitvt on Linux 2-3.x. This was due to an unbounded `sprintf` call, which was exploited by and over long HOME environment variable.

## 1996 <a id="1996"></a>

### 08/11 - smashing the stack <a id="08-11-smashing-the-stack"></a>

#### Smashing the stack published <a id="smashing-the-stack-published"></a>

Elias Levy \(Aleph1\) published what would become the mot referenced paper on memory corruption attacks in Phrack49.

> `smash the stack` \[C programming\] n. On many C implementations it is possible to corrupt the execution stack by writing past the end of an array declared auto in a routine. Code that does this is said to smash the stack, and can cause return from the routine to jump to a random address. This can produce some of the most insidious data-dependent bugs known to mankind.

## 1997 <a id="1997"></a>

### 20/01 <a id="20-01"></a>

#### Stack smashing defenses discussed <a id="stack-smashing-defenses-discussed"></a>

Bugtraq hosts a discussion on defenses against stack smashing

### 21/03 <a id="21-03"></a>

#### Superprobe exploit published <a id="superprobe-exploit-published"></a>

Solar designers overwrite function pointers to hijack execution flow

### 22/04 <a id="22-04"></a>

#### DNS poisoning QID prediction <a id="dns-poisoning-qid-prediction"></a>

CORE and SNI report possible overflow due to bind ignoring

### 12/04 - NX stack <a id="12-04-nx-stack"></a>

#### NX stack <a id="nx-stack"></a>

Alexander Pesylak \(Solar Designer\) posted on Bugtraq a linux patch to defeat the smashing the stack attacks. This included changing the memory permissions of the stack to read and write instead of read, write and execute \(what we know today as NX\)

### 10/08 - ret2libc <a id="10-08-ret-2-libc"></a>

#### Bypassing non-execute stack \(ret2libc\) <a id="bypassing-non-execute-stack-ret-2-libc"></a>

Alexander Pesylak published his first known return-to-libc attack to overcome his own non-executable stack patch.

He demonstrated the technique using the `lpr` exploit against a Linux system with his own non-executable stack patch. His patch ensured that shared libraries are `mmap`ed into regions containing a null byte to reduce their use with unsafe string functions

He also improved his previous non-executable stack patch with a technique called ASCII Armoring. This makes BOFs more difficult because it'll map the shared libraries on memory addresses that contain a null byte, such as `0xb7e39d00`. If `strcpy()` was used, it terminates at a null byte, therefore not all of the address being got from the function.

### 01/09 - nmap <a id="01-09-nmap"></a>

#### Nmap - art of portscanning <a id="nmap-art-of-portscanning"></a>

NMap is released in Phrack 51

### 18/12 - canary <a id="18-12-canary"></a>

#### StackGuard announcement + canary <a id="stackguard-announcement-canary"></a>

Crispin Cowan announced StackGuard on Bugtraq with a link to his pre-released USENIX paper titled "StackGuard: automatic adaptive detection and prevention of buffer-overflow attacks"

​[https://www.usenix.org/legacy/publications/library/proceedings/sec98/full\_papers/cowan/cowan.pdf](https://www.usenix.org/legacy/publications/library/proceedings/sec98/full_papers/cowan/cowan.pdf)​

Their invention was a GCC patch that makes use of a "canary" DWORD on the stack in front of the return address to determine whether the return address has been modified

### 19/12 <a id="19-12"></a>

#### StackGuard bypasses discussed <a id="stackguard-bypasses-discussed"></a>

Tim Newsham posts to Bugtraq with possible weaknesses with StackGuard. He covers attacking local variables and possible leaks of the canary value

## 1998 <a id="1998"></a>

### 14/01 - first heap overflow <a id="14-01-first-heap-overflow"></a>

#### IE4 Heap overflow <a id="ie4-heap-overflow"></a>

Christien Rloux \(DilDog\) published his exploit and advisory for the mk:// exploit on Internet Explorer 4. The bug was a heap based overflow with the exploit code written for Windows 95

### 30/01 <a id="30-01"></a>

Rafal Wojtczuk \(Nergal\) wrote to Bugtraq another way to perform return-to-libc attacks in order to defeat the non-executable stack. This wasn't confined to `system("/bin/sh")` and used other functions such as `strcpy()` and chain them together.

### 16/04 - windows bof <a id="16-04-windows-bof"></a>

#### The Tao of Windows Buffer Overflow <a id="the-tao-of-windows-buffer-overflow"></a>

DilDog published his documents of how to write Windows based Buffer Overflows. "The Tao" becomes a template that Win32 tutorials are based on for the the next couple years

## 1999 <a id="1999"></a>

### 31/01 <a id="31-01-1"></a>

#### w00w00 on heap overflows <a id="w-00-w00-on-heap-overflows"></a>

Matt Conover and w00w00 published "w00w00 on Heap Overflows". The paper credited the previous work done on heap based overflows and documented possible exploitation using sample programs.

> "Some people have actually suggested making a "local" buffer a "static" buffer, as a fix! This not very wise; yet, it is a fairly common misconception of how the heap or bss work."

### 01/05 <a id="01-05"></a>

#### MITRE forms CVE Initiative <a id="mitre-forms-cve-initiative"></a>

### 09/09 <a id="09-09"></a>

#### Dark Spyrit Win32 Buffer Overflow <a id="dark-spyrit-win32-buffer-overflow"></a>

Barnaby Jack \(Dark Spyrit\) published "Win32 Buffer Overflows \(Location, Exploitation and Prevention\)" in Phrack 55. Using a publicly disclosed vulnerability in Seattle Labs Mail Server, he demonstrated how to locate the bug using tracing and debugging, and then walked through the shellcode challenges posed by Win32 system \(covering the use of trampoline calls\)

#### "The frame pointer overwrite" <a id="the-frame-pointer-overwrite"></a>

Klog published "The Frame Pointer Overwrite" in phrack 55. He showed how to gain execution by using a single byte overflow to overwrite the last byte of `%esp` . In some situations, this can result in the calling function retrieving its saved `%eip` from an attacker defined location resulting in altered execution flow.

### 20/09 - format string bug <a id="20-09-format-string-bug"></a>

#### Format string bug in proftpd 1.2.0pre6 <a id="format-string-bug-in-proftpd-1-2-0pre6"></a>

In the first public disclosure of format string bugs, Tymm Twillman posted an email to bugtraq after having notified the proftpd maintainers discussing remote hole in proftpd. This was due to the function `snprintf()`

### 21/10 <a id="21-10"></a>

#### Advanced buffer overflow exploits <a id="advanced-buffer-overflow-exploits"></a>

Taeh Oh publishes "Advanced Buffer Overflow Exploit" describing techniques to bypass filtering restrictions, bypassing `seteuid(getuid())` and breaking out of chroots

## 2000 <a id="2000"></a>

### 01/05  <a id="01-05-1"></a>

#### Bypassing StackGuard and StackShield <a id="bypassing-stackguard-and-stackshield"></a>

In Phrack \#56, Bulba and Kil3r published techniques to bypass StackGuard and StackShield, \(ab\)using adjacent pointers on the stack \(using GOT overwrite for reliable exploitation\).

​[http://phrack.org/issues/56/1.html](http://phrack.org/issues/56/1.html)​

#### Smashing C++ pointers <a id="smashing-c-pointers"></a>

In the same article, Rix published "Smashing c++ pointers" showing how oveflows in c++ can be used to overwrite `vptrs` and `vtables` to hijack control-flow.

#### Exploiting non-terminated adjacent memory space <a id="exploiting-non-terminated-adjacent-memory-space"></a>

Again, in Phrack \#56, [\[email protected\]](https://euanb26.gitbook.io/cdn-cgi/l/email-protection) published "Exploiting non-terminated adjacent memory space". Twitch showed the possibility of this by \(ab\)using functions traditionally cited as safer to `strcpy()`. From his paper:

> "The essence of the issue is that many functions that a programmer may take to be safe and/or 'magic bullets' against buffer overflows and do not automatically terminate strings/buffers with a NULL. That in actuality, the buffer size argument provided to these functions is an absolute size - not the size of the string"

### 24/06 <a id="24-06"></a>

#### WuFTPD: Providing \*remote\* root since at least 1994 <a id="wuftpd-providing-remote-root-since-at-least-1994"></a>

The tf8 WuFTPD remote format string exploit was released publicly bringing format strings exploitation into the public eye.

​[https://www.giac.org/paper/gsec/214/wu-ftp-root/100712](https://www.giac.org/paper/gsec/214/wu-ftp-root/100712)​

#### Format bugs: what are they, Where they come from? <a id="format-bugs-what-are-they-where-they-come-from"></a>

Lamagra Argamal posts to bugtraq \(Linking to the WuFTPD thread\), releasing his mini paper titled "Format Bugs: What are they, Where did they come from, How to exploit them".

His 200 line paper covered the essence of the bug class \(both r/w to arbitrary memory using format string errors\), including sample code and even pointed towards format string 0days in popular FTP servers.

### 25/07 <a id="25-07"></a>

#### JPEG Com Marker vuln in Netscape <a id="jpeg-com-marker-vuln-in-netscape"></a>

Solar Designer makes a low-key posting titled "JPEG COM Marker Processing Vulnerability in Netscape Browsers" to the mailing lists. The write-up of the JPEG bug is very instructive \(and is the harbinger of the file format bugs that's still around today\), but much more interestingly he goes to explain the generic method of gaining code execution through `free()`\(and through `unlink()`\) with heap based overflows.

### 09/09 <a id="09-09-1"></a>

#### Format string attacks <a id="format-string-attacks"></a>

Tim Newsham releases his paper on "Format String Attacks" on bugtraq. The paper was the most comprehensive overview of the nature of the problem, and its exploitation is worth reading:

> "I know it has happened to you. It has happened to all of us, at one point or another. You're at a trendy dinner party, and amidst the frenzied voices of your companions you hear the words 'format strings attack'."

​[http://owsiany.pl/sec/format-string-attacks.pdf](http://owsiany.pl/sec/format-string-attacks.pdf)​

### 01/10 <a id="01-10"></a>

#### PaX first released <a id="pax-first-released"></a>

PaX \(a security patch which provides non-executable memory pages and full address space layout randomisation \(ASLR\) for a wide variety of architectures\) was first released for the Linux kernel.

It was initially released with just the basic PAGEEXEC method of implementing non executable pages \(emulating an NX bit if a hardware NX bit isn't available [https://en.wikipedia.org/wiki/PaX\#PAGEEXEC](https://en.wikipedia.org/wiki/PaX#PAGEEXEC)\). This made some areas of the processes address space not executable, ie. the stack and heap. This is now standard in the GNU Compiler Collection \(GCC\) and can be turned on with the flag "-z execstack".

### 30/11 <a id="30-11"></a>

#### PaX adds MPROTECT <a id="pax-adds-mprotect"></a>

PaX added MPROTECT support to prevent the introduction of new executable code into a processes address space. This is accomplished by restricting access to `mmap()` and `mprotect()` interfaces. [https://en.wikipedia.org/wiki/PaX\#Restricted\_mprotect\(\)](https://en.wikipedia.org/wiki/PaX#Restricted_mprotect%28%29)​

### 12/12 <a id="12-12"></a>

#### Overwriting the .dtors section <a id="overwriting-the-dtors-section"></a>

Juan M. Bello Rivas publishes "Ovewriting the .dtors section". The paper explained how to use of `.ctors` and `.dtors` as a means of gaining code execution. These sections are mapped into elf executables \(by gcc\) containing a list of functions to be executed before `main()`\(`.ctors` or constructors\) or after `main()`\(`.dtors` or destructors\)

## 2001 <a id="2001"></a>

### 18/02 <a id="18-02"></a>

#### Grsecurity released <a id="grsecurity-released"></a>

grsecurity \(developed by Brad Sprengler\) was released initially as a port of Solar Designer's Openwall patch to the 2.4 series of Linux kernels.

It was soon discovered that the PaX project and the developers of the two projects have been working closely together ever since. grsecurity provides complementary protections that are outside of the scope of the PaX project. The project has expanded far beyond its humble roots and pioneered numerous protection mechanisms.

### 18/06 <a id="18-06"></a>

#### IIS .ida ISAPI filter Vuln <a id="iis-ida-isapi-filter-vuln"></a>

eEye published an advisory for MS01-033, an overflow in the index ISAPI extension. The exploit makes use of a heap spray to get execution on NT4 and Windows 2000.

### 13/07 <a id="13-07"></a>

#### Code Red Worm in the Wild <a id="code-red-worm-in-the-wild"></a>

The Code Red worm was observed on the internet. Discovered and researched by Marc Maiffret and Ryan Permeh of eEye Digital Security \(who named the worm, due to drinking Code Red Mountain Dew at the time\). Interestingly, the worm made use of an overwritten Structured Event Handler \(SEH\) handler on the stack.

### 31/07 <a id="31-07"></a>

#### PaX introduces ASLR <a id="pax-introduces-aslr"></a>

PaX introduced Address Space Layout Randomisation

> "The goal of Address Space Layout Randomisation is to introduce randomness into addresses used by a given task. This will make a class of exploit techniques fail with a quantifiable probability and also allow their detection since failed attempts will most likely crash the attacked task."

### 13/08 <a id="13-08"></a>

#### StackGhost released <a id="stackghost-released"></a>

StackGhost presented at USENIX as a kernel modification to guard application return pointers.

​[https://www.usenix.org/legacy/events/sec01/full\_papers/frantzen/frantzen.pdf](https://www.usenix.org/legacy/events/sec01/full_papers/frantzen/frantzen.pdf)​

#### FormatGuard released <a id="formatguard-released"></a>

Crispin Cowan et al published "FormatGuard: Automatic Protection From printf Format String Vulnerabilties"

​[https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.6.9752&rep=rep1&type=pdf](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.6.9752&rep=rep1&type=pdf)​

### 08/11 <a id="08-11"></a>

#### VUDO malloc tricks <a id="vudo-malloc-tricks"></a>

MaXX \(Michael Kaempf\) published "VUDO Malloc Tricks" in Phrack 57. The paper could have been titled "How to smash the Heap for fun and profit". The paper documented techniques against libc's native Doug Lee's `malloc` and demonstrated the generic `unlink()` write4 technique against the published vulnerability in sudo-1.6.1-1. MaXX's article went on, however, to document the DLmalloc allocator in great detail.

#### Once upon a free\(\) <a id="once-upon-a-free"></a>

In the same issue, anonymous wrote "Once upon a free\(\)" as a gentle introduction to heap based overflows using solar's `unlink()` technique.

### 28/12 <a id="28-12"></a>

#### Advanced return-2-libc <a id="advanced-return-2-libc"></a>

Nergal published Advanced return-2-lib\(c\) exploits \(PaX case study\) in Phrack 58. The article covered standard and advanced ret-2-libc attacks and included ret-2-libc chaining with retrop. This bypassed the first ASLR implementations.

## 2002 <a id="2002"></a>

### 04/02 <a id="04-02"></a>

#### Advantages of block based binary analysis <a id="advantages-of-block-based-binary-analysis"></a>

Dave Aitel publishes "The advantages of Block-Based Protocol Analysis for Security Testing". The paper documented SPIKE his block based fuzzer and to some extent revive interest in fuzzing.

​[http://immunityinc.com/downloads/advantages\_of\_block\_based\_analysis.pdf](http://immunityinc.com/downloads/advantages_of_block_based_analysis.pdf)​

### 07/02 <a id="07-02"></a>

#### Third generation exploits <a id="third-generation-exploits"></a>

Halvar Flake presented "Third Generation Exploits on NT/Win2K Platforrms".

He covered the 4 byte write anywhere heap unlink attack and documented using SEH on windows as an attack vector.

### 13/02 <a id="13-02-1"></a>

#### Visual C++ adds /GS compiler protection <a id="visual-c-adds-gs-compiler-protection"></a>

Microsoft released their /GS \(Buffer Security Check\) with Visual C++ 7. The compiler option places a cookie before the return address and calls `__security_check_cookie` during a function epilogue to detect stack corruption. Although /GS is similar to Crispin Cowan's StackGuard, Microsoft claims to have invented it independently.

​[https://docs.microsoft.com/en-us/cpp/build/reference/gs-buffer-security-check?view=vs-2019](https://docs.microsoft.com/en-us/cpp/build/reference/gs-buffer-security-check?view=vs-2019)​

### 14/02 <a id="14-02"></a>

#### Published flaw in /GS <a id="published-flaw-in-gs"></a>

Citigal release a technical paper on Microsoft /GS compiler option and call it a "vulnerability seeder". They document a flaw with adjacent variables/parameters over writes and contrast it with StackGuard.

### 27/02 - first CVE <a id="27-02-first-cve"></a>

Heap corruption in the "/usr/bin/at" program that allows local users to execute arbitrary code via a malformed execution time, which causes at to free the same memory twice.

### 05/03 <a id="05-03"></a>

#### Non-stack based exploitation <a id="non-stack-based-exploitation"></a>

David Litchfield \(mnemonix\) published "Non-stack Based Exploitation of Buffer Overrun Vulnerabilities on Windows NT/2000/XP" which essentially documents ret2libc style attacks on Win32

​[https://research.nccgroup.com/wp-content/uploads/2020/07/non\_stack\_based\_exploitation\_of\_buffer\_overrun\_vulnerabilities\_on\_windows\_nt2000xp.pdf](https://research.nccgroup.com/wp-content/uploads/2020/07/non_stack_based_exploitation_of_buffer_overrun_vulnerabilities_on_windows_nt2000xp.pdf)​

### 28/07 <a id="28-07"></a>

#### Bypassing PaX ASLR protection <a id="bypassing-pax-aslr-protection"></a>

Tyler Durden published an article in Phrack issue 59 titled "Bypassing PaX ASLR protection". He demonstrates using an information leak through partial overwrite to obtain information

#### Advances in format strings exploitation <a id="advances-in-format-strings-exploitation"></a>

In the same Phrack issue, riq and gera publish "Advances in Format String Exploitation".

 From the article:

> "it focused on 'different tiny tricks' that may help speeding up bruteforcing when exploiting format strings bugs, and ... about exploiting \(sic\) heap based format strings bugs"

### 30/07 - integer overflows <a id="30-07-integer-overflows"></a>

#### Integer overflows introduced to the public <a id="integer-overflows-introduced-to-the-public"></a>

During the talk "Professional Source Code Auditing" the group consisting of Mark Dowd, Chris Spencer, Neel Metha, Nishrad Herath and Halvar Flake discus integer overflows publicly for the first time. Several examples are given that include the now infamous pre-malloc multiplication and roundup issues.

### 31/07 <a id="31-07-1"></a>

#### PaX advances <a id="pax-advances"></a>

PaX introduces non-relocatable executable file randomisation and vma mirroring.

### 01/08 <a id="01-08"></a>

#### Syscall proxying <a id="syscall-proxying"></a>

Maximiliano Caceres of CORE publishes "Syscall Proxying - Simulating Remote Execution"

### 03/08 <a id="03-08"></a>

#### grsecurity gets Learning Mode <a id="grsecurity-gets-learning-mode"></a>

A learning mode was added automatically generates policies for individual subjects by exercising normal behavior.

### 04/09 <a id="04-09"></a>

#### 4 tricks to bypass StackGuard and StackShield <a id="4-tricks-to-bypass-stackguard-and-stackshield"></a>

Gerado Richarte \(Gera\) published "4 different tricks to bypass StackShield and StackGuard protection". The paper focused on bypassing compiler level protections StackGuard, StackShield and /GS.

​[https://www.cs.purdue.edu/homes/xyzhang/spring07/Papers/defeat-stackguard.pdf](https://www.cs.purdue.edu/homes/xyzhang/spring07/Papers/defeat-stackguard.pdf)​

### 13/09 <a id="13-09"></a>

#### Slapper worm targets Apache/mod SSL <a id="slapper-worm-targets-apache-mod-ssl"></a>

The Slapper worm was discovered in the wild by exploiting a previously disclosed bug in OpenSSL which was released in July 2002.

The worm was interesting in that it made use of a memory leak to reliably exploit a remote heap overflow.

​[https://www.f-secure.com/v-descs/slapper.shtml](https://www.f-secure.com/v-descs/slapper.shtml)​

### 31/10 <a id="31-10"></a>

#### PaX releases kernel stack randomisation <a id="pax-releases-kernel-stack-randomisation"></a>

PaX introduces RANDKSTACK to introduce randomness into the kernel stack

### 01/12 <a id="01-12"></a>

#### grsecurity adds /dev/mem and /dev/kmem protection <a id="grsecurity-adds-dev-mem-and-dev-kmem-protection"></a>

A feature preventing writes to kernel memory through `/dev/mem` and `/dev/kmem` , while allowing legitimate writes to certain ranges by XFree86 and others, was added. This was developed mainly in response to sd and devik's Phrack issue 58 paper on runtime kernel patching \(through the techniques of `/dev/[k]mem` abuse go back as far as Silvio Cesare's work in November 1998\)

### 31/12 <a id="31-12"></a>

#### Buffer overflow in ssldump 0.9b2 <a id="buffer-overflow-in-ssldump-0-9-b2"></a>

There was a buffer overflow in ssldump v 0.9b2 and earlier which allows remote attackers to cause a denial of service via a crafted SSLv2 challenge value.

​

## 2006 <a id="2006"></a>

### 26/09 <a id="26-09"></a>

Ben Hawkes presented at Ruxcon 2006 detailing how to bypass SSP stack canaries using brute force to find the canary value. This was in his talk "Exploiting OpenBSD" \(A lot of the other talks seem very interesting, not just bin exp related - [https://seclists.org/pen-test/2006/Sep/202](https://seclists.org/pen-test/2006/Sep/202)\)

​[http://inertiawar.com/openbsd/hawkes\_openbsd.pdf](http://inertiawar.com/openbsd/hawkes_openbsd.pdf)​

​

​

​[https://www.cvedetails.com/vulnerability-list.php?vendor\_id=0&product\_id=0&version\_id=0&page=1&hasexp=0&opdos=0&opec=0&opov=0&opcsrf=0&opgpriv=0&opsqli=0&opxss=0&opdirt=0&opmemc=1&ophttprs=0&opbyp=0&opfileinc=0&opginf=0&cvssscoremin=0&cvssscoremax=0&year=0&month=0&cweid=0&order=2&trc=5339&sha=5829c45b747ab5143004640f312c7f72e5b102db](https://www.cvedetails.com/vulnerability-list.php?vendor_id=0&product_id=0&version_id=0&page=1&hasexp=0&opdos=0&opec=0&opov=0&opcsrf=0&opgpriv=0&opsqli=0&opxss=0&opdirt=0&opmemc=1&ophttprs=0&opbyp=0&opfileinc=0&opginf=0&cvssscoremin=0&cvssscoremax=0&year=0&month=0&cweid=0&order=2&trc=5339&sha=5829c45b747ab5143004640f312c7f72e5b102db)​

