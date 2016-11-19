book-rev9
#Chapter0 Operating system interfaces
The collection of system calls that a kernel provides is the interface that user programs see.  

##Processes and memory  
通过fork()调用来创建child process，子进程初始状态的的内存内容与父进程相同，但是子进程和父进程在执行程序时，使用了不同的内存区域和寄存器，因此，改变内存值时两者互不影响。  

* 设计fork()和exec()的原因  
	在xv6中，fork()分配的内存要满足能够存放父进程的内存信息，而exec()则需要能够存放载入的可执行文件。  
	fork()使得子进程复制了父进程中的文件描述符表，以及相同的内存内容，这样，子进程就与父进程具有相同的打开的文件，当子进程调用exec()时，内存内容虽被替换，但文件描述符表依然不变，这样，使得shell很方便的实现重定向操作。 
	fork()和exec()的分开，使得子进程在exec()之前，重定向I/O   
	fork()之后，父进程和子进程共用各文件的偏移。

##I/O and File descriptors  
进程通过打开文件/打开目录/打开设备/创建管道/复制文件来获得一个文件描述符（fd，File descriptors），fd是对文件/管道/设备的抽象，代表着字节流。  
对于当前进程来说，新生成的fd必然是可用的fd中的最小值。

xv6 kernel中，每个进程有一张表，用fd作为索引，fd从0开始，约定0表示标准输入，1标准输出和2表示标准错误输出。  
dup()调用之后，对源fd和目的fd，其共用偏移。

	#2>&1表示值为2的fd是值为1的fd的复制，因此fd=2也是标准输出了，这样无论错误信息（fd=2）还是一般的输出信息（fd=1），都要输出到fd=1了，这样最后重定向到tmp1
	ls existing-file non-existing-file > tmp1 2>&1

##Pipes
pipe是成对出现的，small kernel buffer，一个用于读，一个用于写，用两个fd表示。pipe提供了一种进程通信的途径。  
pipe的读端会等待，直到有数据写入，或者写端关联的fd都关闭时，后一种情况read()返回0，read操作只有在write关联的所有fd都关闭时，才会看到end-of-file；若读端关联的所有fd都关闭后，write操作时，进行write的进程会收到一个SIGPIPE的信号，若该进程忽略SIGPIPE信号，写操作失败并产生EPIPE错误。  
pipe容量有限，具体大小与实现有关，不同的linux版本都会有不同的大小，从Linux 2.6.35开始，可以设置。

* linux manual中的定义  
	Pipes and FIFOs (also known as named pipes) provide a unidirectional interprocess communication channel.  A pipe has a read end and a write end.  Data written to the write end of a pipe can be read from the read end of the pipe.  

* I/O on pipes and FIFOs  
	Pipe和FIFO，二者只有创建和打开时的操作不同，一旦二者建立成功，它们随后的semantics是完全一样的。当一个进程读取空的pipe时，会阻塞，当进程向满的pipe写入时，会阻塞。  

* pipe相对于文件重定向的优势：

		#以下两句有同样的作用
		echo hello world | wc
		echo hello world >/tmp/xyz; wc </tmp/xyz
（1）Pipe无需临时文件那样需要显式的进行清除  
（2）Pipe可以传送任意长度的数据，重定向则需要硬盘上留有足够空间（临时文件）来存储所有传输的数据  
（3）pipe可以并行执行，而重定向只能串行  
（4）pipe效率更高  

* 其它  
	关于P13的一句话，The right end of the pipeline may be a command that itself includes a pipe，以及下一页的In principle, you could have the interior nodes run the left end of a pipeline, but doing so correctly would complicate the implementation. 也就是说在xv6中，目前只是实现了右端的command可以包含pipe，而左端未实现，至于主流的linux等系统，应该是实现了吧，<font color='red'>待核实</font>  
	P14这句话，The leaves of this tree are commands and the interior nodes are processes that wait until the left and right children complete. 内部结点便是书中举例的A，而左右子节点则为b和c，严格来说，b和c只能叫子节点。  

##File system

每一file的信息，保存于结构体stat中，注意，该结构体中并没有包含文件的名字信息，结构体中有一个inode成员（无符号整型），不同的文件其inode成员各不相同。对于每个inode，其可以有多个文件名称，这种实现机制称为links。

	The link system call creates another file system name referring to the same inode as an existing file.
	The unlink system call removes a name from the file system.

当链接到inode的数量为0时，inode和其占用的存储空间才会被释放。  

关于命令的设计，书中说了对于mkdir, ln, rm等都是在用户层，这样在编写其它用户程序时，可以直接借用这些命令，从而可以扩展出新的命令（仅在用户层即可以实现）。书中也说到，对于Unix的这些命令，是将其集成到shell中，而shell又是在kernel中。<font color='red'>待核实</font>  

书中说到了命令cd的特殊情况，在xv6中，其是在shell中实现的。书中也说道了，假如不在shell中（在子线程中实现，即run as a regular command）实现的情形。  

#Chapter 1 Operating system organization
对于进程的管理，OS必须能够满足三种要求：multiplexing, isolation, and interaction.

##User mode, kernel mode, and system calls
x86处理器有两种模式，kernel mode and user mode，在kernel mode，处理器可以执行privileged instructions。应用程序只能在user mode下运行，运行于kernel mode下的程序我们称之为kernel。处理器有一条特殊的指令用于切换到kernel mode，之后进入到由kernel指定entry point。在切换到kernel mode后，kernel会对系统调用的参数进行检查，以决定是否执行或者拒绝执行。

##Kernel organization

* monolithic kernel 和 microkernel  
	在设计kernel时，一个关键问题是，OS的哪些模块需要运行于kernel mode？于此，有两种解决方案，一个是全部，一个是部分，前者便是monolithic kernel，而后者则是microkernel  

* real world  
	Linux及多数Unix，是monolithic kernel，不过有些OS的功能是在用户态运行，如X Window  

## Process overview  
	
The unit of isolation in xv6 (as in other Unix operating systems) is a process.   
The mechanisms used by the kernel to implement processes include the user/kernel mode flag, address spaces, and time-slicing of threads.  
为了增强isolation，process抽象为一台独立的机器（独立的内存空间（其它进程不能读写），独立的CPU），为process中运行的程序所私有。  
xv6使用page tables（由硬件实现），来给每一个进程分配其私有内存空间。Page table将虚拟空间（**the address that an x86 instruction manipulates**）映射到物理空间（(an address that the processor chip sends to main memory）。由此可知，执行时，是在virtual address. <font color='red'>待核实。</font>  
每一个process，其address space（注意，是virtual address space），其会将用户程序和kernel程序映射到地址空间，当用户程序执行系统调用时，会执行映射到该进程的kernel对应的指令。这种设计，可以直接执行user memory中的指令（注意，系统调用需要在kernel model中被执行，而此处是说kernel 的那些系统调用指令，都在user memory中）。因此，为了给用户留足user memory，kernel是占用的最高地址的那部分空间。


* struct proc  
	用于保存进程的信息。  
	Each process has two stacks: a user stack and a kernel stack (p->kstack).当代码在用户态执行时，使用user stack，而当系统调用或者中断时，此时会执行kernel code，并会使用kernel stack。进程会在两个stack之间切换  
	成员pgdir用于保存进程的page table（按x86的格式，page table也用于保存分配给进程的物理页面的地址），当执行进程时，xv6会将pgdir的值给paging hardware使用。

##Code: the first address space  
载入boot loader到内存，之后执行，boot loader载入xv6 kernel（载入到物理地址0x100000处，0x0-0x100000之间已被I/O设备使用，而更高的地址在有些设备上不存在，比如内存很小的设备），然后在entry label处执行kernel。此时，x86的paging hardware未被使能，因此，虚拟地址直接映射到物理地址。  

在entry label开始，将虚拟地址0x80000000映射到物理地址为0x0。  

P22的这句话，Setting up two ranges of virtual addresses that map to the same physical memory range is a common use of page tables, and we will see more examples like this one. 将两段虚拟地址映射到相同的物理地址，这是page tables的惯用法，我的理解是这样，两个进程（或者多个）都可以将其虚拟内存空间中的kernel section映射到0x0. <font color='red'>待核实。</font>  此外，从这句话看出page table起着映射的作用。  


#Chapter 5 Scheduling

如何运行多于处理器个数的进程数？A common approach is to provide each process with the illusion that it has its own virtual processor by multiplexing the processes onto the hardware processors.  

##Multiplexing
切换进程的情形：  
（1）当进程A在等待I/O完成，或者等待子线程退出，或者等待sleep()系统调用返回；  
（2）xv6周期性的强制进行切换；  


#Appendix A

##PC hardware
大体上可以抽象为3个部分：CPU, memory, and input/output (I/O) devices.  

The modern x86 provides eight general purpose 32-bit registers—%eax, %ebx, %ecx, %edx, %edi, %esi, %ebp, and %esp, and a program counter %eip(the instruction pointer).前缀e表示extend之意，因为不加e表示以前使用的16-bit的寄存器。  

%ax is the bottom half of %eax, %al and %ah denote the low and high 8 bits of%ax;

In addition to these registers, the x86 has eight 80-bit floating-point registers as well as a handful of specialpurpose registers like the control registers %cr0, %cr2, %cr3, and %cr4; the debug registers %dr0, %dr1, %dr2, and %dr3; the segment registers %cs, %ds, %es, %fs, %gs, and %ss; and the global and local descriptor table pseudo-registers %gdtr and %ldtr.

#Appendix B
**注意，本节主要是介绍xv6关于bootloader的实现**
##The boot loader

x86的PC在启动时，先执行主板上的BIOS程序。

* BIOS的作用如下：  
	（1）对硬件进行检查；  
	（2）将控制权转交给OS，其实是将控制权交给从boot sector加载的代码（boot disk最开始的512字节）。这部分代码也称为boot loader，作用是加载kernel到内存中。  
	BIOS将boot sector中的代码载入到0x7c00处，并跳到该地址，开始执行boot sector中的代码。  

* boot loader的作用  
	在boot loader开始执行时，处理器为8088模式，loader需要将处理器切换到more modern operating mode，以及载入kernel，并将控制权转交给kernel.  

在xv6中，bootloader由两个文件实现，bootasm.S和bootmain.c

##Code: Assembly bootstrap（bootasm.S）
最开始执行cli（禁用中断），当xv6 kernel运行时，才重新开启中断。  
处理器进入real mode，此时其仿真为8088，

		