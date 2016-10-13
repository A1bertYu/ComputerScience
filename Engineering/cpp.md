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
	