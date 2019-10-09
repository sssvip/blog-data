date: 2019/10/08
title: MIT 6828 lab1-01细读
tags: 
- 2019
- MIT
- OS
- 6828
- COURSE
---

# 课程 MIT 6.828 OS

[课程计划表地址](https://pdos.csail.mit.edu/6.828/2018/schedule.html)

很久以前尝试过学堂在线清华的os课程，Lab1、Lab2确实也存在一定难度😀。

### 为什么记录
> 方便自己严谨的看待每一句话，每一行代码，进行详细分析以及加深理解及记忆，而不是遇到困难直接快速跳过，尝试补齐欠缺知识。

不懂的时候以及理解不深刻的时候可以:

1. 尝试梳理这里有什么前置知识
2. 记录查找前置知识的过程
3. 记录阅读了哪些前置知识

随心及阅读轨迹思考轨迹记录，尽可能的还原学习路线，帮助自己理解及回顾，**如果**能帮助到其他人就是更好的结果了，还有就是尽可能把Lecture读薄。

# Lab 1: Booting a PC


## Introduction
专注3个部分，围绕三个目标学习更有效率：

1. 熟悉X86汇编、QEMU模拟器、PC加电启动程序
2. 测验boot文件夹中的bootloader
3. 深入研究kernel文件中kernel初始化模板

<!-- more -->

#### #Software Setup

> **前置知识** [GIT](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html)

> 如果觉得上面的GIT手册看起来过于专业（后面可当做手册查询），可以重点看下廖雪峰老师的git教程 对于初学和在这个试验用重点看下[时光机穿梭](https://www.liaoxuefeng.com/wiki/896043488029600/896954074659008)，本身不难，只要把它看做是方便自己管理文件版本的工具就行。

对于校外学习这门课程的同学来说，如果不会git学习到add commit diff几个能日常用到能回查改变就行了,完全会更好。

#### #Hand-In Procedure

给mit的同学们说明怎么提交代码、提交作业等等，对校外同学来说不是特重要，当然其中的make grade可以尝试使用，自动化评测当前实验的分数，目前这里用得到。

## Part 1: PC Bootstrap
第一个练习的目的是熟悉X86汇编和PC启动程序，尝试使用QEMU/GDB调试，带着自己的理解随便看😀以及准备回答后面的问题。

#### #Getting Started with x86 assembly
> **前置知识** [汇编 pcasm-book](https://pdos.csail.mit.edu/6.828/2018/readings/pcasm-book.pdf)

> 如果觉得上面的汇编pdf不便于初次看,作为快速入门可以参看[Assembly Programming Tutorial](https://www.tutorialspoint.com/assembly_programming/),但他们都是NASM风格的，但却别都不大哈，文中也提到去看NASM和GUN会汇编的区别[Brennan's Guide to Inline Assembly](http://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html)。

后面提到对于x86汇编最佳参考资料肯定是Intel官方的[80386 Programmer's Reference Manual](https://pdos.csail.mit.edu/6.828/2018/readings/i386/toc.htm),在看这门课之前我下载过[PDF](https://css.csail.mit.edu/6.858/2014/readings/i386.pdf)进行阅读,还是觉得这里给出的html版本方便阅读一些。
> 练习1: 熟悉汇编语言，暂时不用阅读他们，当你用的时候可能会查其中的一些资料。

**练习1尝试:**

练习1应该暂时不用急着尝试😀，继续一路向西...
#### #Simulating the x86

本实验使用[QEMU Emulator](http://www.qemu.org/)作为模拟器，与真机PC来开发调试相比它能更快速的开机启动和减小调测的整个过程的时间，比如不用刻盘或写进U盘插入真机和开关机，这些步骤虽然都不能少，但都进行了模拟化操作，简单快捷。
还要一个好处是可以和[GDB](http://www.gnu.org/software/gdb/)结合调试，设置断点在真机上调测也不是件容易的事情，但让QEMU和GDB联调就方便了OS开发和调测。

在lab1源码根文件夹下打开terminal执行make qemu就能看到效果，可以关闭窗口也可以输入ctrl+a x结束程序。

#### #The PC's Physical Address Space

再多深入一点了解PC启动细节，一个PC的物理地址空间布局如下图：
```c
+------------------+  <- 0xFFFFFFFF (4GB)
|      32-bit      |
|  memory mapped   |
|     devices      |
|                  |
/\/\/\/\/\/\/\/\/\/\

/\/\/\/\/\/\/\/\/\/\
|                  |
|      Unused      |
|                  |
+------------------+  <- depends on amount of RAM
|                  |
|                  |
| Extended Memory  |
|                  |
|                  |
+------------------+  <- 0x00100000 (1MB)
|     BIOS ROM     |
+------------------+  <- 0x000F0000 (960KB)
|  16-bit devices, |
|  expansion ROMs  |
+------------------+  <- 0x000C0000 (768KB)
|   VGA Display    |
+------------------+  <- 0x000A0000 (640KB)
|                  |
|    Low Memory    |
|                  |
+------------------+  <- 0x00000000

```

第一批基于16位Intel8088处理器的PC机只能寻址1MB的物理内存。早期的PC物理地址从0x00000000到0x000FFFFF结束而不是0xFFFFFFFF结束。
早期的PC唯一能使用的随机访问内存RAM是标记`Low Memory`的640KB内的内存区域，事实上，更早的PC只能配置16kb、32kb、64kb RAM.

从0x000A0000到0x000FFFFF的384kb内存区域被保留以供显示输出其它作用。最重要的部分Basic Input/Output System (BIOS)内存区域在0x000F0000到0x000FFFFF的64kb区域。
以前的设备保存在ROM中(只读),现在保存在可更新的闪存中。BIOS做一些初始化和校验内存数量等等的校验工作，然后从软盘、硬盘、CD甚至是网络中加载操作系统并移交控制权。

#### #The ROM BIOS
用QEMU调试32位机启动。

打开一个terminal 输入make qemu-nox-gdb 再打开一个terminal输入make gdb

你将看到这一行的`[f000:fff0] 0xffff0:	ljmp   $0xf000,$0xe05b `出现，这是由gdb反编译出的汇编指令，总结几点：
1. 和上面的内存布局图所表示的那样，bios起点地址为0xffff0
2. PC从 CS = 0xf000 and IP = 0xfff0开始执行
3. 第一条指令是jmp指令，将要跳转到段地址CS = 0xf000 and IP = 0xe05b去执行指令

当设备一旦重启或加电(模拟器就是启动程序),处理器进入实模式，会设置CS寄存器为0xf000和IP寄存器为0xffff0,结合起来最终意义下一次执行代码地址为
0x000fffff0,CS寄存器会向左移一位（乘以16）加上IP寄存器的值，成为下一次执行指令的位置,**前置知识**学习汇编中会说明段寄存器和段偏移地址组合寻址地址。


> 练习2: 用GDB的单步指令si 追踪BIOS的一些指令尝试猜测它们有什么作用。

当BIOS运行，它设置了一个中断描述表以及初始化了一些像VGA显示等等的设备。后面初始胡PCI总线以及所有BIOS知道的重要的设备，然后在软盘、硬盘等启动设备上搜索启动文件，加载并移交控制权执行。

**练习2尝试:**

make gdb 后出现的是这行指令 `[f000:fff0]    0xffff0:	ljmp   $0x3630,$0xf000e05b`

与文中`[f000:fff0] 0xffff0: ljmp $0xf000,$0xe05b`不一致，先暂时不管，往后单步运行分析试试，后面来查下原因(因为我直接运行是完全正确的，这里ljmp参数和文中不一致应该是理解不到位，估计是不影响流程的，后面详细看下),初步发现里面的有的绝对地址不同电脑不同版本的QEMU可能是不一致的，初步学习可以不用太过于关注，重点是熟悉单步调试，和观测BIOS执行流程。

连续运行几次si,也就是执行几条指令看看效果:
```java
 1. [f000:e05b]    0xfe05b:	cmpw   $0xffc8,%cs:(%esi)  //第一条bios跳转过来，进行比较,比较原因暂时未知
 2. [f000:e062]    0xfe062:	jne    0xd241d416  //上一条指令如果不等进行跳转
 3. [f000:e066]    0xfe066:	xor    %edx,%edx  //从左侧连续地址看出上一句没有跳转，说明上上句相等， 当前句edx进行清0
 4. [f000:e068]    0xfe068:	mov    %edx,%ss  //清零栈寄存器
 5. [f000:e06a]    0xfe06a:	mov    $0x7000,%sp //sp设置0x7000
 6. [f000:e070]    0xfe070:	mov    $0x2d4e,%dx //设置dx 0x2d4e
 7. [f000:e076]    0xfe076:	jmp    0x5575ff02  //跳转
 8. [f000:ff00]    0xfff00:	cli   //关闭中断指令，上面一句怎么就跳转到0xfff00来了不清楚，不应该转到5575ff02?
 9. [f000:ff01]    0xfff01:	cld   //字符串方向标识，可查手册
11. [f000:ff02]    0xfff02:	mov    %ax,%cx  //ax->cx 赋值
12. [f000:ff05]    0xfff05:	mov    $0x8f,%ax // 0x8f->ax
13. [f000:ff0b]    0xfff0b:	out    %al,$0x70 // out用到端口,将ax的低8位值写入端口0x70
//查到 070 CMOS RAM/RTC (Real Time Clock  MC146818),说明这里和时钟是相关的
14. [f000:ff0d]    0xfff0d:	in     $0x71,%al //读取0x71到al
//查询到071端口 CMOS RAM data port (ISA, EISA),里面存储了时间相关的值
15. [f000:ff0f]    0xfff0f:	in     $0x92,%al //读取0x92到al
//查询到92端口PS/2 system control port A  (port B is at 0061)，系统控制相关的端口
16. [f000:ff11]    0xfff11:	or     $0x2,%al //0x2换成二进制00000010
//0x2 or al 相当于一定保证al中右边第二位一定为1 也就是al-> xxxxxx1x 字母x是al中原值
17. [f000:ff13]    0xfff13:	out    %al,$0x92 //回写al到92端口
18. [f000:ff15]    0xfff15:	mov    %cx,%ax //cx->ax 还原ax的值，上面保存过
19. [f000:ff18]    0xfff18:	lidtl  %cs:(%esi) //加载中断向量寄存器  但怎么不是lidtw? 
//可以一步一步看BIOS到底干了什么，后面不再分析...个人觉得如果和其它人的绝对地址和指令有一定差异不用太担心
//毕竟你们环境不一致可能存在部分不同，因为不同版本，模拟器实现可能存在差异，不用太在意，先读薄。
```
分析BIOS过程额外参考资料:

> [端口查询参考](http://bochs.sourceforge.net/techspec/PORTS.LST)

> [详细的学习笔记参考](https://www.cnblogs.com/fatsheep9146/p/5078179.html)

## Part 2: The Boot Loader

软盘和硬盘被分为一个个512字节大小的区域，叫做扇区，扇区是最小传输单元，每次读写操作只能是以扇区为单位读取或写入一个或多个扇区的数据。如果磁盘是启动盘(**前置知识**:启动盘的区分,在第一个扇区510字节有标识是启动盘的magic number，0x55aa，这里不讨论大小端问题)
，那么第一个扇区就被叫做启动扇区。BIOS就会加载这启动扇区的数据到内存0x7c00处，也就是结束地址是0x7dff,共计512字节。然后用jmp指令设置CS:IP到0000:07c00,相当于移交了控制权到启动扇区的代码，虽然是武断的硬编码加载到0x7c00,但这是固定的和通用做法，所以没什么问题。

6.828使用传统的方式将bootloader装在512字节中，bootloader有boot文件夹中boot.S和main.c组成，仔细阅读他们，以清楚的知道他们到底干了什么，它至少要执行以下两个主要功能:

1. 从16位实模式切换到32位保护模式，因为保护模式可以寻址1MB以上的内存地址，保护模式的描述可以在[PC Assembly Language](https://pdos.csail.mit.edu/6.828/2018/readings/pcasm-book.pdf)中1.27和1.28查看，但这里你只需要知道在保护模式下段基址:段偏移在实模式下和保护模式下的不同就行，在保护模式下段偏移是32位而不是16位(也就是说段偏移地址范围从0xffff扩大到了0xffffffff)。
2. bootloader从硬盘上通过x86的特殊IO指令控制集成电路磁盘设备寄存器直接读取内核。如果你想了解清楚是哪些特殊的IO指令，可以翻阅[the 6.828 reference page](https://pdos.csail.mit.edu/6.828/2018/reference.html)的`IDE hard drive controller`一节查询，目前不用了解太多关于IO指令的细节，尽管他是开发一个操作系统非常重要的部分，从概念和体系结构来看，他们比较无趣(复杂、难😀)。


当你理解了bootloader的源码后，看看文件`obj/boot/boot.asm`，它是被GNUmakefile指引make执行编译后反编译bootloader得到的文件。反编译文件可以容易看到bootloader在内存中的指令，方便用GDB跟踪以及单步调试。同样的，`obj/kern/kernel.asm`是JOS内核的反编译文件，也可以方便的对JOS的内核进行调试。

你可以用`b`指令设置断点，例如`b *0x7c00`在0x7c00处设置一个断点，设置断点后，你可以用`c`或者`si`继续执行，`c`命令是continue缩写，意味着它会执行到下一个断点停下来，`si`指令则是单步运行，为`step instruction`的缩写，可以用`si N`一次运行N条指令。

为了查看在内存中下一条将被执行的指令，可以使用`x/i`，使用语法为`x/Ni ADDR`，意思是查看地址`ADDR`出`N`条指令情况。

[练习3实验工具资料 lab tools guide](https://pdos.csail.mit.edu/6.828/2018/labguide.html)

> 练习3: 查看实验工具资料lab tools guide,尤其那些GDB命令部分，即使你熟悉GDB,那里面也可能包含很多你不知道的对操作系统开发有用的命令。

> 在bootloader加载处`0x7c00`处设置一个断点，用`c`继续运行然后停在0x7c00,用反编译文件obj/boot/boot.asm跟踪boot/boot.S，也可以用`x/i`命令进行查看指令序列与反编译文件obj/boot/boot.asm进行对比。

> 跟踪boot/main.c中方法`bootmain()`，然后跟进`readsect`，然后精准识别`readsect`中每一条语句对应的汇编指令，跟踪完`readsect`剩余代码最后返回到`bootmain()`,然后搞清楚`for`循环的开始和结束中每一个从硬盘读取内核扇区的操作，再找到循环结束后会执行什么代码，在那设置一个断点，然后执行用`c`继续执行到那停下，然后单步运行bootloder剩余的部分。

连续文字版的`练习3`不方便直接按步骤阅读和执行，尝试进行精简拆解成工作流：

1. 熟悉GDB tool (这是必须的，不过对于练习3也不难,个人觉得可以暂缓，因为后面明确说明了指令)
2. 一个terminal执行`make qemu-nox-gdb` 另一个terminal执行`make gdb`(后面的操作都在这个调试terminal进行)
3. 输入`b *0x7c00`,然后输入`c`继续运行  (提示:每个输入都进行一次回车,虽然像废话，怕有的初学者以为是连续输入)
4. 输入`x/50i 0x7c00`,这里会输出50条后面将要执行的指令,可以对比着obj/boot/boot.asm看，应该是基本一致的代码。
5. 在boot.asm中查看bootmain()的地址，然后用`b`指令进行断点，然后尝试单步运行起来，尝试跟踪到readsect
6. 单步运行readsect地址后的指令，尽可能到搞清楚指令和readsect函数中的语句的对应关系(这里能深刻认识c语句与汇编的转化，看得到一些底层实现)
7. 找到for的起止点，查看循环内读取扇区的操作
8. 最后找到for结束点,设置断点，`c`指令会连续运行到那，单步指令走详细查看直到bootloader结束。

**练习3尝试:**

对比boot.asm内容看

```
obj/boot/boot.out:     file format elf32-i386


Disassembly of section .text:

00007c00 <start>:
.set CR0_PE_ON,      0x1         # protected mode enable flag

.globl start
start:
  .code16                     # Assemble for 16-bit mode
  cli                         # Disable interrupts
    7c00:       fa                      cli    
  cld                         # String operations increment
    7c01:       fc                      cld    

  # Set up the important data segment registers (DS, ES, SS).
  xorw    %ax,%ax             # Segment number zero
    7c02:       31 c0                   xor    %eax,%eax
  movw    %ax,%ds             # -> Data Segment
    7c04:       8e d8                   mov    %eax,%ds
  movw    %ax,%es             # -> Extra Segment
    7c06:       8e c0                   mov    %eax,%es
  movw    %ax,%ss             # -> Stack Segment
    7c08:       8e d0                   mov    %eax,%ss

00007c0a <seta20.1>:
  # Enable A20:
  #   For backwards compatibility with the earliest PCs, physical
  #   address line 20 is tied low, so that addresses higher than
  #   1MB wrap around to zero by default.  This code undoes this.
seta20.1:
  inb     $0x64,%al               # Wait for not busy
    7c0a:       e4 64                   in     $0x64,%al
  testb   $0x2,%al
    7c0c:       a8 02                   test   $0x2,%al
  jnz     seta20.1
    7c0e:       75 fa                   jne    7c0a <seta20.1>

  movb    $0xd1,%al               # 0xd1 -> port 0x64
    7c10:       b0 d1                   mov    $0xd1,%al
  outb    %al,$0x64
篇幅受限省略一部分，可自己查看剩余部分...
```

尝试到步骤4的结果

```
(gdb) x/50i 0x7c00
=> 0x7c00:	cli    //和反编译指令及地址一致
   0x7c01:	cld    //一致
   0x7c02:	xor    %eax,%eax  //可认为一致 xorw 变为了xor  清零 这里不影响
   0x7c04:	mov    %eax,%ds   //movw 变为mov 同样不影响
   //因为都会清零寄存器这里 movw只是清零32位而已.更干净😀 可以认为一致
   0x7c06:	mov    %eax,%es   //可以认为一致
   0x7c08:	mov    %eax,%ss   //可以认为一致
   0x7c0a:	in     $0x64,%al  //inb变为in同样道理不影响 可以认为一致
   0x7c0c:	test   $0x2,%al   //可以认为一致
   0x7c0e:	jne    0x7c0a     //这里将seta20.1换位了绝对地址 可以认为一致
   0x7c10:	mov    $0xd1,%al  //可以认为一致
   0x7c12:	out    %al,$0x64  //可以认为一致
   0x7c14:	in     $0x64,%al  //可以认为一致
   0x7c16:	test   $0x2,%al   //可以认为一致
   0x7c18:	jne    0x7c14     //可以认为一致
   0x7c1a:	mov    $0xdf,%al  //可以认为一致
   0x7c1c:	out    %al,$0x60  //可以认为一致
   0x7c1e:	lgdtl  (%esi)     //可以认为一致   lgdt    gdtdesc换为了这两句
   0x7c21:	fs jl  0x7c33     //可以认为一致
   0x7c24:	and    %al,%al    //movl    %cr0, %eax 换位 al求与
   0x7c26:	or     $0x1,%ax   
   0x7c2a:	mov    %eax,%cr0  //一致
   0x7c2d:	ljmp   $0xb866,$0x87c32  //$PROT_MODE_CSEG, $protcseg换位了绝对地址
   //这里值得注意的是为什么反编译文件是7c32这里段偏移地址却成为了0x87c32
   //还有反编译中找不到PROT_MODE_CSEG的踪迹，PROT_MODE_CSEG 0xb866地址从哪来的
   0x7c34:	adc    %al,(%eax) //没找到此指令
   后面不再比对细看，下面继续实验...
```
继续尝试第5步，`b *0x7d15`,前四句push指令可以暂时不用管，这是编译器为保护调用栈以及保护数据的，与每个方法结束的pop对应,bootmain这里刚好特殊点这里没有返回所以你在后面也看不到成对的pop,不重要，暂时不用管。
开始关心bootmain的中c代码`struct Proghdr *ph, *eph;`申明变量，在汇编中局部体现不出来，这里暂时不用管。接下来看到几句push和call

```
0x7d1a:	push   $0x0  //readseg第三个参数  offset
0x7d1c:	push   $0x1000  //readseg第二个参数  count
0x7d21:	push   $0x10000  //readseg第一个参数  pa
0x7d26:	call   0x7cdc  //7cdc 你在boot.asm中搜索会发现就是readseg的地址
这一句执行后也就意味着会进入readseg函数
```
下面如练习要求所说，我们重点分析readseg函数c代码与汇编指令的大致关系

c代码:
```c
// Read 'count' bytes at 'offset' from kernel into physical address 'pa'.
// Might copy more than asked
void
readseg(uint32_t pa, uint32_t count, uint32_t offset)
{
	uint32_t end_pa;

	end_pa = pa + count;

	// round down to sector boundary
	pa &= ~(SECTSIZE - 1);

	// translate from bytes to sectors, and kernel starts at sector 1
	offset = (offset / SECTSIZE) + 1;

	// If this is too slow, we could read lots of sectors at a time.
	// We'd write more to memory than asked, but it doesn't matter --
	// we load in increasing order.
	while (pa < end_pa) {
		// Since we haven't enabled paging yet and we're using
		// an identity segment mapping (see boot.S), we can
		// use physical addresses directly.  This won't be the
		// case once JOS enables the MMU.
		readsect((uint8_t*) pa, offset);
		pa += SECTSIZE;
		offset++;
	}
}

```

汇编代码:
```asm
(gdb) x/30i 0x7cdc
   0x7cdc:	push   %ebp   #保护堆栈及数据，暂时不管
   0x7cdd:	mov    %esp,%ebp  #保护堆栈及数据，暂时不管
   0x7cdf:	push   %edi  #保护堆栈及数据，暂时不管
   0x7ce0:	push   %esi  #保护堆栈及数据，暂时不管
   0x7ce1:	mov    0x10(%ebp),%edi  #这里将offset赋值到edi寄存器，这里想不清楚的话可以想一下mov %esp,%ebp
   #此时的ebp应该是指向“当时”的栈顶，因为前面push 3个参数后又push %ebp占用4字节，所以要想取出第3个参数，
   #必须取出此栈针指向的倒数第四个push的地方，
   #所以偏移0x10=>16字节 取出了当时push的0x0赋值给edi寄存器，后面的指令(0x7cfb处)能证明edi被当做offset来使用
   0x7ce4:	push   %ebx  #保护ebx值，因为后第二句要使用
   0x7ce5:	mov    0xc(%ebp),%esi  #这里将count赋值到esi寄存器    0xc=>12   取出第二个参数 
   0x7ce8:	mov    0x8(%ebp),%ebx  #这里将pa赋值到ebx寄存器    0x8=>8   取出第一个参数
   0x7ceb:	shr    $0x9,%edi  # edi也就是offset右移九位，暂时不清楚原因
   0x7cee:	add    %ebx,%esi
   0x7cf0:	inc    %edi  #offset++
   0x7cf1:	and    $0xfffffe00,%ebx
   0x7cf7:	cmp    %esi,%ebx  #while 判断条件
   0x7cf9:	jae    0x7d0d   # while 不满足结束跳转
   0x7cfb:	push   %edi  #readsect 函数第二个参数 offset  明显证明edi是offset
   0x7cfc:	push   %ebx  #readsect 函数第一个参数 pa
   0x7cfd:	inc    %edi  #offset++
   0x7cfe:	add    $0x200,%ebx  //pa+=SECTSIZE
   0x7d04:	call   0x7c7c  #调用readsect
   0x7d09:	pop    %eax  #这个的作用不是很理解,因为暂时没找到对应的push call里面跟踪了下，也没找到
   0x7d0a:	pop    %edx  #这个的作用不是很理解
   0x7d0b:	jmp    0x7cf7  # 继续while循环
   0x7d0d:	lea    -0xc(%ebp),%esp   # while结束后执行
   0x7d10:	pop    %ebx  #恢复堆栈及数据
   0x7d11:	pop    %esi  #恢复堆栈及数据
   0x7d12:	pop    %edi  #恢复堆栈及数据
   0x7d13:	pop    %ebp  #恢复堆栈及数据
   0x7d14:	ret    #返回bootmain
```

练习第6步要求单步运行readsect后的指令，搞清楚他们的对应关系，所以可以`b *0x7c7c`然后`c`停下继续分析

这里结合这boot.asm中反编译内容waitdisk分析，因为waitdisk是readsect读写磁盘的必调用的子函数，有必要先看下，后面分析waitdisk直接带过。

```c
waitdisk(void)
{
        // wait for disk reaady
        while ((inb(0x1F7) & 0xC0) != 0x40)
                /* do nothing */;
}

```
waitdisk中又存在一个内联函数，这里从后面的反编译结果能看出来
```c
static inline uint8_t
inb(int port)
{
        uint8_t data;
        asm volatile("inb %w1,%0" : "=a" (data) : "d" (port));
        return data;
}

```

可以看出来，waitdisk是一个不需要任何参数，仅仅是一个当条件不满足一直循环的函数而已，下面是它在boot.asm中被反编译的内容。

```c
void
waitdisk(void)
{
    7c6a:       55                      push   %ebp

static inline uint8_t
inb(int port)
{
        uint8_t data;
        asm volatile("inb %w1,%0" : "=a" (data) : "d" (port));
    7c6b:       ba f7 01 00 00          mov    $0x1f7,%edx  
    //从上面的端口查询参考连接可以查到，0x1f7代表了status register,这里将其送入edx表明告知需要状态寄存器的内容
    7c70:       89 e5                   mov    %esp,%ebp
    //这个虽然是常规保护栈操作，但这里不用深究，因为有asm volatile做底层处理，我们这里只是刚好看到了这三句对应的指令而已
    7c72:       ec                      in     (%dx),%al 
    //将状态寄存器的内容写进ax的第八位，因为状态寄存器本身也只有8位
        // wait for disk reaady
        while ((inb(0x1F7) & 0xC0) != 0x40)
    7c73:       83 e0 c0                and    $0xffffffc0,%eax   //这里0xffffffc0 其实也就是0xc0
    //0xc0->11000000   0x40->01000000 这里的结束条件意味着inb返回值一定要第8位为0，第7位为1
    //根据参考查询bit 7 = 1  controller is executing a command，bit 6 = 1  drive is ready
    //所以这里结束条件是设备状态就绪并且没有执行语句，这里分析符合预期
    7c76:       3c 40                   cmp    $0x40,%al  //while结束判断(设备状态是否就绪)
    7c78:       75 f8                   jne    7c72 <waitdisk+0x8> //不等继续循环
                /* do nothing */;
}
    7c7a:       5d                      pop    %ebp  //恢复堆栈
    7c7b:       c3                      ret    //waitdisk结束



```
分析了readsect子函数，回到readsect方法的0x7c7c处继续分析，后面会发现还需要insl这个内联函数

```c
static inline void
insl(int port, void *addr, int cnt)
{
        asm volatile("cld\n\trepne\n\tinsl"
                     : "=D" (addr), "=c" (cnt)
                     : "d" (port), "0" (addr), "1" (cnt)
                     : "memory", "cc");
}
```

```asm
(gdb) x/50i 0x7c7c
=> 0x7c7c:  push   %ebp #保护堆栈
   0x7c7d:  mov    %esp,%ebp  #将堆栈信息存入ebp
   0x7c7f:  push   %edi #保护edi
   0x7c80:  mov    0xc(%ebp),%ecx  #将第二个readsect参数offset放入ecx(上面分析过为什么这里是第二个参数)
   0x7c83:  call   0x7c6a #call waitdisk，等待设备就绪
   0x7c88:  mov    $0x1,%al #
   0x7c8a:  mov    $0x1f2,%edx
   0x7c8f:  out    %al,(%dx)   #outb(0x1F2, 1);  端口含义:sector count 仅读取1个扇区
   0x7c90:  mov    $0x1f3,%edx
   0x7c95:  mov    %cl,%al
   0x7c97:  out    %al,(%dx)   #outb(0x1F3, offset); 端口含义:sector number  offset控制哪一个扇区
   0x7c98:  mov    %ecx,%eax   #将offset放入edx
   0x7c9a:  mov    $0x1f4,%edx
   0x7c9f:  shr    $0x8,%eax
   0x7ca2:  out    %al,(%dx)   #outb(0x1F4, offset >> 8); 端口含义:cylinder low  
   # 暂时不理解扇区位置和扇柱低位的关系，这里是16倍关系
   0x7ca3:  mov    %ecx,%eax
   0x7ca5:  mov    $0x1f5,%edx
   0x7caa:  shr    $0x10,%eax
   0x7cad:  out    %al,(%dx)   #outb(0x1F5, offset >> 16); 16==0x10(差点以为找不到对应)   端口含义:cylinder high   、
   # 暂时不理解扇区位置和扇柱高位的关系，这里是32倍关系
   0x7cae:  mov    %ecx,%eax
   0x7cb0:  mov    $0x1f6,%edx
   0x7cb5:  shr    $0x18,%eax
   0x7cb8:  or     $0xffffffe0,%eax
   0x7cbb:  out    %al,(%dx)   #outb(0x1F6, (offset >> 24) | 0xE0);   端口含义:drive/head
   #0xE0-> 11100000  暂时不理解这里的右移
   0x7cbc:  mov    $0x20,%al
   0x7cbe:  mov    $0x1f7,%edx
   0x7cc3:  out    %al,(%dx)   #outb(0x1F7, 0x20); 端口含义:status register
   0x7cc4:  call   0x7c6a      #call waitdisk，等待设备就绪
   0x7cc9:  mov    0x8(%ebp),%edi  #这里才取出将第一个readsect参数`*dst`
   0x7ccc:  mov    $0x80,%ecx  #512/4 = 0x80 = 128
   0x7cd1:  mov    $0x1f0,%edx #端口含义:data register
   0x7cd6:  cld
   0x7cd7:  repnz insl (%dx),%es:(%edi)   #// read a sector   insl(0x1F0, dst, SECTSIZE/4);
   # 这里同样可以查端口参考手册，查出来0xf0-0xf71表示了读取扇区相关参数
   0x7cd9:  pop    %edi
   0x7cda:  pop    %ebp
   0x7cdb:  ret    

```

不理解这里的全部我觉得对于初学者不重要，更重要的是怎么分析怎么查手册，对于初学使用来说，知道readsect是传入内存地址dst和扇区偏移offset就会读取offset扇区处一个扇区512字节内容到dst处内存中就够了，对于分析来说，知道这里面有等待磁盘操作，等待磁盘干了什么事？干了读取设备状态，怎么读取设备状态？这里读了0x1f7端口的内容判断了两bit的内容，标志没有命令执行和磁盘就绪就说明磁盘可操作。然后就是outb写入告知怎么样的磁盘操作，0x1f2-0x1f7写入相关需求，然后继续判断磁盘就绪情况，最后insl读取一个扇区内容到dst，过程大致就是如此，分析至此，先走流程读薄。


这里有点跳跃，但就像堆栈一样，这里分析了readseg后又要回到bootmain方法中了
```c
#define SECTSIZE        512
#define ELFHDR          ((struct Elf *) 0x10000) // scratch space
// read 1st page off disk
readseg((uint32_t) ELFHDR, SECTSIZE*8, 0);
```
根据之前的分析，这里可以知道，ELFHDR就是读取磁盘的起点,`SECTSIZE*8`代表读取读取4096字节，0代表偏移量0，这里为什么是读取4096字节，我们来算一算对不对。

```c
struct Elf {
        uint32_t e_magic;       // must equal ELF_MAGIC
        uint8_t e_elf[12];
        uint16_t e_type;
        uint16_t e_machine;
        uint32_t e_version;
        uint32_t e_entry;
        uint32_t e_phoff;
        uint32_t e_shoff;
        uint32_t e_flags;
        uint16_t e_ehsize;
        uint16_t e_phentsize;
        uint16_t e_phnum;
        uint16_t e_shentsize;
        uint16_t e_shnum;
        uint16_t e_shstrndx;
};
```
`32*6 + 8*12 + 16*8` 怎么算都不等于4096，所以不清楚这里读取4096到0x10000的位置的用途，不过不影响后面分析。

```c
void
bootmain(void)
{
        struct Proghdr *ph, *eph;

        // read 1st page off disk
        readseg((uint32_t) ELFHDR, SECTSIZE*8, 0);

        // is this a valid ELF?
        if (ELFHDR->e_magic != ELF_MAGIC)  //数据已经读取，判断此elf是否正确
                goto bad;

        // load each program segment (ignores ph flags)
        ph = (struct Proghdr *) ((uint8_t *) ELFHDR + ELFHDR->e_phoff);  //program header 
        //ELFHDR 基址+ ELFHDR中记录的program header offset偏移
        eph = ph + ELFHDR->e_phnum;  
        //最后一个program header的地址
        //这里有点奇怪e_phnum按字面理解不应该是program header个数吗？这里按逻辑理解e_phnum应该是
        //e_phnum=sizeof(ph)*count
        for (; ph < eph; ph++)
                // p_pa is the load address of this segment (as well
                // as the physical address)
                readseg(ph->p_pa, ph->p_memsz, ph->p_offset);

        // call the entry point from the ELF header
        // note: does not return!
        ((void (*)(void)) (ELFHDR->e_entry))();//内核加载完毕，执行entry地址

bad:
        outw(0x8A00, 0x8A00);
        outw(0x8A00, 0x8E00);
        while (1)
                /* do nothing */;
}

```

上面查看的是c,再具体看一看汇编执行情况:
```asm
(gdb) x/50i 0x7d15
   0x7d15:  push   %ebp
   0x7d16:  mov    %esp,%ebp
=> 0x7d18:  push   %esi
   0x7d19:  push   %ebx
   0x7d1a:  push   $0x0
   0x7d1c:  push   $0x1000
   0x7d21:  push   $0x10000
   0x7d26:  call   0x7cdc   #前几句前面文中已经介绍过，这里call readseg
   0x7d2b:  add    $0xc,%esp   #平栈 这里是esp +c 个人认为相当于pop 三次
   0x7d2e:  cmpl   $0x464c457f,0x10000  # 比较魔数 0x10000及elf地址，这里本身也就是取elf struct首字段
   0x7d38:  jne    0x7d71    # 非elf标准文件进入bad
   0x7d3a:  mov    0x1001c,%eax  #ELFHDR + ELFHDR->e_phoff 同样能被编译器算出绝对地址 0x1001c
   0x7d3f:  movzwl 0x1002c,%esi  #扩展到0x0001002c
   0x7d46:  lea    0x10000(%eax),%ebx #此时ebx是多少  # ebx -> ph
   0x7d4c:  shl    $0x5,%esi  
   0x7d4f:  add    %ebx,%esi  # esi -> eph
   0x7d51:  cmp    %esi,%ebx   #for 结束判断
   0x7d53:  jae    0x7d6b   # 大于等于就call 读取readseg
   0x7d55:  pushl  0x4(%ebx)   #ebx ->ph
   0x7d58:  pushl  0x14(%ebx)  
   0x7d5b:  add    $0x20,%ebx
   0x7d5e:  pushl  -0x14(%ebx)   #readseg(ph->p_pa, ph->p_memsz, ph->p_offset); push3个参数
   0x7d61:  call   0x7cdc    #readseg
   0x7d66:  add    $0xc,%esp  #call 后的平栈
   0x7d69:  jmp    0x7d51   #继续for 判断
   0x7d6b:  call   *0x10018  #for 结束后执行entry()，进入内核   
   #因为elf绝对地址，所以这里也是被计算成绝对地址
   #这里能看出来 0x18=24   24/4=6 也就是结构体第6个参数的地址 
   #也就是变量e_entry的地址所以能看出来每个变量相当于一个指针占用4字节
   0x7d71:  mov    $0x8a00,%edx   # bad
   0x7d76:  mov    $0xffff8a00,%eax
   0x7d7b:  out    %ax,(%dx)
   0x7d7d:  mov    $0xffff8e00,%eax
   0x7d82:  out    %ax,(%dx)
   0x7d84:  jmp    0x7d84   #死循环
```

接下来回答问题:

1. 处理器从哪一个点开始执行32位指令的？是什么导致从16位切换到32位的？
  
    从`.code32 protcseg`这里开始执行的。
    `ljmp    $PROT_MODE_CSEG, $protcseg`此指令前的加载段描述等等前置操作以及此长跳转以及切换到了32位

2. bootloader最后一条指令是什么？内核加载后第一条指令是什么？
    
    最后一条指令是`0x7d6b:  call   *0x10018`，然后就再也没回到bootmain😀
    内核全部加载完后第一条指令也是`call *0x10018`，如果是elf加载后的话是check magic number

3. 内核的第一条指令是什么？（仔细思考和`2`中的第二问有区别吗？）
  
    很多其它问题还可以从boot.asm中静态的找到答案，这里就不能在boot.asm中找了，毕竟call后就跳出bootmain以及bootloader了，
天真的以为在kernel.asm中能找到0x10018，然后并没有找到，这个时候可以在gdb中`b *0x10018`然后si就能回答当前问题了。

    最后从kern/kernel.asm 中entry找到第一条指令`movw    $0x1234,0x472`
  
    gdb断点数据如下:
        ```asm

        Breakpoint 1, 0x00007d69 in ?? ()
        (gdb) c
        Continuing.
        => 0x7d6b:  call   *0x10018

        Breakpoint 2, 0x00007d6b in ?? ()
        (gdb) si
        => 0x10000c:  movw   $0x1234,0x472
        0x0010000c in ?? ()
        ```
      最后尝试出来了，开始理解错误了，不是call 0x10018就断点0x10018,而是应该断点这条call语句，然后成功了，si下一条语句能看出来跳转到了0x10000c,这里也看出来，以后如果想直接跳转到call的地址，只需要call addr+4就行了，这就是它的吓一跳语句地址，也能看出来走一点弯路也能收获到不一样的知识点。

4. bootloader怎么决定读取多少个内核扇区？在哪找到这些信息的？
    
    读取多少个扇区由`eph = ph + ELFHDR->e_phnum; `这句话决定了，这个信息由首次加载8个扇区`readseg((uint32_t) ELFHDR, SECTSIZE*8, 0);`信息到ELFHDR结构体得到的。

#### #Loading the Kernel

到目前为止已经2w多字了，可能大部分都是copy的代码吧😀，还是有点吓人
接下来我们关注更多在boot/main.c中的c语言细节，在开始之前，可以回顾一些c语言编程基本知识。

> 练习4: 阅读c指针部分，最好的参考资料是K&R的C编程语言,推荐在亚马逊购买,或者看mit的拷贝版。

> 阅读K&R中5.1节指针和地址到5.5节字符指针和函数，然后下载pointers.c,运行并要清楚的知道它输出的每一个值。特别是，你要明确理解1到6行的指针位置，以及2到4行的值的原因，还有为什么第五行的值似乎很奇怪。

> 另一个C语言参考资料参考A tutorial by Ted Jensen

> 警告:除非你以及比较了解C，不然别跳过这一节，不然后面的实验相当的困难。

[C编程语言](http://www.amazon.com/C-Programming-Language-2nd/dp/0131103628/sr=8-1/qid=1157812738/ref=pd_bbs_1/104-1502762-1803102?ie=UTF8&s=books)亚马逊地址

[C编程语言](http://library.mit.edu/F/AI9Y4SJ2L5ELEE2TAQUAAR44XV5RTTQHE47P9MKP5GQDLR9A8X-10422?func=item-global&doc_library=MIT01&doc_number=000355242&year=&volume=&sub_library=) MIT拷贝版

[pointers.c](https://pdos.csail.mit.edu/6.828/2018/labs/lab1/pointers.c)

[A tutorial by Ted Jensen](https://pdos.csail.mit.edu/6.828/2018/readings/pointers.pdf)

**练习4尝试:**
这里就简单理解下pointer.c,其余的要靠自己多看。
```c
#include <stdio.h>
#include <stdlib.h>

void
f(void)
{
    int a[4];
    int *b = malloc(16);
    int *c;
    int i;
    //上面申明变量或初始化不用多说
    printf("1: a = %p, b = %p, c = %p\n", a, b, c); //输出指针地址也还好

    c = a; //c指针也指向a数组
    for (i = 0; i < 4; i++)
        a[i] = 100 + i; //对a数组进行组个赋值
    c[0] = 200; //改变c[0],这里也就改变了a[0],因为a和c地址执行的是同一个地址
    printf("2: a[0] = %d, a[1] = %d, a[2] = %d, a[3] = %d\n", 
          a[0], a[1], a[2], a[3]);
    //所以a[0]=c[0]=200,a[1]如期=101 a[2]如期=102 a[3]如期=103
    c[1] = 300; //c[1]=a[1]=300
    *(c + 2) = 301; //c+2代表c[2]   这里也就意味着是c[2]=301
    3[c] = 302; //3[c]是c[3]的另一种写法，罕见。c[3]=a[3]=302
    printf("3: a[0] = %d, a[1] = %d, a[2] = %d, a[3] = %d\n",
     a[0], a[1], a[2], a[3]);
    //a[0]=200  a[1]=300 a[2]=301 a[3]=302
    c = c + 1;  //c指针移位  间接说c指向了a[1]不再指向a或a[0]
    *c = 400;  //a[1]=400
    printf("4: a[0] = %d, a[1] = %d, a[2] = %d, a[3] = %d\n",
     a[0], a[1], a[2], a[3]);
    //a[0]=200 a[1]=400 a[2]=301 a[3]=302
    //记住在执行下面语句之前c目前指向了a[1]
    c = (int *) ((char *) c + 1);  //这里的重点在于优先级，先强转char*再加1
    //也就是在强转int *之前c仅仅向后移动了一个char的位置，因为+1是c的数据类型为char *,+1也就移动一个char位置
    //所以c目前的位置在a[1]+ 1 byte
    *c = 500; //改变c当前位置的值也就改变了当前a[1]+ 1 byte位置后4byte的数据为500
    //这里需要详细写出a[1] a[2]来以说明改变情况
    //a[1] a[2] 再执行`*c = 500`前的数据情况是  400 301
    //00000000 00000000 00000000 00000000   int
    //00000000 00000000 00000001 10010000   400
    //00000000 00000000 00000001 00101101   301
    //00000000 00000000 00000001 11110100   500
    //(这里我是将十进制转16进制然后再8421编码出来得到，你也可以用在线转换工具进行转换)
    //执行`*c = 500`改变的是a[1]起第9bit到41bit之间的数据，500二进制表示00000000000000000000000111110100b
    //所以执行后a[1] a[2]的二进制表示是 00000000000000000000000000011111b 01000000000000000000000100101101
    //对不对后面执行对比下就知道
    printf("5: a[0] = %d, a[1] = %d, a[2] = %d, a[3] = %d\n",
     a[0], a[1], a[2], a[3]);
    //预期结果 a[0]不变=200 a[1]=15  a[2]=41261 a[3]不变302  //这里暂时不对后面再回过头来看下
    b = (int *) a + 1; //这个会移动4位 int
    c = (int *) ((char *) a + 1); //这里会移动1位 char
    printf("6: a = %p, b = %p, c = %p\n", a, b, c);
}

int
main(int ac, char **av)
{
    f();
    return 0;
}
```
pointer.c的line 5输出分析还有点问题，认知看来还是不够，后面回过头来还会继续纠正的。

为了深入理解boot/main.c，你需要知道ELF二进制文件。当编译并链接一个C程序(如JOS内核)时，编译器将每个C源('. C ')文件转换成一个对象('.o')文件，其中包含汇编语言指令，以硬件期望的二进制格式编码。然后连接器组合所有编译的文件到一个单二进制镜像比如obj/kern/kernel,本实验中就是ELF格式的二进制文件，全称是“可执行可连接格式”

详细的关于ELF格式可以查看[the ELF specification](https://pdos.csail.mit.edu/6.828/2018/readings/elf.pdf),但在这个实验中你不用特别深入的了解这个格式。尽管整个格式十分的有用和复杂，最复杂的部分是支持动态加载和共享包，本实验中不涉及。[Wikipedia page](http://en.wikipedia.org/wiki/Executable_and_Linkable_Format)有简短的描述。

为了课程，你可以将elf可执行文件认为是一个加载信息的头文件，紧跟着几个程序段，他们是一些被加载到特定位置的数据和代码连续块，bootloader不修改任何代码和数据，仅仅是加载后执行它。

一个ELF二进制文件仅仅以一个固定长度的ELF头开端，紧跟这一个变长的程序头，列出了需要被加载的程序段。elf头定义在inc/elf.h中，我们感兴趣的程序段有:

- .text 程序的可执行指令
- .rodata 只读数据，例如c编译器生成的ascii字段串常量（我们不用费心去设置不准硬件写入）
- .data 数据段，程序的初始化的数据，例如声明的初始化了的全局变量 int x= 5;

当连接器计算出程序的内存布局，它会保留未初始化的全局变量到.bss段，例如int x;它紧跟着.data段。c要求未初始化的数据赋默认值0。因此不需要存储.bss段的内容在elf二进制文件中，相反的，连接器只记录.bss段的大小和位置。bootloader或者程序本身需要分配0给.bss段。

检查输入`objdump -h obj/kern/kernel`命令后输出的关于内核可执行文件的名称，大小和连接地址。你会发现比我们上面列出的更多的段，但是那些段对于我们本实验不是特别重要，目前不用过多了解。其它更多的是存储了调试信息，通常情况下调试信息会包含在可执行程序中，但是不会被程序加载器加载(意味着是被调试器加载😀)。

```
obj/kern/kernel:     file format elf32-i386

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .text         000019e9  f0100000  00100000  00001000  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .rodata       000006c0  f0101a00  00101a00  00002a00  2**5
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .stab         00003b95  f01020c0  001020c0  000030c0  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .stabstr      00001948  f0105c55  00105c55  00006c55  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .data         00009300  f0108000  00108000  00009000  2**12
                  CONTENTS, ALLOC, LOAD, DATA
  5 .got          00000008  f0111300  00111300  00012300  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  6 .got.plt      0000000c  f0111308  00111308  00012308  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  7 .data.rel.local 00001000  f0112000  00112000  00013000  2**12
                  CONTENTS, ALLOC, LOAD, DATA
  8 .data.rel.ro.local 00000044  f0113000  00113000  00014000  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  9 .bss          00000648  f0113060  00113060  00014060  2**5
                  CONTENTS, ALLOC, LOAD, DATA
 10 .comment      0000002b  00000000  00000000  000146a8  2**0
                  CONTENTS, READONLY
```

特别注意一下，.text代码段的VMA连接地址和LMA加载地址，LMA的地址一段应该加载进内存（上文的00100000）。

bootloader利用ELF程序头决定怎么加载段数据，程序头指明了ELF对象哪些部分要加载近内存中，以及应该每个段应该加载的目标位置。你可以输入`objdump -x obj/kern/kernel`审查程序头，输出结果比刚刚输出要详细一些，这里就不再显示。

BIOS加载boot启动扇区到0x7c00.因此0x7c00也是启动扇区的加载地址，也是执行起始地址，也是连接地址。我们在boot/Makefrag通过-Ttext 0x7c00设置连接地址，因此连接器生成的文件可以产生正确的内存地址。

> 练习5: 大概意思是更改bootloader的连接地址，然后尝试跟踪下bootloader指令，看看会发生什么。改回来后注意make clean。

回过头来看内核的加载和连接地址，不像bootloader一样，两个地址不一样，内核个告诉bootload将其加载在地地址1M处，但它期望从一个高地址执行。我们将在下一节中深入研究如何实现这一点。

除了段信息，在ELF头中还有对我们来说很重要的信息e_entry,这个区域存有程序进入点的连接地址，程序代码段可开始执行的内存地址，你可以用`objdump -f obj/kern/kernel`看到进入点entry point

```
obj/kern/kernel:     file format elf32-i386
architecture: i386, flags 0x00000112:
EXEC_P, HAS_SYMS, D_PAGED
start address 0x0010000c
```

您现在应该能够理解boot/main.c中最小的ELF加载器了，它从磁盘将内核的每一个段读的加载地址然后读取进内存，然后跳转到内核的进入点entry point执行(移交控制权)。

> 练习6: 现在我们可以用GDB的`x`指令检查内存，GDB manual有更详细的介绍，但是现在，已经足够去知道命令`x/Nx ADDR`,它会在ADDR处输出`N`个字的内存指令
注意`x`用小写，警告:字的大小不是完全统一的，在GNU汇编中，一个字两个字节。

重置机器(退出QEMU/GDB然后重启),检查BIOS进入bootloader处0x00100000地址8个字，然后再看bootloader进入内核的点。为什么他们不同？第二个断点是什么?(不用去调试，仅仅思考就行了。)
[GDB manual](https://sourceware.org/gdb/current/onlinedocs/gdb/Memory.html)

练习6差不多前面我们也都涉及，这里就不在尝试了，继续往下。尽快进入lab2。

## Part 3: The Kernel

我们现在开始在最小内核JOS尝试深入一点更多细节。(后面你会写相同代码！)。比如bootloader，内核从一些汇编语言代码开始，这些代码设置了一些东西，以便C语言代码能够正确执行。

#### #Using virtual memory to work around position dependence


当你查看bootloader的连接和加载地址，他们高度吻合，但是他们和内核的连接地址（objdump输出的）和加载地址有（相当大）差距。回去检查确保你知道这里在说什么（上面的对boot和kernel的objdump输出）。（连接内核比连接bootloader复杂很多，因此它的连接和加载地址都在kern/kernel.ld顶部）

操作系统内核通常被连接和执行在一个高的虚拟地址，比如0xf0100000,以便将处理器的低虚拟地址留给用户程序使用，这样分配的原因会在下一个实验变的更加清晰。

很多机器在0xf0100000处并没有物理地址，因此我们不能指望有能力将内核存储在那。相反我们将用处理器的内存管理硬件去映射虚拟地址0xf0100000(因为内核连接在这里，也期望从这里运行)到0x00100000(这里就是bootloader加载内核到物理内存的地址)。这个方式，尽管内核的虚拟地址已经足够高，可以为用户进程留下足够的地址空间。它将被加载到PC设备的物理内存1M处，仅仅在BIOS ROM上，这种方法要求pc最少有1M物理内存(这样0x00100000物理地址才能用)，但是几乎适用于1990年后生产的所有电脑。

事实上，在下一个实验，我们将映射整个底部256MB物理内存空间，从物理地址0x00000000到0x0fffffff,到虚拟地址0xf0000000到0xffffffff对应的。你现在应该知道为什么JOS仅仅只能使用开始的256MB物理内存。

就目前而言，我们仅仅映射前4MB物理内存，已经足够我们开始行动了。我们使用kern/entrypgdir.c中手工编写的静态初始化的页目录和页表来实现这一点。目前你不需要完全明白它们工作的细节，只用关心它们的效果就行了。直到kern/entry.S设置CR0_PG标志，内存引用都被认为是物理地址(严格来说，它们是线性地址，但是boot/boot.S设置了一个唯一的从线性地址到物理地址的映射，我们不再改变它)。一旦CR0_PG被设置，内存引用都是虚拟地址，由虚拟地址硬件转换为物理地址。entry_pgdir翻译从0xf0000000到0xf0400000虚拟地址到物理地址0x00000000到0x00400000,同样虚拟地址0x00000000到0x00400000也被映射到物理地址0x00000000到0x00400000.任何不在这两个范围内的虚拟地址都将导致硬件异常，因为我们目前都还没有设置中断处理，将会导致QEME存储机器状态并退出(如果你没有使用6.828补丁的QEMU可能会不断重启)

> 练习7: 使用QEMU和GDB跟踪JOS内核在`movl %eax, %cr0`停下，检查在0x00100000和0xf0100000处的内存。现在使用`stepi`GDB单步指令然后继续检查0x00100000和0xf0100000出的内存。确保你理解发生了什么。

在新映射建立之后，如果映射没有就位，第一个不能正常工作的指令是什么?在kern/entry.S中注释掉`movl %eax， %cr0`跟踪一下，看看你是不是对的。

**尝试练习7:**
```
(gdb) b *0x10000C
Breakpoint 1 at 0x10000c
(gdb) c
Continuing.
The target architecture is assumed to be i386
=> 0x10000c:  movw   $0x1234,0x472
Breakpoint 1, 0x0010000c in ?? ()
(gdb) si
=> 0x100015:  mov    $0x112000,%eax
0x00100015 in ?? ()
(gdb) si
=> 0x10001a:  mov    %eax,%cr3
0x0010001a in ?? ()
(gdb) si
=> 0x10001d:  mov    %cr0,%eax
0x0010001d in ?? ()
(gdb) si
=> 0x100020:  or     $0x80010001,%eax
0x00100020 in ?? ()
(gdb) si
=> 0x100025:  mov    %eax,%cr0 #下一条单步运行设置内存映射
0x00100025 in ?? ()
(gdb) x/4xb 0x00100000
0x100000: 0x02  0xb0  0xad  0x1b   
(gdb) x/4xb 0xf0100000
0xf0100000 <_start+4026531828>: 0x00  0x00  0x00  0x00   #当前0x00100000和0xf0100000没有一致
(gdb) si     #单步运行mov    %eax,%cr0
=> 0x100028:  mov    $0xf010002f,%eax
0x00100028 in ?? ()
(gdb) x/4xb 0x00100000
0x100000: 0x02  0xb0  0xad  0x1b
(gdb) x/4xb 0xf0100000
0xf0100000 <_start+4026531828>: 0x02  0xb0  0xad  0x1b  #当前0x00100000和0xf0100000一致
(gdb) 

```

尝试注释掉entry.S文件中的%movl %eax, %cr0这句话，make clean,然后make运行

```
(gdb) b *0x10000C
Breakpoint 1 at 0x10000c
(gdb) c
Continuing.
The target architecture is assumed to be i386
=> 0x10000c:  movw   $0x1234,0x472
Breakpoint 1, 0x0010000c in ?? ()
(gdb) si
=> 0x100015:  mov    $0x112000,%eax
0x00100015 in ?? ()
(gdb) si
=> 0x10001a:  mov    %eax,%cr3
0x0010001a in ?? ()
(gdb) si
=> 0x10001d:  mov    %cr0,%eax
0x0010001d in ?? ()
(gdb) si
=> 0x100020:  or     $0x80010001,%eax
0x00100020 in ?? ()
(gdb) si
=> 0x100025:  mov    $0xf010002c,%eax
0x00100025 in ?? ()
(gdb) si
=> 0x10002a:  jmp    *%eax
0x0010002a in ?? ()
(gdb) si
=> 0xf010002c <relocated>:  add    %al,(%eax)
relocated () at kern/entry.S:74
74    movl  $0x0,%ebp     # nuke frame pointer
(gdb) si
Remote connection closed
(gdb) 
```
从0x10002a到0xf010002c这一条指令崩了，所以是通不过的，因为没有正确的设置映射。

#### #Formatted Printing to the Console

大多数人认为printf()这样的函数是理所当然的，有时甚至认为它们是C语言的“基本类型”。
但是在操作系统内核中，我们必须自己实现所有的I/O。

通过阅读kern/printf.c, lib/printfmt.c,和kern/console.c，确保理解他们的关系。后面的实验室将会清楚为什么printfmt.c位于单独的lib目录中。

> 练习8: 我们省略了一小段代码——使用“%o”形式的模式打印八进制数所需的代码。
查找并填充此代码片段。

能够回答以下问题:

1. 解释printf.c和console.c之间的接口，特别是console.c暴露了什么函数？它又是怎么被printf.c使用的？
2. 解释下面console.c中的代码:

        1      if (crt_pos >= CRT_SIZE) {
        2              int i;
        3              memmove(crt_buf, crt_buf + CRT_COLS, (CRT_SIZE - CRT_COLS) * sizeof(uint16_t));
        4              for (i = CRT_SIZE - CRT_COLS; i < CRT_SIZE; i++)
        5                      crt_buf[i] = 0x0700 | ' ';
        6              crt_pos -= CRT_COLS;
        7      }

3. 对于下面的问题，你们可以参考第二讲的讲义。这些注释涵盖了GCC在x86上的调用约定。

    一步一步的跟踪执行下面代码:

          int x = 1, y = 3, z = 4;
          cprintf("x %d, y %x, z %d\n", x, y, z);
    - 在调用cprintf()时，fmt指向什么?ap指向什么?
    - 列出(按执行顺序)对cons_putc、va_arg和vcprintf的每个调用。对于cons_putc，也列出它的参数。对于va_arg，列出调用前后ap指向的内容。对于vcprintf，列出它的两个参数的值。

4. 运行下面的代码:

        unsigned int i = 0x00646c72;
        cprintf("H%x Wo%s", 57616, &i);

    输出是什么?按照前面的练习一步一步地解释这个输出是如何得到的。这是一个ASCII表，它将字节映射到字符。
    
    输出取决于x86是little-endian这一事实。如果x86是big-endian，为了得到相同的输出，您会将i设置为什么?您是否需要将57616更改为不同的值?
    
    下面是对[小端和大端](http://www.webopedia.com/TERM/b/big_endian.html)以及[更异想天开的描述](http://www.networksorcery.com/enp/ien/ien137.txt)。

5. 在下面的代码中，'y='后面会打印什么?(注意:答案不是一个特定的值。)为什么会这样?

        cprintf("x=%d y=%d", 3);

6. 假设GCC更改了它的调用约定，以便按声明顺序在堆栈上推送参数，以便最后一个参数被推送到最后。您将如何更改cprintf或它的接口，使它仍然能够传递可变数量的参数?

#### #
#### #


继续更新中...




### test😀

```c
void 
test(void){
	return 0;
}
```
