# What's Qemu and What It Can Do

If you Google `Qemu`, you can find a lot of webpages of it, such as the wikipedia [Qemu](http://en.wikipedia.org/wiki/QEMU), and the official website [wiki.qemu.org](http://wiki.qemu.org/Main_Page), etc. They all have their different explanations about what's Qemu and what Qemu can do. 

However, based on my understanding, I think one of the most important things that Qemu can achieve is: Qemu essentially **translates** one CPU instruction set to another **on the fly**.

What's a CPU instruction set? It's a set of instructions that the CPU can understand, like the `language` of a CPU. One particular CPU can only understand one instruction set (*what if a CPU can understand more than one instruction set???*), like the people live in english-speaking area can only understand english, of course under the assumption that he or she does not know any other foreign language.

Soon you see the problem here, what if an english-speaking person and a chinese-speaking person try to communicate with each other, if they only speak native language? That's exactly the same issue for CPU: what if a program already compiled for one particular instruction set and tries to execute on another?

The simple answer is you can't unless you play some other tricks. You might wonder why you want to do that, however, it seems that a program or application (whatever you call it) can run on anywhere seems to be one of the most-wanted objectives in computer science, at lease in industry. Just like the slogan `"Write once, run anywhere"`<sup>[1](#slogan)</sup> for early Java. Imagine that an ios game can be run on a intel mac directly, or a xbox one or a ps4 game can be run on PC directly. 

*to be continue*:

What does it mean by **translation**?

What does it mean by **on the fly**?

<a name="slogan">1</a>: ["Write once, run anywhere"](http://en.wikipedia.org/wiki/Write_once,_run_anywhere)