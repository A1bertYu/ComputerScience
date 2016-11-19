
#基本概念  
* Unix signal   



* Page   

	注意page和page frame的定义。

	A **page**, memory page, or virtual page is a fixed-length contiguous block of **virtual memory**, described by a single entry in the page table. Page是OS进行内存管理的最小单位。  

	A **page frame** is the smallest fixed-length contiguous block of **physical memory** into which memory pages are mapped by the operating system

	* page size的考量  
		与处理器架构有关，多数OS允许程序运行时指定。  
	（1）若Page size较小，则Page table中的PTE越多，如此，page table占用空间越大。  
	（2）Page size越大，PTE越少，TLB则命中率越高。  
	（3）Page Size越大，则浪费越多。对于程序用到的pages，最后一页不一定能够全部用上，这样会造成浪费。  
	（4）Page Size越大，从硬盘寻址所费时间越少。
	
	* page size的问题  
		假设内存地址空间为4GB，page size为4KB，一条PTE占用4bytes，那么page table的话就有4MB（每个进程一个page table，这样就占用4MB/进程！）  
		一般程序很少会用到4GB，是否可以考虑page table structres，以便节省空间？  
		**（1）Hierarchical paging**  
			以Two-level paging为例，也就是将上述一张Page table分为两张，类似于加法换乘乘法，例如寻址100个条目，用一个数组来存的话需要100个，而用两组10个来存则只需要20个。那么，上面的例子，可以以10位、10位、12位来寻址，最后12bits算offset（因此成为two-level），PTE相当于只存在于最后一段上，也就是占用2^12*4bytes的大小。  
		**（2）Hashed page table**  
			一般用于地址超过32bits时。Page table实现为hash table，每一个index包含a chain of elements hashing to the same location. hash函数的输入时vpn，搜寻的是index对应的chain.  
		**（3）Inverted page table**  
			每一个PTE对应实际的物理内存，PTE包含有vpn、访问权限和该PTE所属的PID等


* [Page table](https://en.wikipedia.org/wiki/Page_table)  
	page table是用于虚拟内存技术的一种数据结构，其存储了从虚拟地址到物理地址的映射关系（称为 paging or swapping）。如下图所示。  
	![Alt text](/virt_addr_space_and_phy_addr_space.png)  

	在使用了虚拟内存技术的OS中，每一进程使用的是连续且空间很大虚拟地址。实际上，分配给每一进程的物理地址可能是分散在不同的区域。当进程需要访问内存中的数据时，其给OS的是虚拟地址，这就需要OS将其转换为物理地址（数据存储的地方），而page table便是完成这一功能的硬件，page table中的每一条映射称之为page table entry (PTE).  
	
	具体的转换过程如下图所示：  
	
	![Alt text](/page_table_actions.png)  
	CPU中的MMU会缓存最近的映射记录，实现该功能的模块称为translation lookaside buffer (TLB)。因而，在进行映射时，首先肯定是搜寻TLB，若有则返回物理地址（结果）；若没有，则会遍历page table（称为a page walk），若找到，则将该映射关系写入到TLB，刚才失败的指令再执行一次（因为回写TLB，这次肯定能命中并返回结果）；若在page table中都没有找到，则原因有二：①虚拟地址不合法，没有可用的转换（会引发segmentation fault）；②若page没有在物理内存中（也就是page被换出到了backing store，也称为swap partition/swapfile/page file，**注意，PTE是存在的，只是物理内存没有该page**），这时就需要将page换回到物理内存中。从backing store换回page到物理内存时，会有如下操作，若是物理内存没有满，则该page直接写回物理内存，page table和TLB进行更新；若物理内存满了，这时就需要将其他page换回到backing store，这时，若page table和TLB中若有其他page的相应记录，则需要更新这些信息，当然，换到物理内存的page也需要更新，这个与没满时是相同的。  
	
	<font color='red'>有个疑问：上述的过程等价于说只要是合法的，映射关系肯定是存在在page table中，那么，这些关系是什么时候初始化的，如何初始化的？又如何更新，也就是说，总有满的时候，那么肯定要将一些PTE去掉，若是后续又用到了这些PTE，如何操作？</font>

	

	* Page table types  
		（1）Inverted page table  

* 系统调用  
	对于所有的系统调用，参数个数放置于%eax，对于少于6个参数的调用，其依次存放于%ebx,%ecx,%edx,%esi,%edi，系统调用的返回值存放于 %eax；对于大于等于6个参数的系统调用，参数存放于内存中，而内存地址存放于%ebx，


* Linearizability/atomic/indivisible/uninterruptible      
	 Atomicity is a guarantee of isolation from concurrent processes.要么执行成功，要么执行失败且对系统无影响。

#参考
