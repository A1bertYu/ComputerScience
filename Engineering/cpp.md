* 构造函数  
	对于某类，用其构造函数A调用构造函数B，即使B中对某数据成员（非静态）进行了初始化，但最终还是无效的。  
* QT中的数据库驱动  
	在Windows 7环境下，使用了QT的MySQL驱动，那么必须在程序所在目录创建plugins/sqldrivers目录，里面放置驱动才可以正常使用MySQL。当然，通过QCoreApplication::addLibraryPath ( const QString & path )可以添加更多路径，但是，要在路径下创建文件夹sqldrivers并放置相应驱动才行（必须要有这个sqldrivers名称的文件夹）  

* VS的C++编译器文档  
https://msdn.microsoft.com/en-us/library/958x11bc.aspx  
* 堆和栈的区别  
	**The stack** is the memory set aside as scratch space for a thread of execution.The stack is always reserved in a LIFO (last in first out) order;This makes it really simple to keep track of the stack; freeing a block from the stack is nothing more than adjusting one pointer.   
	**The heap** is memory set aside for dynamic allocation. 堆的分配和释放并没有确定的规则（栈是后进先出），因此，跟踪堆的使用是很复杂的。因此，堆分配的方式（usage patterns）对性能是有影响的，这也使得存在很多malloc策略，比如google的tcmalloc等。综上，可得知stack更快。  
	Each thread gets a stack, while there's typically only one heap for the application (although it isn't uncommon to have multiple heaps for different types of allocation).  
	The OS allocates the stack for each system-level thread when the thread is created. Typically the OS is called by the language runtime to allocate the heap for the application.  
	The stack is attached to a thread, so when the thread exits the stack is reclaimed. The heap is typically allocated at application startup by the runtime, and is reclaimed when the application (technically process) exits.  
	The size of the stack is set when a thread is created. The size of the heap is set on application startup, but can grow as space is needed (the allocator requests more memory from the operating system).
	

	参考链接http://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap  

###Extensions to the C Language Family  


####GCC关于汇编的扩展（在C语言中使用汇编）  

有两种形式，basic asm和extended asm。
	
* Basic Asm — Assembler Instructions Without Operands  
	
		语法格式
		asm [ volatile ] ( AssemblerInstructions )

	asm表示GNU扩展，若是ANSI格式，则使用\_\_asm\_\_；All basic asm blocks are implicitly volatile，故对于basic asm，可选项无作用； GCC对汇编语句不会解析也不会进行有效性判断，这是后续assembler的事情；在一条asm语句中，可以放置多条汇编语句。

basic asm存在的理由：  
（1）在函数外面编写汇编语句时，必须使用basic asm；此种需求一般用于<font color='red'>emit(https://msdn.microsoft.com/en-us/library/1b80826t.aspx, http://cs.mtu.edu/~mmkoksal/blog/?x=entry:entry120116-130037) assembler directives，暂时没有找到emit资料，参考msdn帮助</font>   
（2）Functions declared with the naked attribute also require basic asm.（只针对ARM等非x86/x64的平台，不考虑）  

basic asm中的汇编语句不能够修改通用寄存器，但是可以读写全局变量（any globally accessible variable）。

* Extended Asm - Assembler Instructions with C Expression Operands  
	
		语法格式
		asm [volatile] ( AssemblerTemplate
                      : OutputOperands
                      [ : InputOperands
                      [ : Clobbers ] ])
     
     	asm [volatile] goto ( AssemblerTemplate
                           :
                           : InputOperands
                           : Clobbers
                           : GotoLabels)
		从C程序中获取数据时，不通过上述格式所提供的输入输出方式，而是通过类似于使用全局变量（或者调取某个函数）可能出现问题，如此使用时，请注意ABI
		若asm语句有输出变量，当未使用volatile时，GCC优化可能会将代码移出循环外（若一直返回相同的结果，比如在调用时输入一直不变）。若不想GCC做这些优化，请使用volatile。当然，也有些情况即使使用了volatile，GCC也可能做优化，这时就需要额外的一些处理技巧了。

		//PowerPC example
		asm volatile("mtfsf 255, %0" : : "f" (fpenv));  //does not work reliably
		asm volatile ("mtfsf 255,%1" : "=X" (sum) : "f" (fpenv));//work as expected
 	* Assembler Template  
		包含token的汇编语句，这些token可以是输入、输出和goto标签。

	* C语言中的输入输出变量在汇编语句中的使用  
		在汇编语句中，可以使用'%[variableName]'的形式来对变量进行引用，也可以使用该形式'%0'来按顺序对变量进行引用（从输出变量开始，第一个为'%0'）。  
	  	对于输出/输入变量，其格式为[ [asmSymbolicName] ] constraint (cvariablename)：   
		（1）asmSymbolicName若没有（也就是汇编中用的数字），则可以省略
		（2）输出变量的constraint，前缀必须是'='或'+'，'='表示要写，而'+'则会读取并修改该变量，在使用'='时候，我们不能做该变量已经被初始化的假定（要知道，用'='只是表明我们会覆盖该值）；前缀之后，'r'表示register（将该变量放入register），'m'表示memory，更多的constraint选项可以参考文档，使用时对照查阅。

		
	* 特殊符号  
		'%%'，输出单个'%'  
		'%='，输出一个整数值，对每一个asm statement来说，其具有一个该值，且唯一。
		‘%{’，‘%|’，‘%}’,对'{','|','}'这些特殊字符的转义，这三个字符是GCC用来支持编写多个dialect都可以使用的语句的。

	* Clobbers  
		Clobber list items are either register names or the special clobbers (listed below). 主要用于向编译器表明asm code may modify more than just the outputs.（编译器只解析了输出操作数，汇编语言可能有额外的编译器不可知的修改）   
		（1）"cc"：汇编代码会修改flag寄存器；  
		（2）"memory"：汇编语言在内存中进行读写操作  

	* Goto Labels  
		goto 的asm没有输出，若要输出变量，请将colbbers设置为"memory"。汇编语句中也可以跳到goto asm语句中的labels，通过%l的形式（注意l是lower case "L"），若有3个输入变量，想要跳到第一个label，则在语句中使用'%l3'（从0开始，输入变量先占据，之后才是labels）  

	* x86 Operand Modifiers  
		在汇编语句中，引用变量以数值形式表示时，其前面加上不同前缀，表示不同意义，如：  
		%b0 表示 %ah(att)， ah(intel)    

		前缀为'0','1'...'9'时，表示该操作数匹配特定的位置，例如下面语句中的newval前面的'1'，表示其匹配的是'%1'

			asm volatile("lock; xchgl %0, %1" :
               "+m" (*addr), "=a" (result) :
               "1" (newval) :
               "cc");
		
	