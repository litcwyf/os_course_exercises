# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？


硬件设计上至少需要有：MMU内存管理模块、时钟中断、文件系统的支持、CP0寄存器给出相关的信息等。
从课件中可以了解到所需要的特权指令大概分为：

Exceptions: LIDT, LTR, IRET, STI, CLI

Virtual Memory: MOV CRn, INVLPG, INVPCID 

Privilege Modes: SYSRET, SYSEXIT, IRET 

Segmentation/Paging: LGDT, LLDT CRx: CR0, CR3

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？


实模式可访问的内存大小只有1MB（16位），而保护模式下可以访问32位（4GB）。实模式下没有分页和分段的机制，访问的地址空间即为实地址；保护模式下访问的是虚地址，可以在MMU内存管理单元中对某些关键的实地址进行保护。

内存条里面的地址就是所谓物理地址，访问某个物理地址即为读出内存中某个特定位置的内容；逻辑地址是一个类似于虚拟地址一样的概念，应用程序的程序员在使用地址时使用的就是逻辑地址；线性地址是逻辑地址变换到物理地址的中间层，逻辑地址通过段机制转换为线性地址，并通过页机制转换为物理地址（页机制没有启用时线性地址即物理地址）。

- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？

risc-v的特权模式即risc-v的工作模式，分机器模式、监督模式、用户模式、Hypervisor模式四种。其中用户模式硬件必须实现。

不同的模式包含不同的CSR（control and status register）。CSR通过一个12位的CSR地址进行访问控制。在不同的模式下，操作CSR地址对应的内容的权限不同的。具体的权限划分可以参考相应的资料来具体确认。


- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

冒号后的数字表示冒号前的变量在struct中所占的二进制位数。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

0x20003，即为gd_ss和gd_off_15_0拼接在一起的结果。

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

我选择了process文件夹中的switch.s

```
.text
.globl switch_to
switch_to:                      # switch_to(from, to)

    # save from's registers
    movl 4(%esp), %eax          # eax points to from
    popl 0(%eax)                # save eip !popl
    movl %esp, 4(%eax)
    movl %ebx, 8(%eax)
    movl %ecx, 12(%eax)
    movl %edx, 16(%eax)
    movl %esi, 20(%eax)
    movl %edi, 24(%eax)
    movl %ebp, 28(%eax)

    # restore to's registers
    movl 4(%esp), %eax          # not 8(%esp): popped return address already
                                # eax now points to to
    movl 28(%eax), %ebp
    movl 24(%eax), %edi
    movl 20(%eax), %esi
    movl 16(%eax), %edx
    movl 12(%eax), %ecx
    movl 8(%eax), %ebx
    movl 4(%eax), %esp

    pushl 0(%eax)               # push eip

    ret
```
这段代码需要进行的任务应该是操作系统中进程切换时，需要保存当前进程的寄存器的值，并且把要切换的进程的寄存器的值进行读入。
栈中传入的应该是两个进程的某个暂存内容的struct的首地址，保存寄存器即把当前寄存器的值放入struct中，然后从另一个struct中读取寄存器的值。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

例子：在ucore的memlayout.h中，KMEMSIZE作为一个常数，被#DEFINE来进行定义。此处即为起到一个定义常量、方便编写代码的作用。


## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
