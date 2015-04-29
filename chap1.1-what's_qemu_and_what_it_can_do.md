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

## Binary Translation
To understand what the translation means, let's take a look a small example. The piece of code shown below executes in the guest os:

~~~c
#inlcude <stdio.h>
int main(void){
	printf("hello qemu\n");
	return 0;
}
~~~
Code 1.1: Simple Example: hello qemu

And its corresponding i386 instructions:

~~~nasm
0x080483e4	<+0>	push	%ebp
0x080483e5	<+1>	mov 	%esp,%ebp
0x080483e7	<+3>	and 	$0xfffffff0,%esp
0x080483ea	<+6>	sub 	$0x10,%esp
0x080483ed	<+9>	movl	$0x80484c0,(%esp)
0x080483f4	<+16>	call	0x8048318 <puts@plt>
0x080483f9	<+21>	mov		$0x0,%eax
0x080483fe	<+26>	leave
0x080483ff	<+27>	ret
~~~
Code 1.2: i386 Instructions of 1.1

So what's really happening when the instructions showed in 1.2 are executing in the guest OS that Qemu emulates? 

Generally, it can be divided into two steps:

1. Qemu translates the **guest instructions** into its **Intermediate Representations** (IRs)
2. Qemu translates the corresponding IRs into **host instructions**

First, let's take a look an example of step 1:

~~~nasm
OP after optimization and liveness analysis:
 ld_i32 tmp12,env,$0xfffffffc
 movi_i32 tmp13,$0x0
 brcond_i32 tmp12,tmp13,ne,$L0

 ---- 0x80483e4
 movi_i32 tmp12,$0x4
 sub_i32 tmp2,esp,tmp12
 qemu_st_i32 ebp,tmp2,leul,$0x1
 mov_i32 esp,tmp2

 ---- 0x80483e5
 mov_i32 ebp,esp

 ---- 0x80483e7
 movi_i32 tmp1,$0xfffffff0
 and_i32 tmp0,esp,tmp1
 mov_i32 esp,tmp0
 discard cc_src
 discard cc_src2
 discard cc_op
 ...
~~~
Code 1.3: Portion of IRs that Qemu Translates from Code 1.2

Code 1.3 shows a portion of IRs (from `0x80483e4` to `0x80483e7`) that are translated from the instructions shown in Code 1.2. 

Afterwards, Qemu further translates these IRs into host instructions as shown below:

~~~nasm
OUT: [size=326]
0xaa9ab870:  mov    -0x4(%ebp),%ebx
0xaa9ab873:  test   %ebx,%ebx
0xaa9ab875:  jne    0xaa9ab946
0xaa9ab87b:  mov    0x10(%ebp),%ebx
0xaa9ab87e:  sub    $0x4,%ebx
0xaa9ab881:  mov    0x14(%ebp),%esi
0xaa9ab884:  mov    %ebx,%eax
0xaa9ab886:  mov    %ebx,%edx
...
~~~
Code 1.4: Portion of Host Instructions Translated from IRs showed in Code 1.3

Eventually, the instructions showed in Code 1.4 will be executed in the host machine. That finishes the whole translation process from guest instructions to host instructions, which is called [binary translation](http://en.wikipedia.org/wiki/Binary_translation) some time.

---
<a name="is">1</a>: [Instruction set. (2015, March 21). In Wikipedia, The Free Encyclopedia.](http://en.wikipedia.org/w/index.php?title=Instruction_set&oldid=652852936)

<a name="wiki_qemu">2</a>: [QEMU. (2015, April 14). In Wikipedia, The Free Encyclopedia](http://en.wikipedia.org/w/index.php?title=QEMU&oldid=656505615)