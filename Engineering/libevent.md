##Programming with Libevent  
###A tiny introduction to asynchronous IO  
* 关于同步IO和异步IO的简单区别。  

* 每个线程处理一个连接  
	同步IO会导致阻塞发生，当服务器需要响应多个连接时，会造成延时问题。比较容易的一个解决方案是多线程（或者多进程）处理连接，每个连接对应一个线程（进程）。但是，创建新线程（或进程）开销太大，当然，若用线程池可以有所缓解。然而，在应对大量连接的业务场景时，每线程（或进程）处理一个连接的方案是非常低效的（也就是这种方案的处理能力并不是线性增长的）。  

* 非阻塞IO  
	可能会空转。比如在一个循环里不停的recv()，即使该fd是非阻塞的，但程序会在while里空转（spin indefinitely），此外，recv可是成本很高的系统调用哦。  
* select  
	在连接数多的时候性能也会出现问题，Because generating and reading the select() bit arrays takes time proportional to the largest fd that you provided for select(), the select() call scales terribly when the number of sockets is high.  
* select的后继者  
	poll(), epoll(), kqueue(), evports, and /dev/poll.   
	除了poll()，其它的都具有O(1) performance for adding a socket, removing a socket, and for noticing that a socket is ready for IO.  
	上述各接口都不统一，因此，为了统一接口，必须要新加一层抽象层。这就是libevent的作用，当然，抽象后也使得接口更易使用。  
###The Libevent Reference Manual: Preliminaries  
* libevent的特点  
	目的：for writing fast portable **nonblocking IO**  
	Portability  
	Speed  
	Scalability  
	Convenience: 写出的程序是the stable, portable way.  
* libevent的components  
	evutil    
	event and event_base  
	bufferevent  
	evbuffer  
	evhttp  
	evdns  
	evrpc  
* The Libraries  
	libevent\_core  
	libevent\_extra  
	libevent  
	libevent\_pthreads  
	libevent\_openssl   
###Setting up the Libevent library  
* Log messages in Libevent  
	可以自定义日志记录的回调函数。需要注意的是，在自定义的回调函数中，不要调用 Libevent functions  
* Handling fatal errors  
	也可以自定义回调函数。需要注意的是，在该函数中，不要调用Libevent functions，此外，不要再将控制权交回Libevent  
* Memory management  
	可以自定义libevent所用的malloc, realloc, and free函数（其默认用C标准库中的）。文档中示例了一种分配方式，将所分配的内存大小放置于分配区域的最前面ALIGNMENT字节，而将ALIGNMENT之后的区域供libevent使用。  
	对于这三个函数的实现，有内存对齐和线程安全等要求  
* Locks and threading  
	Libevent structures在多线程中的三种情况：  
	Some structures are inherently single-threaded：不支持多线程。    
	Some structures are optionally locked：用户管理安全性。    
	Some structures are always locked：在多线程可安全的使用。  

	若使用 pthreads library, 或者the native Windows threading code，libevent已经定义了相关的locking functions.  
	若使用非上述的两种线程库，则需要定义相关的locking functions（三大类：Locks/Conditions/Threads）.    

	Debugging lock usage    
	在调试时，可以调用函数evthread\_enable\_lock\_debugging()，当非递归锁被锁定两次时，以及解锁一把非持有锁时，libevent会以assert方式退出。调用该函数时，需要有锁被创建或使用之后。  
	
	Debugging event usage  
	以如下方式使用事件：（1）使用未初始化事件（struct event）；（2）重新初始化一个pending struct event.  会发生错误。通过调用event\_enable\_debug\_mode(void)，来产生调试信息。该函数需要在任何事件创建之前调用。  
###Creating an event\_base  
* event\_base  
	若要使用Libevent的功能，则需要创建一个或者多个event\_base。每一个event\_base持有一个事件集，且能够区分出active的事件（借助select/poll/epoll/kqueue/devpoll/evport/win32，用户可以指定使用这些函数中的某几个或某一个，或者不使用某几个）。  
	event\_base的事件循环只能在一个线程中进行，因此，若要在多个线程中轮询IO（polling for IO），则需要在每个线程中创建一个event\_base对象。若event\_base使用了锁，则可以在多个线程中访问该对象。  
* 创建默认的event\_base  
	使用该函数struct event\_base *event\_base\_new(void)  
* Setting up a complicated event\_base  
	调用函数创建配置信息的结构体，struct event\_config *event\_config\_new(void);之后再调用相关函数对event\_config 进行设定，最后调用struct event\_base *event\_base\_new\_with\_config(const struct event\_config *cfg)创建event\_base  
	若要查看event\_base的配置信息，也有相关函数可供使用。  
* Deallocating an event\_base  
	void event\_base\_free(struct event\_base *base);但该函数不会deallocate any of the events that are currently associated with the event\_base, or close any of their sockets, or free any of their pointers.  
* Setting priorities on an event\_base  
	对于event\_base，默认只支持一个优先级，但可以通过设置，让其支持多个优先级，可用的优先级级别从0（最重要）到EVENT\_MAX\_PRIORITIES（最不重要）。比如，设置其可支持的优先级数量为3，那么event\_base中的事件可设定的优先级值可为0,1和2  
	所有新生成的event，其优先级值为event\_base支持的优先级数的一半。  
* Reinitializing an event\_base after fork()  
	
###Working with an event loop  
* Running the loop  
	int event\_base\_loop(struct event\_base *base, int flags);  
	在该函数被调用后，默认情况下，当event\_base对象中再也没有事件来注册，便退出。对于已注册的事件，若被触发（也被称作active），则调用该事件的回调函数。当然，通过设定flags可以改变该函数的运行行为。  **从文档中给的伪码可以看出，在该循环中，可能会处于wait状态（当然，不是while空转）**  
* Stopping the loop  
	函数1：event\_base\_loopexit(struct event\_base *base, const struct timeval *tv);  
	函数2：event\_base\_loopbreak(struct event\_base *base);  
	函数1是在定时到了之后退出，若此时事件循环正在执行active事件的回调函数，则依然执行完毕后退出（注意：是执行完所有active事件的回调函数）。  
	函数2在执行后会立即退出事件循环，若当前正在执行某active事件的回调函数，则执行完该事件的后退出，而其它active回调不会执行。  
###Working with events  
* event是什么？  
	event表示一个条件集（a set of conditions），是libevent的操作的基本单元。包括5个方面：  
	（1）I/O已经就绪的fd  
	（2）I/O即将就绪的fd（仅用于Edge-triggered IO）  
	（3）超时发生  
	（4）收到信号  
	（5）用户触发的事件。  
	通过调用libevent的函数来创建event，并关联到event\_base，这样便完成了event的初始化。通过add调用，event便进入了pending状态，此后，若event的触发条件满足，event进入active状态，且该event的回调函数会被执行。在回调函数被执行之前，该event会进入pending状态或者non-pending状态（依据该event是否被配置为persistent，若是，则pending，否则，non-pending）。对于non-pending，可以再调用add使其进入pending，也可以delete该event
	
* Creating an event as its own callback argument  
	event\_new()函数中，若要传递event自身（即event\_new()的返回值），这是难以实现的，libevent提供了一个函数event\_self\_cbarg()用于解决此难题。  
* 对于signal events的一些注意事项  
	不要为这种类型的事件设置超时。对于当前版本（2.0.x），除了kqueue()，其它的backends是不支持在同一个进程中有两个event\_base都接收signal events的。  
* Setting up events without heap-allocation  
	使用event\_assign  
	只有确定是因为 event\_new()影响了性能，才能使用非堆上的事件，此外，这种方式也肯能产生二进制兼容性问题。