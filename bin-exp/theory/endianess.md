# Endianness

In English, when we read a book, we read from left to right. It's the way that we've been taught, and it's how we'll teach others. Plain and simple.

If we were to read in Arabic, we would read from right to left. That's just the way that they've been taught.

So which way does a computer read? Both! ~~Depending on the system.~~ This is what endianness is all about, which way a computer reads data.

From the miraculous source that is Wikipedia:

> In computing, **endianness** is the order or sequence of bytes of a word of digital data in computer storage. Endianness is primarily expressed as **big-endian** \(**BE**\) or **little-endian** \(**LE**\). A big-endian system stores the most significant byte of a word at the smallest memory address and the least significant byte at the largest. A little-endian system, in contrast, stores the most-significant byte at the smallest address. In some analogy, the order or sequence in which the bits are transmitted over a communication channel is sometimes termed endianness.

So what does all this ramble and jamble mean?

## Big Endian <a id="big-endian"></a>

> A big-endian system stores the most significant byte of a word at the smallest memory address and the least significant byte at the largest

Let's say that we have the integer`1234`. In English, we would read that as is, `1234`. Nice and simple. But what does it mean by the most significant byte? if we split it up into its units column, 10's column, 100's column, etc, we get the following:

| 1000's | 100's | 10's | 1's |
| :--- | :--- | :--- | :--- |
| 1 | 2 | 3 | 4 |

So, we can say that we have

* 1 \* 1000
* 2 \* 100
* 3 \* 10
* 4 \* 1

Which added together, makes `1234`. So we could say that the smallest number is 4, or more specifically `4`. Therefore, as it's the smallest value and makes the most difference to the number, it becomes the `most significant byte`. So therefore, it would be stored first, then the `2` would be next, then then`3`, then the `4`. Therefore, the computer would read the number as `1 2 3 4` \(due to the computer reading in bytes, and one integer is one byte\)

### Who uses this format? <a id="who-uses-this-format"></a>

Big endian is sometimes called "network byte-order", as the network uses it, ranging from standard UNIX socket level to standardized web binary data structures. Also, older Mac computers using 68000-series and PowerPC microprocessors formerly used big-endian.

## Little Endian <a id="little-endian"></a>

> A little-endian system, in contrast, stores the most-significant byte at the largest address

So let's take what we know from big endian, and apply that here. Again, we have the number `1234`. How will the computer store it?

| 1's | 10's | 100's | 1000's |
| :--- | :--- | :--- | :--- |
| 4 | 3 | 2 | 1 |

So, now what we're doing is flipping the number in the opposite way. As we know that the most significant byte is `4`, that would be stored first. So the computer would store the `4` first, then the `3`, then the `2`, and then finally the `1`. So now the computer would read it as `4 3 2 1`. That doesn't look right to us, but it makes perfect sense to the computer.

Say that we had the 32- bit address of `0xf7fb1580` and the computer reads data in little endian format. Well, let's put it into our table, and find out

_Note: two hex digits make up one byte_

| 1 \(0x1\) | 16 \(0x10\) | 256 \(0x100\) | 4096 \(0x1000\) |
| :--- | :--- | :--- | :--- |
| 0x80 | 0x15 | 0xfb | 0xf7 |

So, our most significant byte is `0x80` due to that changing the number the most, and our least significant is `0xf7`. Therefore, the computer would store the address as `\x80\x15\xfb\xf7` \(the `\x` notation just means hex, same as `0x`\).

### Who uses this format? <a id="who-uses-this-format-1"></a>

This is the most common way of storing multiple bytes. It is used on all of Intel's processors, as well as AMD64

## History <a id="history"></a>

Who doesn't love a bit of history, hey? We can use this information to sound like we know what we're talking about when explaining it to our friends and colleagues!

Did you know that the adjective _endian_ was first introduced in the 18th century? In the 1726 novel _Gulliver's Travels_ written by _Jonathon Swift_ , he portrays the conflict between sects of Lilliputians divided into those breaking the shell of a boiled egg from the big end or from the little end. He called them the "Big-Endians" and the "Little-Endians".

This was then brought into the computing world on the 1st of April, 1980, by _Danny Cohen_ and published by the _Internet Engineering Task Force._ He wrote an article, labelled [On Holy Wars and A Plea For Peace](https://www.ietf.org/rfc/ien/ien137.txt) which details how bytes should be ordered.

## Learn More <a id="learn-more"></a>

If you would like to learn more, Wikipedia has a load of information, linked here: [https://en.wikipedia.org/wiki/Endianness](https://en.wikipedia.org/wiki/Endianness)​

A Google search would also be helpful - remember that google is your friend!!!

