# What Qemu is and What It Can Do

If you Google `Qemu`, you will find webpages such as the wikipedia: [Qemu](http://en.wikipedia.org/wiki/QEMU), or the official website: [wiki.qemu.org](http://wiki.qemu.org/Main_Page), etc., which have their own explanations about what Qemu is and what it can do. 

My own understanding: essentially, Qemu **translates** one CPU instruction set to another **on the fly**.

What's a CPU instruction? It's a "command" that CPU can execute, usually as "move value from A to B", or "add values from A and B and store result in A", etc.

What's a CPU instruction set? It's all the "commands" that a CPU can execute. 

A CPU can only understand one instruction set (*what if a CPU can understand more than one instruction set???*), like a person only speaks one language. There exists many [instruction sets](http://en.wikipedia.org/wiki/Comparison_of_instruction_set_architectures), such as x86, arm etc., like there exists many languages as well.

Soon you see the issue, what if two different language-speaking persons try to communicate with each other? Same issue for CPU: what if a program compiles for one particular instruction set, then tries to execute on another?

First you might wonder why you want to do that, however, let a program or application run anywhere seems to be one of the most-wanted goals. Reflected by the early slogan of Java `"Write once, run anywhere"`<sup>[1](#slogan)</sup>. Imagine an ios app can be run on a mac directly, or a xbox one/ps4 game can be run on PC directly. 

*to be continue*:

What does it mean by **translation**?

What does it mean by **on the fly**?

---
<a name="slogan">1</a>: ["Write once, run anywhere"](http://en.wikipedia.org/wiki/Write_once,_run_anywhere)