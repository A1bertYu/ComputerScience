# Compilers
Alfred V.Aho, Monica S.Lam, Ravi Sethi, Jeffrey D.Ullman   
Second Edition, English Version, 2006 机械工业出版社

##Chapter 2  A Simple Syntax-Directed Translator  
###2.1 Introduction  

* 基本概念   
	Syntax : the proper form of its program  
	Sematics : define what its programs mean.    
	Analysis Phase: source-> intermediate code，主要做syntax分析。  
	Synthesis Phase: intermediate code -> target program  
	* [Production][production_url]   
		也称为production rule，是一种[rewrite rule][rewirting_url]，用于指定符号的替换方法，该方法可以递归的执行。  
	* [Context-free grammar][context-free-grammar-url]  
		写CFG有两种技术，[BNF][BNF_url]和[Van Wijngaarden grammar][vW-grammar-rul].  
		称其为context free是因为在应用其production rules时，不需要考虑nonterminal所处的context，也就是左边的nonterminal总是可以被右边的body替换。当然，不是context free的称为[Context-sensitive grammar][CSG-url].

##Principle
* [Execution model][Execution_model_url]  
	每一种语言都有其Execution model，which determines the manner in which the units of work (that are indicated by program syntax) are scheduled for execution. An implementation of an execution model controls the order in which work takes place during execution. This order may be chosen ahead of time, in some situations, or it can be dynamically determined as the execution proceeds.   
	The static choices are most often implemented inside a compiler, in which case the order of work is represented by the order in which instructions are placed into the executable binary. The dynamic choices would then be implemented inside the language's runtime system.   

* [Runtime system][Runtime_system_url]  
	其中一个较为争议的定义： any behavior that is not directly the work of a program is runtime system behavior. 比如，C++中，对于一个函数的调用，将参数放入栈中，便可以称为runtime system behavior.  
	In addition to the execution model behavior, a runtime system may also perform support services such as type checking, debugging, or code generation and optimization.  
	There are often no clear criteria for deciding which language behavior is considered inside the runtime system versus which behavior is "compiled".  
* Automatic variable  
	an automatic variable is a local variable which is allocated and deallocated automatically when program flow enters and leaves the variable's scope. 在C++中，static local则不算antomatic
  
	
* Closure（也称lexical closures or function closures）   

* Blocks (C language extension)  
	Apple公司对C/C++/Obj-C的扩展，使其支持closure。Apple的初衷是为了方便写the Grand Central Dispatch threading architecture程序。  
	GCC也对C进行了扩展，以便支持内嵌函数，但是内嵌函数只能在the containing scope里面调用。  

* Thunk  
	a thunk is a subroutine that is created, often automatically, to assist a call to another subroutine.   
* Nested function  
	字面意思就是在函数内部定义的函数，称为内嵌函数。Due to simple recursive scope rules, a nested function is itself invisible outside of its immediately enclosing function, but can see (access) all local objects (data, functions, types, etc.) of its immediately enclosing function as well as of any function(s) which, in turn, encloses that function. 【只对包围它且最内层的函数可见，不过，对于嵌套函数，其可以获取包围它的所有外部函数的objects，这样使得数据的传递极为方便】  
	
	若函数为first class object，而内嵌函数能够作为变量返回，或者作为参数传递到其它函数，那么其就是closure。在调用closure时，can access the environment of the original function（闭包）.对于包围闭包的函数（最内层的包围函数），其调用帧（frame）必须要保证最后一次调用closure之前还有效，因此，closure中的non-local变量不能放在stack中，这就是著名的funarg problem.  

	引入nested function的目的之一为information hiding，将大的过程拆分为小的，且定义的函数只是meaningful locally，这就避免了将这些子函数信息暴露出去。They are typically used as helper functions or as recursive functions inside another function  

	关于nested function的其它实现， 例如C语言中，可参考Nginx源码中，关于过滤模块组成的单链表。此外，对于Nginx中的module结构体，其中包含了函数指针，也可以视为nested function的一种实现。

	<font color='red'>关于具体实现方法，还需要参考wikipedia</font>

* Funarg problem  
	
* Type introspection  
    
##Visual Studio

###文章摘录
本小节记载一些在网上看到的博文。
####Character encodings and the beauty of UTF-8
[本文地址][utf-8-history]，发表时间April, 2011，笔记记载时间Jan, 2016  

#### Using UTF-8 as the internal representation for strings in C and C++ with Visual Studio
[本文地址][utf-8-vs]，发表时间May, 2011，笔记记载时间Jan, 2016  
本文主要分享了使用Visual Studio进行C/C++开发时，通过char和std::string来处理Unicode文本（必须编码为UTF-8）的经验。

###工程实践  
* 关于多重继承  

		//该代码在没有TestMultiA实例调用test()方法时，会编译通过；但在调用test()方法时（因为两个基类都有该函数的实现），
		//编译不能通过。即使将两个基类中的test形参设置为不同（一个为空，一个有一个int形参），还是不能通过编译
		class TestMultiB
		{
		public:
		    void test()
		    {
		        std::cout<<"TestMultiB::test()"<<std::endl;
		    }
		    virtual void testv()
		    {
		        std::cout<<"TestMultiB::testv()"<<std::endl;
		    }
		};

		class TestMultiC
		{
		public:
		    void test(int a)
		    {
		        a = 0;
		        std::cout<<"TestMultiC::test()"<<std::endl;
		    }
		    virtual void testv1()
		    {
		        std::cout<<"TestMultiC::testv()"<<std::endl;
		    }
		};

		class TestMultiA : public TestMultiB, public TestMultiC
		{
		
		};
* 关于基类中的静态变量在子类被操作后  
	若基类A有一个static int a; 子类B和子类C都是继承自类A，那么子类B的实例对a进行了操作，子类C的实例中的a也会改变，也就是说，基类中的静态变量被所有子类（当然还有基类本身）的实例共享。  
* 关于inline  
	VS2010中，若某些函数被声明为inline，在其它class中引用会出现链接不到的错误。  

##GCC Compiler

##参考链接
[utf-8-vs]:http://www.nubaria.com/en/blog/?p=289  
[utf-8-history]:http://www.nubaria.com/en/blog/?p=132
[production_url]:https://en.wikipedia.org/wiki/Production_(computer_science)
[rewirting_url]:https://en.wikipedia.org/wiki/Rewriting  
[context-free-grammar-url]:https://en.wikipedia.org/wiki/Context-free_grammar  
[BNF_url]:https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_Form  
[vW-grammar-rul]:https://en.wikipedia.org/wiki/Van_Wijngaarden_grammar  
[CSG-url]:https://en.wikipedia.org/wiki/Context-sensitive_grammar  
[Execution_model_url]:"https://en.wikipedia.org/wiki/Execution_model"
[Runtime_system_url]:"https://en.wikipedia.org/wiki/Runtime_system"
