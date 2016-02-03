# C++ Concurrency in Action  
Anthony Williams, 2012 MANNING Publications Co.
##Chapter 01 Hello, world of concurrency in C++
C++11开始支持多线程编程，也是通过提供library的方式来实现（provide components in
the library for writing multithreaded applications），从语言层面解决也增加了可移植性。  
本书主要是讲述以多线程方式来实现concurrency。
###1.1 What is concurrency  
关于concurrency的定义，请参考CSAPP第12章的笔记。
####1.1.1 Concurrency in computer systems  
单个系统同时执行相互独立的多个任务，这称之为concurrency，比如浏览网页时听歌。
  
concurrency is achieved through task switching or by genuine hardware concurrency. task switching会有上下文切换开销，有时候称假的concurrency（an illusion of concurrency）；使用genuine hardware的concurrency一般称为Parallelism，称为真的concurrency（hardware concurrency）. 

hardware threads：OS通过硬件方式能够同时执行的线程数。
####1.1.2 Approaches to concurrency  
两种方式，以多进程（每个进程只有一个线程）和多线程（一个进程中多个线程）方式来实现并发，当然，也可以是两种基本方式的组合，比如多个进程，而有的进程中又有多个线程。  

* CONCURRENCY WITH MULTIPLE PROCESSES  
	多个进程之间通过IPC通信，包括信号、socket、files和pipes等，通信方式较为复杂并且有可能速度不快；多个进程的overhead较高。优点是容易写出安全的concurrency程序，此外，可以在不同机器上运行不同的process，各process之间可以通过网络通信。  
	C++语言对多进程的通信并没有进行语言级别（内在）的支持，因此，这可能就需要用到特定平台的API了。

* CONCURRENCY WITH MULTIPLE THREADS  
	通过在一个进程中创建多个thread，每一个thread执行不同的逻辑流，但所有threads共享the same address space和数据（全局变量、指针和引用）。The shared address space and lack of protection of data between threads，使得overhead较小。但是，共享的数据的读写需要在写程序时注意。
###1.2 Why use concurrency?  
两个理由：separation of concerns and performance.
####1.2.1 Using concurrency for separation of concerns  
比如UI线程和工作线程，两个线程关注点不一样。  
the interactions between threads can be limited to clearly identifiable
points. （这样描述线程间通信有点简单吧？）  
在这种情况下，线程数量与CPU核心数量是无关的，	因为这是基于conceptual design而不是为了提高throughput。  
####1.2.2 Using concurrency for performance  
有两种方式：  
（1）将单个任务进行分块处理（task parallelism）  
将一个任务分成多个部分，各部分并行运行（run each in parallel）。  
实现方式又分两种，①. 将处理过程（算法）分为不同部分，各部分创建单独的线程进行执行；②. 每个线程的处理过程相同，但是处理的数据不同（也称data parallelism）。  
（2）通过并行解决更大的问题  
对多个数据集进行同样的操作，这点与data parallelism有点相似.
####1.2.3 When not to use concurrency  
讲述了多线程情况下可能的问题，总之，有利有弊，利大于弊时才会使用。
###1.3 Concurrency and multithreading in C++  
从C++11开始才提供对多线程程序的支持。  
####1.3.1 History of multithreading in C++  
C++98的多线程需要compiler-specific extensions才行。

##Chapter 02 Managing threads  
通过std::thread对象对thread进行管理。
###2.1 Basic thread management  
C++程序至少会有一个线程，也就是main()所在的线程。
####2.1.1 Launching a thread  
通过创建std::thread对象来创建线程。

	std::thread my_thread(do_some_work);
do\_some\_work可以是函数指针，也可以使定义了operator()的class的object，这种class也称为callable type.

	/* 这种创建方式是不正确的，编译器会将class_name理解为函数名称，而不是类名称。 
		my_thread则被视为一个函数，返回std::thread类型的object	*/
	std::thread my_thread(class_name());
	/* 正确的两种形式 */
	std::thread my_thread((class_name()));
	std::thread my_thread{class_name()};
在启动一个线程后，可以通过my\_thread.join()（等待线程执行完毕）或my\_thread.detach()（不管线程的执行情况）的方式来对其进行控制，如这两种方式都不指定，在my\_thread对象被析构时，会调用std::terminate()来终止其开启的线程。  
若我们使用my\_thread.detach()的方式，那么要确保开启的线程所访问的数据要一直有效（在该线程结束之前）。这有点类似顺序程序中，不要访问失效的局部引用。在多线程程序中，这种错误更容易犯，且不易察觉。如P18的例子。  
####2.1.2 Waiting for a thread to complete  
通过调用my\_thread.join()，可以等待开启的线程一直执行完毕，父进程此时会进入阻塞状态。对于一个给定线程，只能调用一次join()，调用之后便不能joinable,故join之前需要进行joinable的检查。<font color='red'>关于join() 的cleans up之类没看懂。</font>  
####2.1.3 Waiting in exceptional circumstances  
父线程需要等待子线程结束时，在父线程代码中的每一个出口，包括正常的return和异常的catch，需要调用join()函数。文中开始举了一个在每个出口都添加join()这种比较臃肿的解决方式。后来，通过采用RAII方式，增加一个类，在其析构函数调用join()，而析构时便是父进程退出main()时。  
####2.1.4 Running threads in the background  
detach()调用，表示开启的线程在后台运行（也称daemon threads），对该线程的拥有和控制的权限移交到C++ Runtime Library，以便该线程结束时回收相应资源。detach()也需要在joinable为true时才可以调用。  
After the call completes, the std::thread
object is no longer associated with the actual thread of execution and is therefore no longer joinable. <font color = red>此句中的associated是与哪个进程，父或者子？</font>  
文中举的打开文件编辑的例子貌似有问题，新创建的线程t不是又立即执行了吗？这样又弹出打开界面？<font color = red>此处有疑问</font>  
###2.2 Passing arguments to a thread function  
需要记住参数是被拷贝到新创建的线程环境再进行处理的（the arguments are copied into internal storage）。  
对于隐式转换，记住是在新的线程环境中进行的，对比下面例子：

	void f(int i,std::string const& s);
	/* 对于C++的隐式转换，是在新的线程环境中执行的，这里只是copy */
	/* 此处copy的是char const*，转换为std::string */
	std::thread t(f,3,”hello”);
	/* 此处的buffer是指向局部变量，因此可能会有UB发生 */
	char buffer[1024];
	std::thread t(f,3,buffer);
	/* 正确实现 */
	std::thread t(f,3,std::string(buffer));
对于引用，会有如下可能性发生：父线程想通过创建子线程来处理数据，子线程在处理完成之后更新这个数据到父线程，因此，用到了引用。下面代码列出了一种错误写法，同时给出了正确实现。
	
	void update_data_for_widget(widget_id w,widget_data& data);
	widget_data data;
	/* 注意copy到新线程后，新线程再对这个data进行引用 */
	/* 所以修改后的data是无法更新到父线程的 */
	std::thread t(update_data_for_widget,w,data);
	/* 正确实现 */
	std::thread t(update_data_for_widget,w,std::ref(data));
对于类成员函数的传递，因为其第一个形参是this，故可以这样调用成员函数：

	std::thread t(&X::do_lengthy_work,&my_x);/* X my_x; */	
有些参数不能够copy，而只能够move，比如std::unique_ptr，需要调用std::move()，

	void process_big_object(std::unique_ptr<big_object>);
	std::unique_ptr<big_object> p(new big_object);
	p->prepare_data(42);
	/* 对于具名对象，需要调用std::move()，而临时对象则不需要 */
	std::thread t(process_big_object,std::move(p));
现在对C++11不熟悉，关于std::unique_ptr还需要回头看这段。<font color='red'>需要回头看</font>
std::thread和std::unique_ptr有相同的ownership语意，前者拥有管理线程执行的资源，而后者指向一个动态对象，都可以moveable，但不能copyable。一个std::thread object同一时间只能关联一个线程。
###2.3 Transferring ownership of a thread  
转移例子：

	void some_function();
	void some_other_function();
	/* 创建新线程 */
	std::thread t1(some_function);
	/* 将线程的Ownership从t1转给t2，注意此处要用std::move */
	std::thread t2=std::move(t1);
	/* 创建新线程并转移，注意此处是临时变量，故可不用std::move，这种转移是自动的 */
	t1=std::thread(some_other_function);
	std::thread t3;
	t3=std::move(t2);
	/* t1之前就关联了一个线程，现在再接收的话，会调用std::terminate()终止程序 */
	t1=std::move(t3);
从函数内转移到函数外：

	std::thread f()
	{
		void some_function();
		return std::thread(some_function);
	}
从函数外转移到函数内：  

	void f(std::thread t);
	void g()
	{
		void some_function();
		/* 临时对象 */
		f(std::thread(some_function));
		std::thread t(some_function);
		/* 具名对象 */
		f(std::move(t));
	}
利用std::thread的move特性，可以改写2.1.3节中的RAII类，将std::thread拥有的线程转移到RAII类中。具体参考listing 2.6.  
listing2.7中，对于创建的多个线程进行join()时，<font color='red'>难道不阻塞吗？join()到底阻塞不阻塞？</font>  
###2.4 Choosing the number of threads at runtime  
std::thread::hardware_concurrency(),可以获取能真正并发的数量，一般是返回CPU核心数，若相关信息无法获取，也可能返回0值。  
listing 2.8中，the addition operator for the type T is not associative（结合律），因此并行计算值可能有差异。线程无法返回计算结果，此例中是传递引用解决，后面章节会介绍通过futures的方式。
###2.5 Identifying threads  
线程标识类型为std::thread::id，可以通过两种方式获取。  
通过调用std::thread object的get\_id()成员函数，若该object没有关联线程，则会返回a default constructed std::thread::id object. 在当前线程中，也可以通过调用std::this\_thread::get_id()来获取类型。  
std::thread::id对象可以被复制和比较（可以排序），若两者相等，则是同一个线程，当然，也可能都是default id（即不表示任何线程）。

##Chapter 03 Sharing data between threads  
###3.1 Problems with sharing data between threads  
问题源于数据可能被部分线程修改。
对于代码的分析，我们引进一个概念，invariants（不变式），该invariants可能会在多线程修改过程中被破坏。
####3.1.1 Race conditions
In concurrency, a race condition is anything where the outcome depends on the
relative ordering of execution of operations on two or more threads;the threads
race to perform their respective operations.（也就是多个线程执行顺序不定，而又可能修改同一数据）  
一般我们所称的Race conditions为problematic race conditions，即可能导致程序错误。  
data race会造成undefined behavior.  
Problematic race conditions一般发生在一个操作需要修改两处或者以上的数据时，非常难以查找和重现，通常是timing sensitive，也就是执行次数很多时才会出现，这样debug的时候便不容易发现。
####3.1.2 Avoiding problematic race conditions  
（1）对共享数据加入保护机制，确保在修改时只有一个修改线程使用它  
（2）修改数据结构，使得对invariants的操作，每一步都是原子的，且每一步修改之后都是invariants。这也通常称为lock-free programming.当然，这种方式会使得程序变得复杂。  
（3）利用事务修改共享数据，这种方式称之为software transactional memory(STM)，目前处于研究阶段，C++也没有直接支持。
###3.2 Protecting shared data with mutexes  
####3.2.1 Using mutexes in C++  
通过std::mutex对象创建一个mutex，其有成员函数lock()和unlock()。mutex最好通过RAII的方式使用，C++为此专门提供了std::lock_guard。  
对于类中需要保护的数据，将其设为private，并确保所有成员函数在访问前对其lock()，这样可以避免race conditions。当然，若成员函数返回了保护数据的指针或引用(或者通过输出参数输出了指针或引用)，以及成员函数调用了其它函数，而这些函数又不受调用者控制（比如这些函数将指针保存等着后续使用），则保护进制会出现漏洞（a big hole）。因此，程序设计时需要仔细考虑。
####3.2.2 Structuring code for protecting shared data  
本节的Listing 3.2中，说明了调用一个用户函数时，传递了保护数据的指针，结果导致保护机制破坏。文中指出了避免这种情况的设计原则：  
Don’t pass pointers and references to protected data outside the scope of the lock, whether by returning them from a function, storing them in externally visible memory, or passing them as arguments to user-supplied functions.
####3.2.3 Spotting race conditions inherent in interfaces  
考虑这种情况，stack的数据都通过mutex进行了保护，多个线程调用其如下函数：

	stack<int> s;
	if(!s.empty())
	{
		int const value=s.top();
		s.pop();
		do_something(value);
	}  
我们知道，在空栈上进行top()操作是未定义的。试想，若一个线程刚调用了empty()进行检查，结果为非空，那么其会继续执行top()操作，而在top()之前，另一个线程进行了处理使得stack现在为空，那么这就会出现未定义行为。  
上述问题如何解决？因为是接口引起，因此需要对接口进行修改。有一种方式调用top()若stack中没有元素，则抛出异常（也就是将刚才的UB改为异常），但考虑到会使用try...catch，会导致代码臃肿。  
再考虑一种可能，若两个线程分别依次调用了top()，那么其处理的元素是一样的，随后两个线程又依次调用了pop()，这样两个元素就被弹出来了，但却只有一个元素被处理，这就隐藏了一种很难察觉的race condition.

考虑如下问题，在类型为stack<vector<int\>\>的堆栈中，当你复制一个vector<int\>元素时，系统需要分配空间，若分配空间失败，则可能会抛出std::bad_alloc异常。若pop()需要返回弹出的元素，返回元素肯定是发生在stack被修改后（即该元素被remove），返回元素会涉及到copy操作，那么，若copy发生std::bad\_alloc异常，而此时stack中的元素又被remove，这样会使得数据丢失。这也是为什么设计了pop()和top()接口，pop()只弹出元素，而top()返回的原因，如此，二者可各司其职。  

对于上述问题，若不适用top()，即只定义pop()，下面从几个方面探讨了解决途径：  
  
* OPTION 1: PASS IN A REFERENCE  
	修改pop()接口，使其可以引用传值。这必须要调用代码创建一个对象，而当创建对象成本太高时或者根本就无法创建对象时（如构造函数所需参数无法得到），就不可行了。此外，该方式需要对象能赋值（support assignment），若不能赋值也不可行。

		std::vector<int> result;
		some_stack.pop(result);
* OPTION 2: REQUIRE A NO-THROW COPY CONSTRUCTOR OR MOVE CONSTRUCTOR  
	如题，但具体原因未懂，因为C++11还没怎么看，不抛异常就行？<font color='red'>此处有疑问</font>
* OPTION 3: RETURN A POINTER TO THE POPPED ITEM  
	指针的拷贝很容易，且不会抛异常，但需要对stack中每一个元素单独分配内存（这回有额外的overhead，相对于整体来使用内存），对于int等内置类型或者简单的类型来说，管理的代价可能要超过进行值返回的代价。此外，这也需要考虑到内存泄露问题，不过这可以用std::shared_ptr解决。
* OPTION 4: PROVIDE BOTH OPTION 1 AND EITHER OPTION 2 OR 3  
	就是若能提供2或3，那么也就能提供1，也最好将1提供，这样方便用户。  

锁的粒度要控制得当，比较极端的一种情况是，全部的共享数据用一把global锁，这会使得性能下降，Linux kernel的第一个版本就是这样做的。  
One issue with fine-grained locking schemes is that sometimes you need more
than one mutex locked in order to protect all the data in an operation. 
但多个mutex会造成Deadlock问题。  
####3.2.4 Deadlock: the problem and a solution  
避免deadlock的基本原则：两个线程以相同顺序加锁。  
考虑一种情况： 若某class的instance使用了一个mutex，而swap()交换该class的两个instance，
若两个线程分别以这两种形式swap(A, B)和swap(B, A)调用，则也会发生deadlock，即调用参数互换位置。不过，针对此种情况，C++供应了std::lock，使用时注意判断两个参数不是同一个instance，因为对一个instance调用两次lock()会UB，std::lock provides all-or-nothing semantics with regard to locking the supplied mutexes.（类似transaction，要么有要么无）
std::lock可以在获取两把或者以上的锁时使用，但若每把锁都需单独获取，则不能使用，这种情况下只能凭经验以防deadlock。
####3.2.5 Further guidelines for avoiding deadlock  
Deadlock主要发生在lock的时候，当然，也有其它情况，you can create deadlock with two threads and no locks just by having each thread call join() on the std::thread object for the other.（<font color='red'>这怎么实现？</font>）  
原则： don’t wait for another thread if there’s a chance it’s waiting for you.  
下面一些方法可以防止别的线程在等待你当前的线程：  
（1）AVOID NESTED LOCKS  
don’t acquire a lock if you already hold one. 若一次需要获取多把锁，请用std::lock()。  
（2）AVOID CALLING USER-SUPPLIED CODE WHILE HOLDING A LOCK  
因为用户代码是否用锁我们并不知道。但是，在写泛型程序时，有些操作会交由用户定义，这时，就需要另说了。  
（3）ACQUIRE LOCKS IN A FIXED ORDER  
若要获取多把锁，而又不能使用std::lock()，那么，在每一个线程中将它们按同样顺序获取。  
（4）USE A LOCK HIERARCHY  
该方式算一种特殊lock ordering。具体做法如下：对代码分层，所有的mutex有一个层次属性（可以用正整数表示），若某一层的mutex已经被锁，只有层数值小于它的mutex才可以被锁。比如10的mutex被锁，那么你可以接着锁5的mutex，而不能锁90的mutex。当然，这也限制了一层只能用一把锁，因为一层若有多把锁，也就是值相同，那么在这一层中的锁，其先后顺序就无法规范了。  
（5）EXTENDING THESE GUIDELINES BEYOND LOCKS  
对上面几点的总结
####3.2.6 Flexible locking with std::unique_lock	
std::unique\_lock比std::lock_guard更灵活，其并不总是拥有其关联的mutex。std::unique\_lock比std::lock\_guard占用更大空间，且性能也稍差，因为std::unique\_lock记录了锁的状态，而状态值需要维护更新。若std::lock\_guard满足要求，那就优先使用std::lock\_guard。
####3.2.7 Transferring mutex ownership between scopes  
One possible use is to allow a function to lock a mutex and transfer ownership of
that lock to the caller.   
std::unique_lock在析构前，也可以其释放关联的锁，通过调用其unlock()实现。如此设计的原因：The ability to release a lock before the std::unique\_lock instance is destroyed means that you can optionally release it in a specific code branch if it’s apparent that the lock is no longer required.
####3.2.8 Locking at an appropriate granularity  
持有锁的原则：In general, a lock should be held for only the minimum possible time needed to perform the required operations.  
注意listing3.10，在程序调整后，the semantics of the operation改变了，这种分析需要学习。    
if you don’t hold the required locks for the entire duration of an operation, you’re exposing yourself to race conditions.  
###3.3 Alternative facilities for protecting shared data  
####3.3.1 Protecting shared data during initialization  
只有初始化时需要保护，之后不需要。  
Double-Checked Locking pattern：

	if(!resource_ptr)/*锁之外读取*/
	{
		std::lock_guard<std::mutex> lk(resource_mutex);
		if(!resource_ptr)
		{
			resource_ptr.reset(new some_resource);/* 创建 */
		}
	}
	resource_ptr->do_something();/* 使用 */
A线程在锁之外读取resource\_ptr，会造成race condition，因为，此时若B线程在创建过程中，有可能resource\_ptr指针不为空，但new some\_resource却没有完成，这样，在使用some_resource会出现错误的结果。
  
针对上述问题，C++提供了std::once\_flag和std::call\_once.  
调用std::call\_once之后，在该函数返回时，便已经完成了初始化操作。使用std::call\_once比直接使用mutex开销要小，特别是初始化工作先前就被完成的情况下。  
当使用局部静态变量时，其初始化过程也可能导致race condition。在C++11之前，这个问题确实是存在的。但是在C++11中，对于局部静态变量的初始化确保只在一个线程里进行，并且其它线程只有在初始化完成后才能使用。  

####3.3.2 Protecting rarely updated data structures  
使用读写锁，目前只有boost支持。对于update的线程，使用std::lock\_guard<boost::shared\_mutex> 和 std::unique\_lock<boost::shared\_mutex>；对于read的线程，使用boost::shared\_lock<boost::shared\_mutex>。
####3.3.3 Recursive locking  
不推荐使用[Reentrant mutex][Reentrant_mutex_url]  


##Chapter 04 Synchronizing concurrent operations
有时候不光是要同步数据，也需要同步操作（synchronize actions），后者类似于线程间的观察者模式。C++11以condition variables和futures对此进行了支持.
###4.1 Waiting for an event or other condition  
####4.1.1 Waiting for a condition with condition variables  
C++标准库提供了两个条件变量：std::condition_variable和std::condition_variable_any，两者都需要mutex才能提供同步机制，前者只能使用std::mutex，而后者适用于anything
that meets some minimal criteria for being mutex-like. 因此，前者更轻量，故优先使用。  
在listing4.1中，准备数据的线程的代码如下：  

	data_chunk const data=prepare_data();
	std::lock_guard<std::mutex> lk(mut);
	data_queue.push(data);
	data_cond.notify_one();

处理数据的线程的代码如下，

	std::unique_lock<std::mutex> lk(mut);
	data_cond.wait(lk,[]{return !data_queue.empty();});
在准备数据的线程通过notify\_one()将处理数据的线程唤醒后，处理数据的线程首先获得mutex，其wait()检查条件，代码中的lambda表达式，若条件为真，则返回，并继续保持mutex的lock状态；若条件为假，则unlock mutex，并将线程恢复到等待状态。注意处理数据的线程用的锁是std::unique_lock，是因为the waiting thread must unlock the mutex while it’s waiting and lock it again afterward，而std::lock\_guard无法做到。  

从上面可知，wait()在条件为真时会立即返回。  
wait()中做条件检查的原因是，线程可能不是因为其他线程发送了条件变量信号而被唤醒（即notify\_one()），  而是其它情况被唤醒（称为假醒，[spurious wake][Spurious_wakeup_url]），若不做条件检查，而wait()直接返回的话，此时进行数据处理可能会出错，因此，通过一个条件来与发送信号的线程进行关联，只有该条件为真了，证明此时可以返回并正确的进行数据处理。因为wait()可能被调用多次（包括假醒时），因此，检查条件的函数最好不要有副作用。  

The flexibility to unlock a std::unique_lock isn’t just used for the call to wait();it’s also used once you have the data to process but before processing it.因为数据处理时间较长，<font color='red'>这种锁还需要看文档，回头再更新</font>。
####4.1.2 Building a thread-safe queue with condition variables  

##参考  
[Reentrant_mutex_url]:https://en.wikipedia.org/wiki/Reentrant_mutex  
[Spurious_wakeup_url]:https://en.wikipedia.org/wiki/Spurious_wakeup

