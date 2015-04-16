# What is Qemu and What It Can Do

## System Emulator
When people mention "using a PC or a smart device", they actually mean that they are using the operating system, such as Mac OS or Windows for PCs, Android or iOS for smart devices, etc.

So what about running an operating system under another, such as running a Windows under a Mac OS, or vice versa, like a Matryoshka doll?

The question actually entails two sub questions,

1. Is it able to run nested operating systems?
2. Is it able to run nested operating systems even under different instruction set architectures?

An instruction is a "command" that a CPU can execute (i.e., moves value from A to B, or adds values from A and B and stores result in A, etc.). An instruction set is all the "commands" that a CPU can execute. And an instruction set architecture includes the instruction set, as well as registers, addressing mode, etc.<sup>[1](#is)</sup>

As you might have guessed, Qemu (short for Quick Emulator<sup>[2](#wiki_qemu)</sup>), originated by [Fabrice Bellard](http://en.wikipedia.org/wiki/Fabrice_Bellard), allows

1. running nested operating systems
2. running nested operating systems under different instruction set architectures (i.e., runs an Android system under a Windows or Mac OS of x86 architecture)

To achieve the purposes, essentially, Qemu **translates** one CPU instruction set to another **on the fly**.


---
<a name="is">1</a>: [Instruction set. (2015, March 21). In Wikipedia, The Free Encyclopedia.](http://en.wikipedia.org/w/index.php?title=Instruction_set&oldid=652852936)

<a name="wiki_qemu">2</a>: [QEMU. (2015, April 14). In Wikipedia, The Free Encyclopedia](http://en.wikipedia.org/w/index.php?title=QEMU&oldid=656505615)