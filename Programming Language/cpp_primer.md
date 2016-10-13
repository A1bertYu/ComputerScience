#C语言  
1、关于C语言中struct尾部成员时char member[1]的使用问题

	struct name {
		int namelen;
		char namestr[1];
	};
在网上搜索了一些[资料][c_last_data_url]  
在看Python2.7源码时，PyString\_FromString()函数在创建PyStringObject时（见stringobject.c），创建的struct大小是最后一个成员的偏移位置再加1，最初感觉不太正确。就拿上述struct来说（该结构与PyStringObject类似，也是最后为一个大小为1的char数组），为了放置该struct，malloc了大小为5字节的空间（int占4字节），最初认为考虑内存对齐的话应该是8字节才对。仔细想来，若是malloc的，确实只需分配5个字节就能放下该struct，然而，要是放在栈上（也即不是通过malloc的struct），那么编译器会考虑内存对齐，此时会额外占用3个字节。

#C++  
##基本概念  

* value initialized   

* top-level const和low-level const  
	int * const p = &i;	//top-level  
	const char *pc;	//low-level
	
###类型转换（Type Conversions）  

* 数值类型转换的基本规则    

	（1）对bool类型的变量，若对其赋值为非bool型的数值型值（nonbool arithmetic types），则0表示false，其它为true；  
	（2）将bool值赋值给非bool型数值型，则true为1，false为0；  
	（3）将浮点型赋值给整型，则小数部分会去掉，只保留整数部分；  
	（4）将整型赋值给浮点型，则小数部分为0，而整数部分则根据浮点数的精度会有所调整。  
	（5）对于无符号整型进行赋值时，若超过其范围，则赋值结果为余数。  
	（6）对于有符号整型，若超出其范围，则**结果未定义**。   
* 字面值的类型  
	（1） 几个例子  
		① 20，根据该字面值所在的表达式，从int, long, long long三者中选择适合的最小的类型。  
		② 024（octal），0x14（hex），从int, unsigned int, long, unsigned long, long long, or unsigned long long六者中选择适合的最小的类型。  
		③带后缀或者前缀的字面值，字符或者字符串可以带前缀，而整型或者浮点型可以带后缀，带后缀后比①和②的规则更明确。   
		

* 数值类型的优先级  
	（1）bool与其它整型之间无法比较优先级，因为要看具体情况，例如条件判断中，其它整型会转换为bool，而计算中则bool会转换为数值；  
	（2）unsigned 比signed优先级高，运算时，后者会先转换为前者。两个unsigned进行减法计算时注意不要出现负值（负值会wrap around）。  
<font color='red'>数值型还有待完善</font>  

* Explicit Conversions（显示转换）  
	格式：cast-name<type\>(expression)   
	（1）static\_cast  
		较常见的用途，数值类型直接的转换，void *类型的指针转换为具体的类型（注意，必须要保证转换到了正确的类型，否则行为未定义）
	（2）const\_cast  
		仅用于对low-level const进行转换（一般用于const类型的重载函数）。注意，若指向的object原本就是const，对其进行写入的话，行为是未定义的。  
	（3）reinterpret\_cast  
		performs a low-level reinterpretation of the bit pattern of its operands.注意，该操作是机器相关的，使用时要注意。	  
	（4）dynamic\_cast  
		将基类指针或引用转换为派生类指针或引用。转换失败的结果是0（对于指针），或者抛出bad\_cast异常（对于引用）。指针指向的实际类型为基类时，不能将其该指针转换为派生类来使用。      
	（5）Old-Style Casts  
		这种转换的写法不推荐使用  
		type (expr); // function-style cast notation  
		(type) expr; // C-language-style cast notation   
		以上两种写法适用于const\_cast, a static\_cast, or a reinterpret\_cast，根据使用场景，自动匹配，若前两者不匹配，则执行reinterpret\_cast  
##Dynamic Memory  
本小结主要以<<C++ Primer (5th, EN)>>的第12章进行总结。  
###内存区域  

* Static memory  
	Static memory is used for local static objects (§ 6.1.1, p. 205), for class static data members (§ 7.6, p. 300), and for variables defined outside any function.编译器进行管理，使用前分配，程序结束时释放。  

* Stack memory  
	Stack memory is used for nonstatic objects defined inside functions.编译器进行管理，语句块在执行时，该块中定义的变量才有效。  
* heap（也称free store）  
	内存池，由程序员管理。  
	使用动态内存的三种情况：  
	（1）无法确定需要创建的objects的数量  
	（2）无法确定所要创建的object类型  
	（3）在多个objects直接共享数据时

###Smart Pointers  

* 定义（smart pointer）  
	用于动态内存管理，Library type that acts like a pointer but can be checked to see whether it is safe to use. The type takes care of deleting memory when appropriate.  
	在C++11中，头文件memory中定义了三种智能指针（模板类），shared\_ptr, unique\_ptr和weak\_ptr。  

* shared\_ptr  
	默认值为nullptr，  

	(1)deleter  
	functor（仿函数），参数为managed object对应类型的指针。  
	
		template <class D, class T>
  		D* get_deleter (const shared_ptr<T>& sp) noexcept;//该函数用于提取sp的deleter

	
		template <class T> class default_delete;//functor，使用默认的delete来对T进行删除
		template <class T> class default_delete<T[]>;//functor，使用delete []来进行删除  

#参考资料  
[c_last_data_url]:http://c-faq.com/struct/structhack.html