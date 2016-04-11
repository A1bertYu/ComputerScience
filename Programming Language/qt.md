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

####内存管理问题  
<font color='red'>待续</font>   
这篇blog待读：http://blog.qt.io/blog/2009/08/25/count-with-me-how-many-smart-pointer-classes-does-qt-have/
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
####qmake Function Reference  
对qmake自带的函数进行介绍。对于具有返回值的函数，可以通过$$操作符获取其返回值。值得说明的是，在测试这些函数的过程中，若在pro文件中没有为CONFIG指定debug/release信息，这样qmake会生成三个文件，分别为Makefile<font color='red'>（代表什么模式？）</font>, Makefile.debug, and Makefile.release三个文件。在4.8.4中，若在CONFIG中指定了一些自定义的值，比如CONFIG+=xxx，则只会产生Makefile文件（qmake只会生成一个文件）。若pro中有message输出信息，对于每一个Makefile文件，则会输出一次，也就是说一条message语句可能会因为产生多个Makefile文件，而多次输出信息。若只想message只输出一条信息（即不论产生几个Makefile文件），可以使用语句

	#如此，可保证message只会输出一次信息，不论产生几个Makefile文件
	!build_pass:message( "This is a message" )  
此外，函数中处理的variablename，是指形如CONFIG，而CONFIG的值一般的形式为debug release debug\_and_release等，列表形式，用空格隔开，非空格（如debug）为其中一个值。     

* packagesExist(packages)  
	从帮助文档了解，是Symbian系统特有，故此处只列出而不做说明。  
* basename(variablename)  
	例如，变量值是/etc/passwd，那么结果将是passwd。也就是获取路径中的最深一层（最右侧）。  
* CONFIG(config)  
	该函数可以接收一个或者两个参数。当只接收一个参数时，用于判断CONFIG变量中是否已经包含了该参数值；接收两个参数的处理过程如下，首先依旧是判断是否包含了第一个参数值，若是，还需要判断该参数是否是第二个参数列出的所有候选项中最后出现的那一个（the active config），若还是，结果返回为真；其它情况结果返回都为假。所谓最后出现，是指与候选列表中的参数相比（当然，候选列表要求包含第一个参数才行，不然直接判断为假了），此参数是最后一个添加到CONFIG变量中的。  
* contains(variablename, value)  
	顾名思义，比如contains(CONFIG, debug)  
* count(variablename, number)  
	变量中包含number个数量的值  
* dirname(file)  
	与前面basename(variablename)互补，将最深一层去掉之后的值，为返回值  
* error(string)  
	输出错误信息后，qmake会直接退出。我们前面说到，qmake存在运行三次的情况，在输出error的情况下，第一次执行便退出后续不再执行。  
* message(string)  
	输出信息到console  
* warning(string)
	与message类似，输出信息到console，只是该信息表示警告。  
* eval(string)  
	返回true/false，如eval(TARGET=myapp)  
* exists(filename)  
	支持正则表达式，'/'可用于目录分割符（所有平台），exists( $(QTDIR)/lib/libqt-mt* )  
* find(variablename, substr)  
* isEmpty(variablename)  
	等价于count(variablename, 0) ，例如isEmpty( CONFIG )，若是空，则可以配置该变量  
* join(variablename, glue, before, after)  
	例如若CONFIG的值为debug release qt,且设定glue,before,after分别用G,B和A代替，那么产生的结果就是BdebugGreleaseGqtafter。另外，后面三个参数若不指定就默认为空值。  
* member(variablename, position)  
	返回position处的值，position默认为0，若指定的position不存在，则返回空字符串。  
* find(variablename, substr)   
	返回variablename中与substr能匹配的值，匹配支持正则表达式。  
* for(iterate, list)  
	遍历列表  
* include(filename)  
	将filename中的内容包含到当前工程中，内容导入位置即在include语句的地方。若包含成功，则返回true，否则返回false。若返回false，需要对文件进行检查，因为这会影响整个工程的构建。     
* infile(filename, var, val)  
	 文件filename中，是否包含了变量var，且var的值是否包含val   
* prompt(question)  
	输出提示信息，并返回输入内容。  
* quote(string)  
	顾名思义，将string放入双引号，如quote("yu xiao long")  
* replace(string, old_string, new_string)  
	第一个参数比如可以使CONFIG，那么该函数的作用是将CONFIG的内容中的old_string替换为new_string  
* sprintf(string, arguments...)  
	<font color='red'>按照C语言并结合QString.arg()形式的来使用，会报错，还不知道具体使用方法</font>
* system(command)  
	执行系统命令，返回0表示成功，其它表示失败。  
* unique(variablename)  
	若变量中包含了多个值，而多个值中有可能是相同的，那么该函数对相同值只输出一次。
	
		ARGS = 1 2 3 2 5 1
 		ARGS = $$unique(ARGS) #1 2 3 5
####qmake Variable Reference  
变量可以用来声明资源，配置工程信息，设置编译参数和设置链接参数。
  
* TEMPLATE  
	**app** 默认值，用于构建应用程序。  
	**lib** 构建library  
	**subdirs** 在子目录中分别进行构建，子目录由SUBDIRS指定。  
	**vcapp** Creates an application project for Visual Studio  
	**vclib** Creates a library project for Visual Studio  

* CONFIG  
	用于配置工程信息和编译器参数。  
	**release** 表示工程编译为release模式，若同时又指定了debug，则会忽略该参数。  
	**debug** 同理，debug模式  
	**debug\_and_release** 工程编译为debug和release模式，会有一些副作用。  
	**build\_all** 如果指定为debug\_and\_release，则会分别在debug和release模式下进行构建。  
	**ordered**  配合TEMPLATE的值为subdirs时使用，根据声明顺序对子目录进行构建。  
	**precompile\_header** 预编译头。<font color='red'>待详细说明</font>  
	**warn\_on** 编译器产生最多警告。  
	**warn\_off** 编译器产生最少警告。若warn\_on被指定，则warn\_off会被忽略。 

	当TEMPLATE是app或lib时：  
	**qt** 包含qt库，qt库的路径会被自定添加。  
	**thread** 多线程app或lib。qmake 

####qmake Advanced Usage  
* Operators  
	* = 
		对某变量赋值，该变量之前的值会被覆盖。  
	* +=  
		将值添加到变量。前面笔记也说过，变量是对形如CONFIG之类的称呼，而CONFIG的值其实类似于一个列表，因此+=就是将新的值添加到该列表。  
	* -=  
		将值从变量中移除  
	* *=  
		将值添加到变量。若该值已经存在，不做任何操作。  
	* ~=  
		使用正则表达式，替换字符串。<font color='red'>具体规则没看懂</font>  
	* $$  
		当qmake处理pro文件时，用于提取变量中的值。  
	* $  
		当Makefile文件被处理时，用于提取变量中的值。
	* :  
		用于嵌套作用域，当{}中只包含一个条件语句时，等效于“与逻辑”。如下所示，两种写法等价        
			
			macx {
     			debug {
         			HEADERS += debugging.h
     			}
 			}
			macx:debug {
     			HEADERS += debugging.h
 			}  
	* |  
		也是用于嵌套作用域，等效于“或逻辑”。  
			
* Scopes   
	条件语句和之后{}，注意，{要与条件语句在同一行，而}另起一行。  
	
		<condition> {
     		<command or definition>
     		...
 		}
* Configuration and Scopes  
	CONFIG变量比较特殊，其包含的值可以用于条件判断，比如若其包含了debug，那么debug可以被用作条件语句，其后面可以跟{}，{}里面的语句会被qmake执行。  
* Platform Scope Values  
	在QT安装目录下的mkspecs文件夹里面，有很多 built-in platform and compiler-specific values，这些值也可以用于条件判断.  
* Variables  
	除了qmake内置的变量外，也可以自定义变量。若qmake第一次遇到非内置变量被赋值时，将该变量当做自定义变量。  
	连接变量值：  
		
		#该语句会使TARGET的值为myproject_app（假设TEMPLATE=app，也就是连上TEMPLATE的值）  
		TARGET = myproject_$${TEMPLATE}    
* 自定义函数  
	格式如下：
	
		defineReplace(functionName){
     		#function code
 		}
		
		defineTest(){
			
		}
* Adding New Configuration Features  
	也可以自定义工程配置参数（ Features are collections of custom functions and definitions in .prf files ），将该参数名称加入到CONFIG变量中，qmake会根据该名称来搜索对应的prf文件。  
####Creating Shared Libraries  
在帮助文档中，QT介绍了一种结合pro和头文件的方式，来使得创建或者使用 Shared Libraries的通用形式。此外，使用 Shared Libraries的用户，需要包好Shared Libraries的公共头文件。但是，在创建Shared Libraries一些内部的头文件，若被公共头文件包含了，怎么办？这时可以考虑使用
 
	
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

###QtNetwork Module  

###Concurrent Programming  
####Threads and QObjects  
QThread继承自QObject. QObject对象（即继承自QObject），可以在多个线程中使用，发送signal和signal对应的slot可以在不同的线程，此外，postEvent的线程和处理该event的线程，也可以不同。因为每一个线程可以有其自己的事件循环。  

* QObject Reentrancy   
	（1）QObject中，父子必须要在相同的线程中进行创建。所以，在创建QThread时，不能指定this为其parent，因为QThread自身做为一个线程，被创建后肯定不会与当前的this在同一个线程。    
	（2）事件驱动的objects（Event driven objects）应该只能用在单线程中。  
	（3）创建于线程中的对象，必须要在删除线程本身之前进行删除。   
	（4）QObject是reentrant，且大多数non-GUI subclasses也是reentrant，但是GUI的类，则不是reentrant，且GUI的类只能用于主线程。    
* QWaitCondition  
	注意是不同线程之间的唤醒，一般是各线程公用同一个QWaitCondition对象，且QWaitCondition对象wait同一把锁，当进入wait状态，锁释放，在其它线程调用wake后，wait的线程返回，返回前会对锁进行lock()，也就是说，其返回代表其得到了锁。  
* Per-Thread Event Loop  
	An event loop in a thread makes it possible for the thread to use certain non-GUI Qt classes that require the presence of an event loop (such as QTimer, QTcpSocket, and QProcess). It also makes it possible to connect signals from any threads to slots of a specific thread.  
	每一个线程都有一个事件循环，而且具体事件的收发对象只能在同一个线程，也就是说不能跨线程收发事件。  
	在任何线程中，可以调用thread-safe函数QCoreApplication::postEvent()来发送事件，QT会自动将事件发送至事件对象所在的线程。  
	但是对于QCoreApplication::sendEvent()只能将事件发送至调用该函数的线程中。  
* Accessing QObject Subclasses from Other Threads  
	不在QObject对象所在的线程中访问QObject对象时，可能会出现这种情况，QObject可能正在接受事件循环中的事件（事件循环所在线程即为QObject对象所在线程），这时需要对QObject加锁，否则会出现UB

####例子  
* Mandelbrot Example  
	The Mandelbrot example shows how to use a worker thread to perform heavy computations without blocking the main thread's event loop.  
	在主线程中，创建了一个QThread（子类RenderThread），当需要background计算时，主线程调用了RenderThread对象的函数，注意，RenderThread所使用的数据都在自身对象中，其与主线程通过传值方式交互。RenderThread发送signal，而主线程实现了slot，两者位于不同线程。这种信号槽的连接，实际是Qt::QueuedConnection类型（虽然type传的参数值还是Qt::AutoConnection），而queue类型的话，QT需要将signal传递的参数值先保存起来，故要qRegisterMetaType()。    
	注意此例子中的QMutex和QWaitCondition的配合。  


###The Event System  
####Event loop  

* QCoreApplication::exec()  
	在调用该函数后，便进入了主事件循环（main event loop），直到QCoreApplication::exit()调用，便退出了主事件循环。主事件循环接收窗口事件，并分发至应用的各Widgets中。  
	对于清理代码（如释放资源），推荐是收到aboutToQuit()信号（该信号在即将退出主循环时发送，此时不会再有用户交互）后进行处理。而不是在QCoreApplication::exec()之后处理，因为有些系统执行该函数后不会返回，这样其后续的清理代码则无法被执行。  



####Timers   

* 直接调用QObject的函数来使用定时器       
	QObject基类中的函数startTimer ( int interval_ms )可以创建定时器，当定时事件发生，创建定时器的QObject对象（也可能是子类）的 timerEvent()会被调用（子类可以改写该虚函数）。注意，若时间间隔设置为0，则在没有窗口事件的情况下，会一直有QTimerEvent事件发生。  
	QApplication::exec()用于启动事件循环，当定时事件发生，QTimerEvent直到被处理才能离开事件循环，若一直不处理，则后续定时事件即使到了时间也不会发生。  
* 使用QTimer  
	在多线程环境下，可以使用QTimer，且只有开启事件循环时才能使用QTimer，在非GUI线程中，通过执行QThread::exec()来进行事件循环。QTimer通过 thread affinity来判断发送timeout()信号的线程，因此，开启和停止QTimer必须在QTimer对象所在的线程进行。  
* 使用QBasicTimer  
	This is a fast, lightweight, and low-level class used by Qt internally. 而QTimer则是high-level。接收QBasicTimer的QTimerEvent事件的QObject必须要实现 timerEvent()。  

###Others  
####Dialog  
* QProgressDialog  
	modal和modeless两种，后者一般用于显示后台任务进度。通过不停调用setValue() 来展示当前进度。对话框只有在minimumDuration()时间之后才会显示，也就是说，操作时间小于minimumDuration()的（默认4秒），不会显示进度对话框。  




###网络文章  
####Qt Internals & Reversing
[文章地址][Qt_internal_reversing_url]    
Daniel Pistelli  

* 几个重要的宏  
	
		#define SLOT(a)          "1"#a  /* 在4.8.4版本中稍有区别 */
		#define SIGNAL(a)        "2"#a

		#if defined(QT_NO_KEYWORDS)
		#define QT_NO_EMIT
		#else
		#define slots	/* 槽 */
		#define signals protected /* 信号 */
		#endif
		#define Q_SLOTS
		#define Q_SIGNALS protected
		#define Q_PRIVATE_SLOT(d, signature)
		#define Q_EMIT
		#ifndef QT_NO_EMIT
		#define emit /* 发射 */
		#endif

		#define Q_OBJECT_CHECK \
		    template <typename T> inline 
		    void qt_check_for_QOBJECT_macro(const T &_q_argument) const \
		    { int i = qYouForgotTheQ_OBJECT_Macro(this, &_q_argument); i = i; }

		/* 注意，下面两个模板函数不是宏 */
		template <typename T>
		inline int qYouForgotTheQ_OBJECT_Macro(T, T) { return 0; }
		
		template <typename T1, typename T2>
		inline void qYouForgotTheQ_OBJECT_Macro(T1, T2) {}
		#endif // QT_NO_MEMBER_TEMPLATES

		/* 用作国际化 */
		#define QT_TR_FUNCTIONS \
		    static inline QString tr(const char *s, const char *c = 0) \
		        { return staticMetaObject.tr(s, c); } \
		    static inline QString trUtf8(const char *s, const char *c = 0) \
		        { return staticMetaObject.trUtf8(s, c); } \
		    static inline QString tr(const char *s, const char *c, int n) \
		        { return staticMetaObject.tr(s, c, n); } \
		    static inline QString trUtf8(const char *s, const char *c, int n) \
		        { return staticMetaObject.trUtf8(s, c, n); }
		
		#define Q_OBJECT \
		public: \
		    Q_OBJECT_CHECK \
		    static const QMetaObject staticMetaObject; \
		    virtual const QMetaObject *metaObject() const; \
		    virtual void *qt_metacast(const char *); \
		    QT_TR_FUNCTIONS \
		    virtual int qt_metacall(QMetaObject::Call, int, void **); \
		private:
从上面的宏Q_OBJECT的定义中，我们可以看到其有一个重要的public static数据成员staticMetaObject，其类型为QMetaObject，其定义重要部分摘录如下，具体可参考QT源码（QT4.8.4在文件qobjectdefs.h中）  
 
		struct Q_CORE_EXPORT QMetaObject
		{
			struct { // private data
		        const QMetaObject *superdata;
		        const char *stringdata;
		        const uint *data;
		        const void *extradata;
	    	} d;
		};
d的成员superdata是指使用了宏Q_OBJECT的类的直接基类的staticMetaObject（若是多重继承，则是第一个基类的staticMetaObject），因此，其直接基类必须是QObject或者QObject的派生类（因为要有staticMetaObject成员）。  
d的成员stringdata包含了类的the literal metadata（文字信息）。  
d的成员data包含了类的the metadata offsets, flags etc.通过data，得到offset，再通过offset以及stringdata来获得类成员函数名称。  
The fourth member is a null terminated array of QMetaObject classes. 很少使用。  
Qt dynamism relies on indexes, avoiding pointers.   
参数的传递是通过指向指针数组的指针进行，当调用具体方法时，再进行适当的cast。Using pointers, of course, is the only way to put all kinds of types in an array. Arguments start from position 1, because position 0 is reserved for the data to return.   
对于定义的信号，moc会自动生成函数的定义，函数体内主要是调用QMetaObject的 Activate method.  
QMetaObject::activate()中，当前的connection（注意，信号和槽是connect的，故做此名称）是否需要立即被处理，或者是否放入队列稍后处理。若要立即处理，首先从当前的connection中获得需要调用的方法的ID，之后调用接收者的qt\_metacall方法。<font color='red'>关于Connection class，还需要研究。</font>  
关于动态调用，使用QMetaObject提供的invokeMethod。

###Dynamic C++ Proposal  
[文章地址][Dynamic C++ Proposal url]  
##参考
[Qt_internal_reversing_url]:http://www.codeproject.com/Articles/31330/Qt-Internals-Reversing?msg=2827698#xx2827698xx   
[Dynamic C++ Proposal url]:http://www.codeproject.com/Articles/31988/Dynamic-C-Proposal  
