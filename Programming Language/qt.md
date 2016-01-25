#QT学习
本文档主要记录QT使用的技术，以及实践中遇到的工程问题。因此分为两个部分，原理和工程，并且讨论版本为QT4.8.4.

##QT原理
在Assistant的章节"Develop with Qt"，介绍了QT中使用到的技术，下面对此进行分别说明。

###Qt Technologies
QT使用信号-槽机制来进行对象间通信（signals and slots），以此来替代传统框架中的不安全的回调函数。同时，对于用户的输入（包括鼠标键盘事件等）的处理，QT也提供了合适的事件模型（ a conventional event model）。

#### **Qt's Meta-Object System**  

Qt's meta-object system provides the signals and slots mechanism for inter-object communication, run-time type information, and the dynamic property system.  （摘自手册，说明了The Meta-Object System的重要性。）  
MOS要发挥作用，以如下三个队友为支撑：（1）QObject；（2）Q_OBJECT；（3）Meta-Object Compiler (moc) 。下面对此进行逐一介绍。
	
#####QObject  
QObject是所有QT对象的基类。若要使用MOS，必须要继承QObject。

1. Object Model  
	在C++现有模型基础上，QT增加了以下特性：
	* 基于信号-槽（signals and slots）的对象间的无缝通信机制  
	* 可查询和可设计的对象属性（object properties）
	* 强大的事件和事件过滤器（powerful events and event filters）
	* 基于间隔驱动的定时器（sophisticated interval driven timers），可以优雅的将多个任务整合到事件驱动的GUI
	* 层级鲜明以及可查询的对象树（object trees）
	* 智能指针（guarded pointers）QPointer
	* a dynamic cast that works across library boundaries<font color='red'> 待研究</font>
	
	Object Model的基础类  
	* QMetaClassInfo  
		只是一些辅助信息，其中在assistant中介绍该类时提到宏Q_CLASSINFO，该宏只是其注释性作用，无它。另外，看了QMetaClassInfo的源码，其有一个QMetaObject指针类型的数据成员，在name()和value()是返回的该成员的相关信息。
	* QMetaObject  
		QMetaObject实例存储了QObject子类的所有meta-information。QObject的子类可以调用metaObject()来获得指向该子类的meta-object的指针，QObject的所有子类都拥有属于自己的QMetaObject实例。  
		该类通常的程序中不会使用，除非要写meta-applications（例如脚本引擎、GUI生成器）
	* QVariant  
		The QVariant class acts like a union for the most common Qt data types.C++中，union  	

		
	
#####Q_OBJECT


#####Using the Meta-Object Compiler (moc)
The Meta-Object Compiler, moc, is the program that handles Qt's C++ extensions(摘自Manual，此处所述扩展也就是MOS)。moc会对头文件进行处理，当头文件中有QObject子类的Q_OBJECT macro	
####The Event System
在QT中，事件也是对象，都是继承于QEvent。事件可以在当前程序内部产生或者外部产生（外部一般是指当前程序需要知道的信息）。事件可以被任何继承自QObject的实例处理（widgets一般与事件无关）。 
 
1. 事件的分发（How Events are Delivered）  
	当有事件产生时，QT产生一个QEvent子类的实例，然后通过调用QObject子类实例的event()来传递给它。在event()中，根据事件的类型，来调用相应的handle来进行处理，之后该函数返回事件被处理的状态（accepted or ignored）。
2. Event Types  
	大多数事件都有特定的类，如 QResizeEvent, QPaintEvent和QMouseEvent，并在QEvent的基础上增加了相应的方法。某些class可能支持多种的事件类型，如QMouseEvent 支持单击、双击和移动等。在运行时，通过type()函数（返回类型QEvent::Type）可以快速的确定当前的事件对象类型。
3. Event Handlers  
	事件的传递一般是调用虚函数（例如event()或者QWidget::paintEvent()）。可以通过覆写基类的函数，来对某些关注的事件类型的处理方式进行改写，而非关注的类型依旧调用基类函数来进行处理。返回true则表示事件处理完毕。
4. Event Filters  
	有些时候某些object需要对传递到别的object的事件进行拦截过滤等处理。函数QObject::installEventFilter(QObject *object, QEvent *event)可以实现此功能。若 对QApplication或者QCoreApplication object安装fileter，则会形成全局过滤（针对整个程序），此方法会降低事件分发速度。
5. Sending Events  
	使用 QCoreApplication::sendEvent()和QCoreApplication::postEvent()，程序也可以发送自定义事件。其中，sendEvent()会立即发送事件，其返回时，事件已经被处理。postEvent()则不同，其会将事件加入队列而延时发送。当下一次QT事件主循环时，所有队列的事件被发送，在这过程中，会有一些优化，比如将多次缩放窗口操作合并为一次。postEvent（）也用于对象的初始化期间，自定义的事件类型必须要大于QEvent::User。
####The Property System
属性与数据成员类似，然而，因为Property基于MOS，因此其比数据成员拥有更多的特性。若要使用属性，需要在QObject的子类对象中，使用Q_PROPERTY()来进行声明。  
1. 语法  

			Q_PROPERTY(type name  
            READ getFunction  
            [WRITE setFunction]  
            [RESET resetFunction]  
            [NOTIFY notifySignal]  
            [REVISION int]  
            [DESIGNABLE bool]  
            [SCRIPTABLE bool]  
            [STORED bool]  
            [USER bool]  
            [CONSTANT]  
            [FINAL])  
上述中的[]表示可选参数，其它则是必要参数。  
READ函数必须返回属性所对应的类型或者该类型的引用或指针，一般来说，最好是const函数。  
WRITE函数必须返回void，且必须要求一个形参，该形参类型是属性所对应的类型或者该类型的引用或指针。  
RESET必须返回void并且不能有形参，用于将值恢复到默认值。  
NOTIFY必须关联对象中已存在的信号，当该属性值变化时，会发射该信号。  
REVISION用于QML，暂且不表。  
DESIGNABLE用于QtDesigner，暂且不表。  
SCRIPTABLE用于脚本，暂且不表。  
SCRIPTABLE

 

####Why Doesn't Qt Use Templates for Signals and Slots?
模板和C语言的预编译器可以用来做一些奇技淫巧的事情（ incredibility smart and mind boggling things）。有一个很现实的原因，要做到跨平台，需要考虑多种编译器，有些对template的支持并不全面。不过，编译器的限制并不是主要原因，之所以MOC（meta object compiler）采用基于字符串的方式（string-based approach）而不是基于template，有以下五大原因。  

1. Syntax matters  
	语法要直观、简单、易用、易读和易维护，目前的信号-槽语法就相当符合要求，也很实用。信号的声明与普通成员变量类似，可以收到类定义的保护（访问等级限制）。 
2. Code Generators are Good  
	MOC对于包含Q_OBJECT宏的类，产生额外的C++代码（the meta object code ）。除了MOC，UIC（User Interface Compiler）也会产生代码，其可将XML中的界面信息转换为C++代码。
3. GUIs are Dynamic  
    C++有很多优点，其the static object model对于数据库或者OS开发很有优势，但对GUI开发却是劣势（ component-based graphical user interface programming）。原因是什么？但MOC将该劣势化为了优势，增加了GUI开发的灵活性，并且基于字符串的方式比基于模板的方式能实现更多的特性。例如，可以实现object properties，信号不占用对象空间（即不会破坏二进制兼容性）。此外，不需要像template那样inline实现，因而，可以使最终代码更小。新建一个连接只是简单的增加了一个函数调用，而不是增加一个复杂的模板函数。运行时若不知道被连接的对象类型，也可以建立连接（establish connections using type-safe call-by-name，该方式算是一种runtime introspection ）。
4. Calling Performance is Not Everything
	QT采用的基于字符串的方式没有基于模板的快。当发射一个信号时，基于模板的机制需要大约4次的函数调用，而QT当前采用的机制需要大约10次的函数调用。因为QT机制包括a generic marshaller, introspection, queued calls between different threads, and ultimately scriptability，而不是inline和代码扩展。这段说template不能type safe，表示不理解？QT的信号-槽机制的开销一般可忽略不计，若该开销不能忽略， 可考虑使用listener-interface pattern。
5. No Limits
	超越基于模板机制的局限，可以实现其不能实现的功能，
	* tr() function
	* property system with introspection and extended runtime type information
	* a dynamic qobject_cast<T>() mechanism
	* dynamic meta objects

最后，点睛之笔。C++ with the moc essentially gives us the flexibility of Objective-C or of a Java Runtime Environment, while maintaining C++'s unique performance and scalability advantages. It is what makes Qt the flexible and comfortable tool we have today.

##QT工程实践
###Qt Linguist Manual
主要涉及到两个命令行工具，lupdate和lrelease，前者用于搜寻源码文件（.cpp, .h和Qt Designer产生的.h文件）中的tr字符串，以便产生或者更新.ts文件；后者则将.ts文件转换为二进制的.qm文件，用于程序运行时的本地化，通过该.qm文件可以非常快速的查询翻译。
###qmake

###Qt Designer
使用Qt设计师时，可以有两种模式，在“设置->属性”菜单中可以调整，分别为multi-window和docked window，这两种模式只是在编辑UI时呈现的编辑界面有不同（对UI本身没有影响），个人感觉后者在编辑UI时好用一点。
  
* Layout  
Qt中的布局（Layouts）概念，布局用于管理和放置构成UI的元素（也就是QWidget）。Qt中的类QHBoxLayout, QVBoxLayout, QGridLayout, and QFormLayout用于自动处理布局。每一个QWidget有一个推荐的大小（即函数sizeHint()）。  
Qt布局的作用：  
（1）为适应不同的窗口大小，重新调整UI尺寸；  
（2）能够调整UI中的各element大小，以适应本地化；  
（3）Arrange elements to adhere to layout guidelines for different platforms.  

* Containers in Qt Designer   
	Container widgets可以以表格形式来管理一组object。
