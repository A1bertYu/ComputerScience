###Message Passing Interface([MPI][MPI_wiki])  
The standard defines the syntax and semantics of a core of library routines useful to a wide range of users writing portable message-passing programs in C, C++, and Fortran.   
MPI provides parallel hardware vendors with a clearly defined base set of routines that can be efficiently implemented.


###Entry point  
An entry point is where control is transferred from the operating system to a computer program, at which place the processor enters a program or a code fragment and execution begins.   
我们在谈到entry point时，通常针对的是源码；对于可执行文件，也有entry point的概念，这时则与OS的ABI（application binary interface ）有关，该entry point通常由compiler or linker生成；对于Non-executable object files，也存在entry point，其会被linker使用（当linker生成可执行文件的entry point时）。  

* 语言中的entry point  
	当前主流语言中，程序通常只有一个entry point，如C/C++/D中的main()，Java的静态方法main()，C#的静态方法Main()。对于脚本语言，则是从程序的起始处开始执行。  

C/C++的main()的四种原型，  
	int main(void);  
	int main();  
	int main(int argc, char **argv);  
	int main(int argc, char *argv[]);  
当然，也有OS特定的main()原型，比如Unix和Windows会接收第三个参数，用于指定环境变量，OS X则会包含第四个参数，用于指定特定的信息。

		rm file  
		argc = 2 and argv = {"rm", "file", NULL}  
C/C++中，main()必须在全局命名空间中，可被外部链接（即不能为static或inline），也不能是递归函数。

However, some languages can execute user-written functions before main runs, such as the constructors of C++ global objects.


* 可执行文件中的entry point  
	ELF文件（Unix/Unix-like OS中使用），由头部的e_entry域指定。对于PE/COFF文件（Windows使用），由AddressOfEntryPoint域指定。  

* 非可执行文件的entry point  
	gcc的linker，使用_start符号指定；COM文件，在地址0x100处。  

特殊情况，Android系统没有单一的entry point，而是activity和service组件，系统通过对这些组件的调用来启动app。  

* 其它  
	有一种技术成为<font color='red'>fat binary（待了解）</font>，在单一的binary文件中，可以被多个目标系统执行，一般是通过一个单一的全局entry point，再根据目标系统的不同，选择一个target-specific entry point。还有另一种替代形式，在多个fork调用中，执行不同的storing separate executables，根据目标OS，进行不同的fork  
* exit point  
	通常来说，不会有单一的exit point。然而，runtimes总是力图使程序在一个单一exit point结束，这样以便进行清扫工作。达到这种要求，可以有如下途径，包括main()返回，调用一个指定的退出函数，使用OS的异常和信号等。当然，runtime自身若crashed，则还是无法保证从单一exit point结束。  





###Memory protection
用以控制计算机内存的访问权限，已成为现代处理器架构和操作系统架构不可或缺的一部分。内存保护的主要目的是阻止进程访问未分配给它的内存区域。对于违法的访问，将会导致硬件错误（hardware fault：  a segmentation fault or storage violation exception），该进程也会被异常终止。

###Instruction set simulator
It mimics the behavior of a mainframe or microprocessor by "reading" instructions and maintaining internal variables which represent the processor's registers.  

使用ISS的三个原因：  
（1）为了模拟其它种类的硬件设备（本机器的指令集当然不需要simulate了）或者向上兼容（比如586兼容8086的指令集）  
（2）在同一设备上，for test and debugging purposes，需要对指令执行情况进行监视。例子，memory protection  
（3）硬件设计的需要，具体可参考wiki  

ISS通常会提供一个debugger，一般会整合 timers, interrupts, serial ports, general I/O ports, etc. 以便模拟microcontroller 的行为。  

仿真器的原理：  
首先执行monitoring program ，并将target program的名称作为参数传递给monitoring program；  
target program载入到内存之后，控制权并不会传递给target program，Instead, the entry point within the loaded program is calculated, and a pseudo program status word (PSW) is set to this location. A set of pseudo registers are set to what they would have contained if the program had been given control directly.  

###Object file  
An object file is a file containing object code, meaning relocatable format machine code that is usually not directly executable.   

In addition to the object code itself, object files may contain metadata used for linking or debugging, including: information to resolve symbolic cross-references between different modules, relocation information, stack unwinding information, comments, program symbols, debugging or profiling information.

###Debugger  
debugger使被调试的程序运行于instruction set simulator (ISS)，这样当特定条件满足时，可以暂停运行。

###Debugging data format  
A debugging data format is a means of storing information about a compiled computer program for use by high-level debuggers.  
Some object file formats include debugging information, but others can use generic debugging data formats such as stabs（已基本不用） and DWARF.

###Interrupt
系统级程序中，中断是软件或者硬件发送给处理器的一个信号，以尽快引起处理器的注意。相对于处理器正在执行的程序，中断表明有更高优先级的事情需要处理器去处理。处理器会暂停当前的程序，保存现场，通过调用中断函数（interrupt handler or an interrupt service routine, ISR）去进行处理。  

* 硬件中断  
	外部的硬件设备传递处理器并需要OS处理的信号。硬件中断是异步的，也就是说中断处理的过程中可能新的中断发生。硬件中断一般用IRQ（interrupt request ）表示。  

* 软件中断  
	处理器出现exception/trap（如除数为0），或者执行了会产生中断的指令，这时会出现软件中断。

无论硬件中断还是软件中断，都有其对应的interrupt handler，硬件中断的数量受限于处理器的IRQ lines，但是软件中断的数量则没有限制。中断常用语多任务处理（Computer multitasking）。  

中断还可以细分为如下类型：  
（1）Maskable interrupt (IRQ)  
（2）Non-maskable interrupt (NMI)：NMIs are used for the highest priority tasks such as timers, especially watchdog timers.  
（3）Inter-processor interrupt (IPI)：多处理器系统，某进程来中断其它进程。  
（4）Software interrupt:  
（5）Spurious interrupt: a hardware interrupt that is unwanted（如硬件干扰）  

###Thread
a thread of execution is the smallest sequence of programmed instructions that can be managed independently by a scheduler.  
在单核系统中，多线程一般通过time slicing实现，即频繁的切换线程，以使得用户认为程序在running in parallel.  
在多核系统（multiprocessor or multi-core system）中，多线程可以被并行执行（execute in parallel）. Multithreading can also be applied to one process to enable parallel execution on a multiprocessing system.  

a process is a unit of resources, while a thread is a unit of scheduling and execution.Resources include memory (for both code and data), file handles, sockets, device handles, windows, and a process control block.

Scheduling can be done at the kernel level or user level, and multitasking can be done preemptively or cooperatively(relies on the threads themselves to relinquish control once they are at a stopping point).   
Cooperatively scheduled user threads are known as fibers; 

User threads may be executed by kernel threads in various ways (one-to-one, many-to-one, many-to-many).

 Kernel threads do not own resources except for a stack, a copy of the registers including the program counter, and thread-local storage (if any), and are thus relatively cheap to create and destroy.Thread switching is also relatively cheap: it requires a context switch (saving and restoring registers and stack pointer), but does not change virtual memory and is thus **cache-friendly** (leaving TLB valid). The kernel can assign one thread to each __logical core__ in a system (**because each processor splits itself up into multiple logical cores if it supports multithreading, or only supports one logical core per physical core if it does not**), and can swap out threads that get blocked. However, kernel threads take much longer than user threads to be swapped.


###[Concurrent computing][Concurrent_computing_url] 

* main challenge  
	The **main challenge** in designing concurrent programs is concurrency control: ensuring the correct sequencing of the interactions or communications between different computational executions, and coordinating access to resources that are shared among executions.

* Models  
	<font color='red'>几种模型，待看</font>  

* Implementation  
	可以将每一个计算单元设计为一个进程，也可以将其设计为同一进程下的一个线程。  
	在有一些系统中，各计算单元之间的联系对程序员是隐藏的，例如使用future；但是有一些必需要程序员来进行处理，即使用的显式方案：  
	（1）Shared memory communication  
		这需要使用到锁，若一个程序正确的进行了设置，则称其为thread-safe  
	（2）Message passing communication  
		计算单元通过进行消息交换来通信，

* Race condition  
	the output is dependent on the sequence or timing of other uncontrollable events.  
	
* Concurrency control  
	<font color='red'>待细看</font>

* Dekker's algorithm  
	

###Undefined behavior  
语言规范中没有定义，This happens when the translator of the source code makes certain assumptions, but these assumptions are not satisfied during execution. UB会导致结果不可预测，因此会有难以发现的bug.   
例如，在《C++ Primer》(5th, EN)中提到，通过const\_cast来将变量的const属性去掉，并对其进行写操作，这种行为是未定义的。也就是说，编译器在优化时，可能直接将写操作忽略了，而这种优化行为是合规的。UB的wiki举了一个给int赋值overflow的例子，也能揭示UB的危险。  
<font color='red'>未看完</font>


###Lock
a lock or mutex (from mutual exclusion) is a synchronization mechanism for enforcing limits on access to a resource in an environment where there are many threads of execution. A lock is designed to enforce a mutual exclusion concurrency control policy.  

* 类型  
	* advisory locks  
		在获得锁之前，查询the corresponding data，以确定锁是否被占用；其实，这种情况下，即使没有获取到锁，也能够访问a locked resource，其实，约束在于最后产生的结果是否正确。也就是说，若不考虑结果正确与否，你在未获取到锁时，依旧能够操作a locked resource.这与下一种类型是截然不同的。  
	* mandatory locks  
		attempting unauthorized access to a locked resource will force an exception in the entity attempting to make the access.  
		


"test-and-set", "fetch-and-add" or "compare-and-swap".  
The most common strategy is to standardize the lock acquisition sequences so that combinations of inter-dependent locks are always acquired in a specifically defined "cascade" order.

* 锁的粒度  
	在设计粒度时，需要考虑如下三个方面  
	（1）lock overhead   
		在使用锁时所需要的额外资源（内存、CPU获取和释放锁）。  
	（2）lock contention  
		锁的粒度越小，竞争就越少  
	（3）deadlock  
	

* Database locks  
	用于同步transaction，

#参考  
[MPI_wiki]:https://en.wikipedia.org/wiki/Message_Passing_Interface  
[Concurrent_computing_url]:https://en.wikipedia.org/wiki/Concurrent_computing