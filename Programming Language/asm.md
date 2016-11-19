基于x86, intel style

##intel vs att
intel中，寄存器和立即数是没有前缀的，立即数若第一个是字母，必须最前面加0，且十六进制后缀为h，二进制后缀为b。  
att中，寄存器前缀为%，而立即数为$。  

intel中，第一个操作数为目的操作数，第二个操作数为源操作数；att刚好相反。  

intel中，对于内存操作数：  

		mov	eax,[ebx]			; intel
		mov	eax,[ebx+3]  		; intel
		movl (%ebx), %eax		; att
		movl 3(%ebx), %eax		; att
		; intel : [base+index*scale+disp]
		; att : disp(base,index,scale)
		; att is very obscure
att的助记符，需要根据操作数size来添加适当后缀，而intel不用。long("l"), word("w"), byte("b")

###register

关于寄存器，请参考[x86架构][x86_url].

对于flag寄存器的操作指令：POPF, POPFD, and POPFQ对应16/32/64bits，

* FLAG register  
	16bits，这些位包含了当前处理器的状态。  
	（1）**bit 0(CF)** ，status  
		carry flag，当ALU操作导致符号位（常说的最高位，most significant）有借位或者进位时。会影响CF的操作包括ADD/SUB, ADC/SBC (ADD/SUB including carry), SHL/SHR (bit shifts), ROL/ROR (bit rotates), RCR/RCL (rotate through carry)等。  
	（2）**bit2(PF)** ，status  
		Parity flag, 对于最后一次操作产生的结果，其二进制表示的位数中，为1的数量，若为奇数个则PF=0，偶数个则为1  
	（3）**bit4(AF)** ，status  
		Adjust flag(also known as the Auxiliary flag)，该标志位的设立是为了兼容所有x86的CPU，最低4位（低半个字节，lower nibble）若有进位借位产生，则置位。  
	（4）**bit6(ZF)** ，status  
		Zero flag，若arithmetic operation (including bitwise logical instructions)的结果为0，则ZF=1  
	（5）**bit7(SF)** ，status  
		sign flag(also known as Negative flag)，若结果为负，则SF=1，下列指令会对其有影响：  
		① All arithmetic operations except multiplication and division;  
		② compare instructions (equivalent to subtract instructions without storing the result);  
		③ Logical instructions - XOR, AND, OR;  
		④ TEST instructions (equivalent to AND instructions without storing the result).  

	（6）**bit8(TF)** ，control  
		trap flag置1时，可以使处理器进入单步模式，这样debugger可以控制程序的执行。  
		在单步模式下，CPU执行一条指令后会停止，8086没有直接设置TF的指令，而是通过将Flag寄存器的值先放入栈端，对栈进行修改后，再将修改后的值放回Flag寄存器。  
		从wiki上看到了Int3ServiceRoutine（应该是debugger会用到的），monitoring a program from an ISR.  

	（7）**bit9(IF)** ，Control  
		Interrupt flag，CPU是否去处理可屏蔽的硬件中断（handle maskable hardware interrupts），若IF=0，则CPU会忽略这些中断。IF对于不可屏蔽的硬件中断（non-maskable interrupts）或者有INT产生的软件中断无影响。  
			
			non-maskable interrupt (NMI)，通常用于对不可恢复的硬件错误产生信号，这种中断在系统运行时是不能禁用的，并且需要立即响应。
			NMI包括 internal system chipset errors, corruption in system memory such as parity and ECC errors, and data corruption detected on system and peripheral buses.
		
	CLI用于清除该位（立即生效），而STI用于设置该位（紧跟其后面的一条指令执行完毕之后才生效）。对于单核处理器，CLI通常用于同步机制，在CLI之后，kernel可以避免因为中断到来，进而通过interrupt handler引起的race condition。对于多处理器系统，CLI仅能影响执行它的processor，因此，其它processors仍能接收中断并通过interrupt handler进程处理，这样就可能引起race condition。这样，在多核系统中，需要CLI/STI结合其它同步机制来避免race condition.  
	CLI执行之后，再执行HLT会hang the computer，直至下一个中断发生。  
	若STI之后执行CLI，则处理器会一直接收中断但是不处理。  
	除了STI/CLI，也可以通过POPF进行设置。  
	
	（8）**bit10(DF)** ，Control  
		Direction flag，当拷贝多个字节的数据时，拷贝方向，特别是目标地址和源地址之间有重叠时。  
		cld and std用于clear和set DF
	（9）**bit11(OF)** ，Status  
		the overflow flag is "meaningless" and normally ignored when unsigned numbers are added or subtracted.  
	（10）**bit12-13(IOPL)** ，System  
		 I/O Privilege level is a flag found on all IA-32 compatible x86 CPUs


* EFLAGS  
	比FLAG多了16bits，即其包含32bits  

* RFLAGS  
	比EFLAGS多了32bits，即其包含64bits  

* GDTR  
	The Global Descriptor Table (GDT) is a table in memory that defines the processor's memory segments. The GDT sets the behavior of the segment registers and helps to ensure that protected mode operates smoothly.  
	GDT寄存器指向GDT，该寄存器有48bits，低16位用于表示GDT的大小（实际大小为该值加1），高32位则指向GDT的起始位置。

		lgdt [gdtr] ; load the GDTR  
	OS会为每一个程序分配不同memory segments，
			 
###指令

* INT  
	INT X  
	产生软件中断，X取值范围为0-255，

	interrupt address table，在real mode下载入，位于内存的0-1024bytes区域，因此，可以手动的通过far-call指令来调用中断函数。大多数Unix/Linux不用软件中断，除了0x80用于系统调用的软件中断外，其系统调用实现的过程：将系统调用的函数编号（整型）放入到EAX，之后再执行INT 0x80    
	INT 3  
	通常被debugger使用，用作设置断点。注意，INT 3被特别编码为了0xCC（只有一个字节），区别于INT immediate（编码为0xCD imm8，两个字节）<font color='red'>This makes them unsuitable for use in patching instructions (which can be one byte long);</font>

#参考资料

[x86_url]:https://en.wikipedia.org/wiki/X86