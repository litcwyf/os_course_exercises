#lec8: 虚拟内存spoc练习


NOTICE
- 有"w4l2"标记的题是助教要提交到学堂在线上的。
- 有"w4l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。

提前准备

- 完成lec8的视频学习和提交对应的在线练习
- git pull uco
re_os_lab, v9_cpu, os_course_spoc_exercises 　in github repos。这样可以在本机上完成课堂练习。
- 理解如何实现建立页表，给用户态建立页表，在用户态使用虚地址，产生各自也访问错误的基本应对

## 视频相关思考题
### 8.1 虚拟存储的需求背景

1. 寄存器、高速缓存、内存、外存的访问特征？

存储器的访问的位置、访问的频率、访问是否需要操作系统参与管理。

2. 如何理解计算机中的存储层次结构所的理想状态是“均衡繁忙”状态？

即各个层次的存储器都处于繁忙的状态，且单位时间内对各个存储器的访问次数与访问效率的比值大致相当。

3. 在你写程序时遇到过内存不够的情况吗？尝试过什么解决方法？

遇到过；解决方法：将不需要及时访问的数据存入硬盘

### 8.2 覆盖和交换

1. 什么是覆盖技术？使用覆盖技术的程序开发者的主要工作是什么？

覆盖技术指将程序分块，并让不会同时执行的程序块共享一块内存。工作：划分程序模块、分配共享的内存。

2. 什么是交换技术？覆盖与交换有什么不同？

交换技术指将暂时不执行的程序放到外存；不同：覆盖以程序模块为单位，交换以进程为单位；覆盖需要程序员划分模块结构，交换不需要。

3. 覆盖和交换技术在现代计算机系统中还有需要吗？可能用在什么地方？

有需要。覆盖技术和交换技术的思想可以作为程序员编程时节省内存使用的方式。

4. 如何分析内核模块间的依赖关系？

使用depmod命令；静态代码分析

5. 如何获取内核模块间的函数调用列表？

静态代码分析、动态函数调用跟踪

### 8.3 局部性原理

1. 什么是时间局部性、空间局部性和分支局部性？

时间局部性：被引用过一次的存储器位置在短时间的未来很可能会被多次引用；被执行过一次的指令在短时间的未来很可能会被多次执行

空间局部性：一个存储器的位置被引用，在短时间的未来它附近的位置也很可能会被引用

分支局部性：一条跳转指令跳转到某个位置，在短时间的未来这条跳转指令也很可能会跳转到这个位置

2. 如何提高程序执行时的局部性特征？

在不影响程序结构及程序语义时，尽可能地访问最近访问地址附近的内容、尽可能地使用最近访问过的数据。

3. 排序算法的局部性特征？

基于扫描+交换的排序算法（如冒泡排序、快速排序、归并排序等），由于需要指针的单方向移动，所以具有较强的局部性。其中，冒泡排序只使用一根指针，局部性强；快速排序、归并排序有两根指针同时移动，会对局部性造成干扰。

基于数据结构的排序算法（如堆排序、使用平衡树的排序等），局部性较弱。



  * 参考：[九大排序算法再总结](http://blog.csdn.net/xiazdong/article/details/8462393)

### 8.4 虚拟存储概念

1. 什么是虚拟存储？它与覆盖和交换的区别是什么？它有什么好处和挑战？

虚拟存储：只把部分程序放到内存中。

区别：在内存和外存之间只交换进程的部分内容，交换技术需要交换整个进程的内容。

好处：可以使用大于物理空间的空间地址；速度较快（部分交换）；

挑战：实现复杂、与多核技术兼容难度大

2. 虚拟存储需要什么样的支持技术？

需要地址转换的支持以及设计优秀的置换算法


### 8.5 虚拟页式存储

 1. 什么是虚拟页式存储？缺页中断处理的功能是什么？
 
 虚拟页式存储：只装入部分页面；有需要的代码或数据不在内存时，将外存中相应的页面调入内存

 1. 为了支持虚拟页式存储的实现，页表项有什么修改？
 
 增加驻留位、修改位、访问位、保护位

 2. 页式存储和虚拟页式存储的区别是什么？
 
 页式存储只是地址映射，虚拟页式存储需要在此基础上增加置换算法，以实现虚拟内存。

### 8.6 缺页异常

1. 缺页异常的处理流程？

先在外存中查找需要访问的页面，再将其换入，并修改页表项。最后重新执行导致缺页异常的指令


2. 虚拟页式存储管理中有效存储访问时间是如何计算的？

EAT = 访存时间 * (1-p) + 缺页异常处理时间 * 缺页率p


## 个人思考题

### 内存访问局部性的应用程序例子
---
(1)(w4l2)下面是一个体现内存访问局部性好的简单应用程序例子，请参考，在linux中写一个简单应用程序，体现内存局部性差，并给出其执行时间。
```
#include <stdio.h>
#define NUM 1024
#define COUNT 10
int A[NUM][NUM];
void main (void) {
  int i,j,k;
  for (k = 0; k<COUNT; k++)
  for (i = 0; i < NUM; i++)
  for (j = 0; j	 < NUM; j++)
      A[i][j] = i+j;
  printf("%d count computing over!\n",i*j*k);
}
```
可以用下的命令来编译和运行此程序：
```
gcc -O0 -o goodlocality goodlocality.c
time ./goodlocality
```
可以看到其执行时间。

## 小组思考题目
----

### 缺页异常嵌套

（1）缺页异常可用于虚拟内存管理中。如果在中断服务例程中进行缺页异常的处理时，再次出现缺页异常，这时计算机系统（软件或硬件）会如何处理？请给出你的合理设计和解释。

> 提示：https://en.wikipedia.org/wiki/Double_fault 和 https://en.wikipedia.org/wiki/Triple_fault

### 缺页异常次数计算
（2）如果80386机器的一条机器指令(指字长4个字节)，其功能是把一个32位字的数据装入寄存器，指令本身包含了要装入的字所在的32位地址。这个过程在OS合理处理情况下最多会引起几次缺页异常？
> 提示：内存中的指令和数据的地址需要考虑地址对齐和不对齐两种情况。需要考虑页目录表项invalid、页表项invalid、TLB缺失等是否会产生异常？

### 虚拟页式存储的地址转换

（3）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持8KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示内存映射不存在（有两种情况：a.对应的物理页帧swap out在硬盘上；b.既没有在内存中，页没有在硬盘上，这时页帧号为0x7F）。
PFN6..0:页帧号或外存中的后备页号
PT6..0:页表的物理基址>>5
```

已经建立好了1个页目录表和8个页表，且页目录表的index为0~7的页目录项分别对应了这8个页表。

在[物理内存模拟数据文件](./04-1-spoc-memdiskdata.md)中，给出了4KB物理内存空间和4KBdisk空间的值，PDBR的值。

请手工计算后回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents，the value of addr in phy page OR disk sector。
```
1) Virtual Address 6653:
2) Virtual Address 1c13:
3) Virtual Address 6890:
4) Virtual Address 0af6:
5) Virtual Address 1e6f:
```

请写出一个translation程序（可基于python、ruby、C、C++、LISP、JavaScript等），输入是一个虚拟地址，依据[物理内存模拟数据文件](./04-1-spoc-memdiskdata.md)自动计算出对应的pde index, pde contents, pte index, pte contents，the value of addr in phy page OR disk sector。

**提示:**
```
页大小（page size）为32 Bytes(2^5)
页表项1B

8KB的虚拟地址空间(2^13)
一级页表：2^5
PDBR content: 0xd80（1101_100 0_0000, page 0x6c）

page 6c: e1(1110 0001) b5(1011 0101) a1(1010 0001) c1(1100 0001)
         b3(1011 0011) e4(1110 0100) a6(1010 0110) bd(1011 1101)
二级页表：2^5
页内偏移：2^5

4KB的物理内存空间（physical memory）(2^12)
物理帧号：2^7

Virtual Address 0330(0 00000 11001 1_0000):
  --> pde index:0x0(00000)  pde contents:(0xe1, 11100001, valid 1, pfn 0x61(page 0x61))
  page 6c: e1 b5 a1 c1 b3 e4 a6 bd 7f 7f 7f 7f 7f 7f 7f 7f
           7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f
  page 61: 7c 7f 7f 4e 4a 7f 3b 5a 2a be 7f 6d 7f 66 7f a7
           69 96 7f c8 3a 7f a5 83 07 e3 7f 37 62 30 7f 3f
    --> pte index:0x19(11001)  pte contents:(0xe3, 1 110_0011, valid 1, pfn 0x63)
  page 63: 16 00 0d 15 00 1c 1d 16 02 02 0b 00 0a 00 1e 19
           02 1b 06 06 14 1d 03 00 0b 00 12 1a 05 03 0a 1d
      --> To Physical Address 0xc70(110001110000, 0xc70) --> Value: 02

Virtual Address 1e6f(0 001_11 10_011 0_1111):
  --> pde index:0x7(00111)  pde contents:(0xbd, 10111101, valid 1, pfn 0x3d)
  page 6c: e1 b5 a1 c1 b3 e4 a6 bd 7f 7f 7f 7f 7f 7f 7f 7f
           7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f
  page 3d: f6 7f 5d 4d 7f 04 29 7f 1e 7f ef 51 0c 1c 7f 7f
           7f 76 d1 16 7f 17 ab 55 9a 65 ba 7f 7f 0b 7f 7f
    --> pte index:0x13  pte contents:(0x16, valid 0, pfn 0x16)
  disk 16: 00 0a 15 1a 03 00 09 13 1c 0a 18 03 13 07 17 1c
           0d 15 0a 1a 0c 12 1e 11 0e 02 1d 10 15 14 07 13
      --> To Disk Sector Address 0x2cf(0001011001111) --> Value: 1c
```

## 扩展思考题
---
(1)请分析原理课的缺页异常的处理流程与lab3中的缺页异常的处理流程（分析粒度到函数级别）的异同之处。

(2)在X86-32虚拟页式存储系统中，假定第一级页表的起始地址是0xE8A3 B000，进程地址空间只有第一级页表的4KB在内存。请问这4KB的虚拟地址是多少？它对应的第一级页表项和第二级页表项的物理地址是多少？页表项的内容是什么？

## v9-cpu相关


[challenge]在v9-cpu上，设定物理内存为64MB。在os.c,os2.c,os4.c,os5的基础上实现os6.c，可体现基本虚拟内存管理机制，内核空间的映射关系： kernel_virt_addr=0xc00000000+phy_addr，内核空间大小为64MB，虚拟空间范围为0xc0000000--x0xc4000000, 物理空间范围为0x00000000--x0x04000000；用户空间的映射关系：user_virt_addr=0x40000000+usr_phy_addr，用户空间可用大小为2MB，虚拟空间范围为0x40000000--0x40200000，物理空间范围为0x02000000--x0x02200000，但只建立低地址的1MB的用户空间页表映射。可参考v9-cpu git repo的testing分支中的os.c和mem.h。修改代码为os5.c

- (1)在建立页表后，进入用户态，能够在用户态访问基于用户空间的映射关系
- (2)在用户态和内核态产生各种也访问的错误，并能够通过中断服务例程进行信息提示
- (3)内核通过中断服务例程在感知到用户态访问高地址的空间，且没有超过0x40200000时，内核动态建立页表，确保用户态程序可以正确运行



如果一个用户态进程访问一个合法用户地址，产生内存访问异常后，v9-cpu会把产生异常的pc存在哪里？中断服务例程应该如何设计，可以返回到用户态产生错误的地址再次执行?
