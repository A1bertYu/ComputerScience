##Kqueue: A generic and scalable event notification facility  
Jonathan Lemon jlemon@FreeBSD.org  

* Introduction  
	FreeBSD提供了两个系统调用，poll() and select(). 但这两个调用在fd数量很多（several thousand）时，性能不好（not scale very well）。  
* Problem  
	当初设计select/poll时，考虑了stateless（通过传参的形式实现），这样，kernel无需保存app的任何信息。这样，就导致了在多个系统调用之间，无法共享app的一些信息，从而会重复计算某些信息。  
	select/poll的过程如下：app的每一次select/poll调用，都需要将整个fd列表的数据从用户态传递到内核态，而在实际应用中，整个列表中的大约只有5%是active状态，也就是说95%左右的拷贝都是无用的。  
	当系统态返回后，app在用户态依旧需要遍历整个fd列表，来寻找active状态的fd（而这部分工作kernel已在系统态做过，算是重复工作）。若kernel返回给app的就是已active的list，则会改善性能  
	在系统态，kernel需要为list malloc内存，在返回时还需要free。kernel需要遍历两次list，首先查找具有pending activity的fd，若没有找到，则更新fd的select information  
##Reactor An Object Behavioral Pattern for Demultiplexing and Dispatching Handles for Synchronous Events  
Douglas C. Schmidt  


##Proactor : An Object Behavioral Pattern for Demultiplexing and Dispatching Handlers for Asynchronous Events	
Irfan Pyarali, Tim Harrison, and Douglas C. Schmidt Thomas D. Jordan  
通过异步机制的并发，可避免多线程机制并发所产生的同步问题。本文主要讲述了如何通过OS提供的异步机制，来进行并发应用程序的设计。所谓异步，是指当应用程序进行异步操作时，操作系统代为完成，因此，在委托给OS后，应用程序可以去进行别的的任务。  

###几种并发Models的优缺点  
* 多线程并发（synchronous multi-threading）  
	优点是，模型简单，程序设计容易；每个线程处理一个连接，而一般各连接基本是独立的，也就是说基本无需同步。  
	**缺点**：  
	（1）多线程策略（例如，怎样设计线程池？）可以利用CPU个数等资源进行优化，但因连接数无法掌控，而难以利用这些信息。  
	（2）共享资源的同步比较复杂  
	（3）线程切换、同步和CPU间的数据交换会带来很大开销  
	（4）可移植性差，各OS对多线程的支持情况不同。  
* Reactive Synchronous Event Dispatching   

##Architectural Styles and the Design of Network-based Software Architectures  
Roy Thomas Fielding  

an Internet-scale distributed hypermedia system  

**Software architecture** research investigates methods for determining how best to
partition a system, how components identify and communicate with each other, how
information is communicated, how elements of a system can evolve independently, and
how all of the above can be described using formal and informal notations.  

An architectural style is a named, coordinated set of architectural constraints.  

Representational State Transfer (REST) architectural style guides the design and development of the architecture for the modern Web  

the two specifications (HTTP and URI) that define the generic interface used by all component interactions on the
Web.  
###Chapter01 Software Architecture  

* 1.1 Run-time Abstraction  
	At the heart of software architecture is the principle of abstraction: hiding some of the details of a system through encapsulation in order to better identify and sustain its properties.【抽象的重要性】  
	一个复杂的系统包含多层次的抽象，每一层抽象都有其自身架构。架构用于表现抽象，架构的元素通过抽象接口进行描绘，这些接口被提供给架构中同层次的其它元素使用。当然，架构中的元素可以包含子架构，如此递归，直至由最基本的元素组成（无法抽象的元素）。  
	要考虑运行时的行为，architectural design（包含动态的处理过程） and source code structural design（静态的源码）, though closely related, are separate design activities.  

* 1.2 Elements  
	**data elements**：contain the information that is used and transformed.  
	（1）定义：A datum is an element of information that is transferred from a component, or received by a component, via a connector.注意，不包括只在单个component中的数据（类似于私有概念，即不能只是一个component私有，而不给别的component使用。）   
	（2）举例：byte-sequences, messages, marshalled parameters, and serialized objects。
   
	**Processing elements（components）**：perform transformations on data.   
	（1）定义：A component is an abstract unit of software instructions and internal state that provides a transformation of data via its interface.  
	（2）补充：a component is defined by its interface and the services it provides to other components, rather than by its implementation behind the interface.     

	**connecting elements（connectors）**：the glue that holds the different pieces of the architecture together.    
	（1）定义：A connector is an abstract mechanism that mediates communication, coordination, or cooperation among components.   
	（2）实例：shared representations, remote procedure calls, message-passing protocols, and data streams.  
	（3）从外在表现来看，Connectors enable communication between components by transferring data elements from one interface to another without changing the data.  
* 1.3 Configurations

	A configuration is the structure of architectural relationships among components, connectors, and data during a period of system run-time
	
* 1.4 Properties   
	从软件系统所需要的功能，可以推导出功能性的properties；而非功能方面，包括易于演进和扩展、组件重用性好等可以推导出非功能性的properties.  
	
* 1.5 Styles  
	An architectural style is a coordinated set of architectural constraints that restricts the roles/features of architectural elements and the allowed relationships among those elements within any architecture that conforms to that style.	 
	styles基于architecture的普遍特征来对architectures进行分类，每一种style是对组件交互的一种抽象。  
	关于style为什么会是architectural constraints，文章对此也进行了历史论述，这是因为借用了建筑学的术语，过去的建筑很多时候要考虑到选材、文化等很多外界因素，并受限于这些，故style反倒不是建筑师个人的风格，而是这些外界限制。  
* 1.6 Patterns and Pattern Languages  
	a primary benefit of patterns is that they can describe relatively complex protocols of interactions between objects as a single abstraction.  
	a pattern defines a process for solving a problem by following a path of design and implementation choices.  
	
* 1.7 Views  
	it is also possible to view an architecture from many different perspectives.  
	有人提出三种重要的view：processing, data, and connection views.  
###Chapter02 Network-based Application Architectures  

* 2.1.1 Network-based vs. Distributed  
	基于网络的架构与常规的非网络架构，最大的区别就是，components is restricted to message passing。  
	文中还对distributed systems and network-based systems进行了对比说明。  

* 2.1.2 Application Software vs. Networking Software  
	本文仅限于讨论Application Software  

* 2.2 Evaluating the Design of Application Architectures   
	The first level of evaluation is set by the application’s functional requirements.  
	由前文可知，an architectural style is a coordinated set of architectural constraints，所以通过增减以及合并constraints，可以形成新的style（当然，合并的话要求constraints无冲突）  
	style树：the space of all possible architectural styles as a derivation tree, with its root being the null style (empty set of constraints). 因而，对于某一种style树，其architectural properties的演进是针对特定的application domain的，故不同的application domain，不能比较architectural designs。
* 2.3 Architectural Properties of Key Interest   

 * 2.3.1 Performance  
	 Throughput, Overhead, Bandwidth and Usable bandwidth.  
* 2.3.1.2 User-perceived Performance  
	latency and completion time  
* 2.3.1.3 Network Efficiency  
	An interesting observation about network-based applications is that the best application performance is obtained by not using the network.  
* 2.3.2 Scalability   
	Scalability refers to the ability of the architecture to support large numbers of components, or interactions among components, within an active configuration.  
* 2.3.3 Simplicity  
	首要方式：the principle of separation of concerns to the allocation of functionality within components.  
* 2.3.4 Modifiability  
	Modifiability can be further broken down into evolvability, extensibility, customizability, configurability, and reusability.    
	(1)Evolvability  
	在不影响其它组件的情况下，可以演进的程度，  
	（2）Extensibility  
	Extensibility is defined as the ability to add functionality to a system.  
	(3)Customizability  
	(4)Configurability  
	(5)Reusability  

* 2.3.5 Visibility  
	Visibility in this case refers to the ability of a component to monitor or mediate the interaction between two other components.
* 2.3.7 Reliability  	
	Styles can improve reliability by avoiding single points of failure, enabling redundancy, allowing monitoring, or reducing the scope of failure to a recoverable action.  

###Chapter03 Network-based Architectural Styles  

* 3.1.1 Selection of Architectural Styles for Classification   
	本文只关注那些强调the communication or interaction properties的styles  
* 3.1.2 Style-induced Architectural Properties  
	本文仅研究network-based hypermedia systems，主要是研究每一种style对该种类的系统架构的影响。  
* 3.1.3 Visualization  
	本文中，用表来表征style对architectural properties的影响。  

* 3.2 Data-flow Styles  
	
	* 3.2.1 Pipe and Filter (PF)  
		各filter之间独立。  
	* 3.2.2 Uniform Pipe and Filter (UPF)  
		所有的filter之间有相同的接口。  
	
* 3.3 Replication Styles  
	
	* 3.3.1 Replicated Repository (RR)  
		类似于git的思想，中央仓库本地化，这需要考虑到一致性。  
	* 3.3.2 Cache ($)  
		A variant of replicated repository is found in the cache style.  
* 3.4 Hierarchical Styles  
	* 3.4.1 Client-Server (CS)  
		A client is a triggering process; a server is a reactive process. It is often referred to by the mechanisms used for the connector implementation  
	* 3.4.2 Layered System (LS) and Layered-Client-Server (LCS)  
		Layered-client-server adds proxy and gateway components to the client-server style.  
	* 3.4.3 Client-Stateless-Server (CSS)  
		在CS Style基础上，加上了限制，即no session state is allowed on the server component. session state完全保存在客户端。  
	* 3.4.4 Client-Cache-Stateless-Server (C$SS)  
		client和server之间增加一个中间件，用作cache  
	* 3.4.5 Layered-Client-Cache-Stateless-Server (LC$SS)  
		An example system that uses an LC$SS style is the Internet domain name system (DNS).  
	* 3.4.6 Remote Session (RS)  
		client-server的一个变种，主要用于轻客户端。Each client initiates a session on the server and then invokes a series of services on the server, finally exiting the session. Application state is kept entirely on the server.例子TELNET  
	* 3.4.7 Remote Data Access (RDA)  
		client-server的一个变种，spreads the application state across both client and server. 例子，数据库的客户端和服务器端。查询产生了a very large data set（服务器端分配空间），客户端可以继续对服务器端产生的数据进行操作（不需要拷贝到客户端来进行），比如table joins等，这样可以依次将结果范围缩小，最后得到的结果（需要传送到客户端的数据）数据量就比较小了。 

* 3.5 Mobile Code Styles  
	Mobile code styles use mobility in order to dynamically change the distance between the processing and source of data or destination of results.   
	* 3.5.1 Virtual Machine (VM)  
		Underlying all of the mobile code styles is the notion of a virtual machine, or interpreter, style.  
	* 3.5.2 Remote Evaluation (REV)  
		derived from the client-server and virtual machine styles  
	* 3.5.3 Code on Demand (COD)  
		client获取了资源，但是不知道如何处理，其发送一个请求至服务器端，以便获知如何在client本地进行处理（返回结果类似于可执行的代码）  
	* 3.5.4 Layered-Code-on-Demand-Client-Cache-Stateless-Server (LCODC$SS)  
		the code can be treated as just another data element，因此，可将COD和LC$SS结合（两者不冲突）。  
	* 3.5.5 Mobile Agent (MA)  
		an entire computational component is moved to a remote site, along with its state, the code it needs, and possibly some data required to perform the task.  

* 3.6 Peer-to-Peer Styles  
	
	* 3.6.1 Event-based Integration (EBI)  
		
	* 3.6.2 C2  
		combining event-based integration with layered-client-server.   
	* 3.6.3 Distributed Objects  
	* 3.6.4 Brokered Distributed Objects  
		
* 3.7 Limitations  
	
###Chapter04 Designing the Web Architecture: Problems and Insights  

* 4.1 WWW Application Domain Requirements  
	
	

###Chapter05 Representational State Transfer (REST)  

* 5.1 Deriving REST  
	通过增加constraints，以导出REST style  
* 5.1.1 Starting with the Null Style  
	the null style describes a system in which there are no distinguished boundaries between components. It is the starting point for our description of REST.	

两种常见的关于网站架构（Web architecture）设计的方法：  
（1）从下到上，先组件（component），再到整体；  
（2）从上到下，从整体到局部，REST使用的这种方式。  

* 5.1.2 Client-Server  
	首先增加的是Client-Server style。组件模块化（the separation allows the components to evolve independently）。Perhaps most significant to the Web, however, is that the separation allows the components to evolve independently, thus supporting the Internet-scale requirement of multiple organizational domains.

* 5.1.3 Stateless
	其次增加的是communication must be stateless in nature。  
	客户端与服务器交互的无状态约束：  
	（1）客户端发送的每一个请求必须包含所有信息，以便服务器对其进行相应，而不能依赖任何服务器端的信息；  
	（2）由（1）可知，Session state需要完整的保存在客户端。  
* 5.1.4 Cache  
	对于client-cache-stateless-server style，响应（response）中的数据，可以显式或者隐式的进行cacheable/non-cacheable标记。若response是cacheable，则客户端就可以利用之前的数据，但这种方式也有可能会遇到数据不一致的问题。  
* 5.1.5 Uniform Interface   
	The central feature that distinguishes the REST architectural style from other networkbased styles is its emphasis on a uniform interface between components  
	REST is defined by four interface constraints: identification of resources; manipulation of resources through representations; self-descriptive messages; and, hypermedia as the engine of application state.   
	
* 5.2 REST Architectural Elements  
	The Representational State Transfer (REST) style is an abstraction of the architectural elements within a distributed hypermedia system.  
	REST不会去考虑组件的具体实现，而是关注组件之间的关系，以及组件对于基础数据的处理。  

	* 5.2.1 Data Elements  
		（1）5.2.1.1 Resources and Resource Identifiers  
		在REST中，任何信息都可以称为资源。资源R是一个映射关系，可以表示为关于时间的函数，在时间t的函数值是实体集合（a set of entities），集合中的元素是resource representations和/或resource identifiers。  
		资源R也可以映射到空集合，相当于在实现之前先占位之意。  
		对于静态资源，其映射集合一直不会变化。  
		REST uses a resource identifier to identify the particular resource involved in an interaction between components.  
		REST connectors provide a generic interface for accessing and manipulating the value set of a resource, regardless of how the membership function is defined or the type of software that is handling the request.  

		（2）5.2.1.2 Representations  
		定义：A representation is a sequence of bytes, plus representation metadata to describe those bytes.  
		REST components perform actions on a resource by using a representation to capture the current or intended state of that resource and transferring that representation between components.  
		**Control data** defines the purpose of a message between components，请求的action，响应返回的意思。  
		The data format of a representation is known as a **media type**.   

	* 5.2.2 Connectors  
		All REST interactions are stateless.  
		Shared caching can be effective at reducing the impact of “flash crowds” on the load of a popular server.<font color='red'>没理解，需要举例，待搜索</font>  
		
	* 5.2.3 Components  
		

##Chapter06  

REST is not intended to capture all possible uses of the Web protocol standards.

REST was originally referred to as the “HTTP object model,” 一个良好的网络应用，应该满足：a network of web pages (a virtual state-machine), where the user progresses through the application by selecting links (state transitions), resulting in the next page (representing the next state of the application) being transferred to the user and rendered for their use.  

The definition of resource in REST is based on a simple premise: identifiers should change as infrequently as possible
    