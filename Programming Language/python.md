#The Python Tutorial  
##基本语法  

* multiple assignment
	
		a, b = 0, 1
		a, b = b, a+b	#等号右边的先计算，且遵循从左到右的计算顺序。在右边计算完成后，再进行赋值操作。  
* True and False  
	整型值：非0即为True，0为False  
	sequences：非空即为True，空则为False  
	
##Strings  
下标寻址返回的值与C++标准库中的string类似，因为不可修改，故不能对该返回值赋值，即使通过slice的形式获取超过size的返回值，例如word='python'，则执行word[8:]='xx'，是不可以的。另外，单个元素的下标寻址，下标是不允许越界的。  
##Lists  
lists是mutable的，可以对其下标返回的值进行修改，当然，修改后list的值会被改变。若是单个下标寻址，则下标不允许越界。对于slice的返回值，若对下标范围超出了list大小的返回值进行赋值，则将这些值加入到list尾部。   
A list comprehension consists of brackets containing an expression followed by a for clause, then zero or more for or if clauses. 若最前面的表达式是一个tuple，则必须要加括号。  
另外，还有Nested List Comprehensions  
##Tuple   
multiple assignment is really just a combination of tuple packing and sequence unpacking.
##控制语句  
for和while可以接else，若循环不是因为break结束的话，就会在结束后执行else语句块。try语句也可以接else，若没有exception发生，才会执行else语句块。  
##函数  
在函数执行时，会为local变量产生一张符号表，在查找变量时，先查该表，若没找到，则从全局表中查询；若还是没有查到，最后则查询内置名称的表（built-in）。
Python中的函数参数与Java相同，传递的是引用，也就是说，对于mutable的对象，在函数体内修改后会反映到调用者那边。实参的名称会加入到 local symbol table，函数的名称也会加入到该表。  
对于没有显式return的Python函数，其返回None  

* 默认参数  
	默认参数值在函数定义所在的作用域被计算（注意，这里与C++有很大的不同，其默认参数值可以是变量，也就是可以在运行时才能知道值的变量），在函数定义处，变量此时取何值，那么默认参数就是何值，且默认参数只会被求值一次。  

* Keyword Arguments  
	相对于通过位置信息来传参（C++只支持这种，因为C++并不记录形参的具体名称），Python支持通过形参名称来指定参数值，这样，就可以不通过位置信息来传参。当然，没有指定名称的参数值，只能靠位置形式来传递。  
	keyword arguments must follow positional arguments.  
	（1）当最后一个参数名称是"\*\*name"这种形式时，表示其接收的是一个dictionary；当参数名称是"*name"时（若同时有"\*\*name"形式的参数，则必须要在"\*\*name"参数前面），接收一个tuple。  

* Arbitrary Argument Lists  
	显然，"*name"的参数接收一个tuple，而tuple变长，则通过该机制可以实现变长参数的函数。函数的其它参数，只有Keyword Arguments才能在"\*name"参数之后。  

* Unpacking Argument Lists  
    有时候，参数值在list或者tuple中，那么，在函数调用时，还要将这些值从list或tuple中取出来，再一一传递，可以通过在list或tuple名称前面加*，以此将元素一一取出，注意，要注意通过slice来控制取出的元素数量，以满足函数调用需求。同样，对于keyword arguments，可通过dictionary传递，unpack元素时在dictionary名称前加\*\*  
* Lambda表达式  
	只能有一个表达式。注意，在lambda中，捕捉的外部变量(enclosing function)即为[nonlocal][nonlocal_url]。在非lambda的[closure][Closure_url]中，需要显式的使用nonlocal关键字来声明nonlocal类型的变量。
* Function Annotations  
	对于函数参数和返回值的注释，每个待注释的参数后面加冒号，之后跟一个表达式；
##Modules  

**main module**:the collection of variables that you have access to in a script executed at the top level and in calculator mode.   
A module is a file containing Python definitions and statements. 文件名称即为module名称。在执行时，可以通过全局变量\_\_name\_\_来获取module名称，当然，若是main module，则名称为"\_\_main\_\_"  
module中除了可以包含函数的定义之外，还可以包含可执行的语句（也就是函数和类之外的语句，这与C++有很大不同（C++只允许在外部定义全局变量））。module中的可执行语句在作为脚本运行或者import时被执行，且只执行一次。当然，对于函数（module-level function ），其实也可以认为被执行了，因为函数名称会加入到module的全局符号表中。  
在import语句中，对于module名称的搜寻，会按以下顺序展开：  
（1）首先搜寻built-in module   
（2）其次，在sys.path路径中，搜寻与module名称相同的py文件。包括：当前被执行的脚本所在的路径（注意，对标准库的搜寻是在这之后，因此，该路径下的module名称最好不要与标准库中的同名）；PYTHONPATH环境变量中的路径；python安装目录中的默认路径<font color='red'>（不懂这个路径是什么）</font>。  
##Packages  
用于对module的管理。若某个目录包含Package，则需要在该目录中编写\_\_init\_\_.py文件。
语句

	import item.subitem.subsubitem #除最后一个item之外，其余都必须为package，而最后一个可以为package或者module  
##Errors and Exceptions  
The use of the else clause is better than adding additional code to the try clause because it avoids accidentally catching an exception that wasn’t raised by the code being protected by the try ... except statement.  
这句话的理解：我们知道，else语句是在try块没有发生异常才执行的，也就是说，else语句也可以放在try块里面（最后面，两者等效的）。但是，若else语句里面发生了exception（else语句全部放在try内的最后面，取消else），而这些exception不是try...except能处理的，这样就使得try...except的机制显得不完善。若使用else语句，即使发生了exception，但是这跟try...except无关。其实，个人感觉二者没什么区别。
##Classes
成员默认为public，所有成员函数为虚函数。不同于C++，成员函数的第一个参数必须要显式的指定为当前实例（名称不强制，但惯例用self）.  

* Python Scopes and Namespaces  
	global namespace是相对module来说的，每个module都有一个global namespace，module的属性与其全局变量对应（只有一个属性例外，也就是\_\_dict\_\_，所有全局变量都在该map中，故其不能称为一个全局变量）。  
	对于built-in namespace，其所在的module称为builtins，在Python解释器启动时创建。对于一般的module，则是执行module语句时创建namespace，而该module语句执行完毕后删除namespace。  
	local namespace是相对于函数来说的。  
	
	A scope is a textual region of a Python program where a namespace is directly accessible. 所谓直接访问，是指name没有限定词。  
	在搜寻时，首先是最里层的函数，只包含local names。其次是外层的函数（注意，python中函数的定义与C++有不同之处，即函数中可嵌套定义其它函数，内嵌的函数即使不被外部调用，也会），在外层函数体中定义的变量，若要在其内嵌函数中修改，则需要在内嵌函数中使用nonlocal声明（若内嵌函数声明了nonlocal变量，而外层函数在调用该内嵌函数前又不定义该变量，则会出现错误）。global变量若在函数体内定义，则需要使用global前缀。    
* function object和method object  
	前者类似于静态成员函数的概念，而后者则相当于非静态成员函数，不过，这与C++还是存在一些区别，Python中class的成员函数没有静态非静态一说，只是与调用方式有关，其包括两种方式，一是通过类的名称调用，此时，就需要显式的指定该类的某实例化对象为调用函数的第一个参数；而若是通过实例化对象调用，则Python会隐式的将该对象指定为调用函数的第一个参数。   
	
* 关于名字冲突的问题  
	Python中无法对数据进行隐藏（nothing in Python makes it possible to enforce data hiding — it is all based upon convention，名字以单下划线打头的成员，无论是数据成员还是成员函数，都应当以non-public来对待），class中数据成员名称可能会与成员函数名称冲突，因此，要约定好命名习惯，以避免错误发生。  
* 私有变量  
	Python的class无私有变量一说，但是，当成员名称以两个下划线或者三个下划线打头时，Python会对这种变量进行name mangling，即给该变量名称加上前缀"\_classname"。在外部进行访问时，依然可以通过name mangling后的名称来访问变量。  
	<font color='red'>9.6最后一段没看懂</font>	

#The Python Language Reference  
##Lexical analysis  
source file 默认为UTF8编码。

* 物理行与逻辑行    
	Python程序以逻辑行进行组织，关于逻辑行（以\n结尾）和物理行（Win平台以CRLF结尾，Unix以LF结尾）的区别，可以参考"A byte of python"中的例子。  
	多个物理行可以合并为一个逻辑行，在行尾加上"\"字符（注意，行尾为"\"字符的行，尾部不准有注释）。   
	当然，对于在(),[],{}中的表达式，可以不用"\"字符，而将多个物理行合并为一个逻辑行。  
* Indentation（缩进）  
	indentation is Python’s way of grouping statements.   
	制表符和空格用于计算缩进，制表符被替换为1到8个空格，以使得总空格数是8的整数倍，空格数用于determine the grouping of statements。第一句声明决定了缩进大小。制表符和空格最好不好混用！   
	Note that each line within a basic block must be indented by the same amount.
* Reserved classes of identifiers  
	（1）形式为\_*的标示符（identifier ）    
		语句from module import *不会导入这种标示符。  
	（2）形式为\_\_\*\_\_的标示符  
		系统使用，用户不要使用  
	（3）形式为\_\_\*的标示符  
		作为class的私有成员，Python会mangle这种成员的名称，以保证基类和派生类中不会有同样的名字。  
##Data model
每一个object都有an identity, a type and a value. is操作符比较的是两个对象的identity，id()函数返回的是identity值（整型表示，CPython返回的是内存地址）。type()返回的是Type Object（内置类型）。  

###Numeric
Numeric objects are immutable.

* numbers.Integral（整型）  
	包含int 和 bool两类，其中int表示的范围仅受限于可用虚拟机内存的大小。int也能用于位操作，对于负数，可以看做左边有无数个1（因为Python中的int是没有位数之说的）  
* numbers.Real (浮点型)   
	float，双精度浮点，精度与机器有关。Python没有单精度浮点。  
* numbers.Complex (complex)   
	实部和虚部都是双精度浮点型。
  
###Sequences   

可分为Immutable sequences和Mutable sequences两类。  

* Immutable sequences   
	If the object contains references to other objects, these other objects may be mutable and may be changed; however, the collection of objects directly referenced by an immutable object cannot change.  
	即sequence中的元素若是指针之类的，指针所指向的内容可以更改，但是指针本身不能更改。  
	（1）Strings  
	Python中没有char类型，String中的每一个元素代表一个Unicode code （范围U+0000 - U+10FFFF ），每个元素长度为1。  
	（2）Tuples  
	元素可以是任意的Python对象，不要求所有元素都是一个类型。  
	（3）Bytes  
	每个元素都是8-bit的byte，取值为[0,256)。
* Mutable sequences   
	The subscription and slicing notations can be used as the target of assignment and del (delete) statements.  
	（1）Lists   
	元素可以是任意的Python对象，不要求所有元素都是一个类型。  
	（2）Byte Arrays   
	除了Mutable属性不同外，其接口与Bytes相同。  
###Set  
These represent unordered, finite sets of unique, immutable objects.   
常见的用途：Common uses for sets are fast membership testing, removing duplicates from a sequence, and computing mathematical operations such as intersection, union, difference, and symmetric difference.  
内置了两种set，Sets（mutable）和Frozen sets （immutable）  
###Mappings  
* Dictionaries  
	当前只有该内置类型。对于key的类型，有如下一些限制：  
	（1）lists or dictionaries，或者不能通过identity进行比较的mutable types ，这些都不能作为key。之所以有这些限制，是为了效率考虑，要求key的hash值保持为常量。  
###Callable types   

* User-defined functions （自定义函数）  
	系统定义了一些特殊属性，有些属性可以修改。get和set这些属性时通过点号"."操作符进行。

		def yxlfun():
    		'''it is my function'''
			yxl7fun.__name__="yxl"
		yxlfun.__name__ = "newyxl"
		print (yxlfun.__name__)
		print (yxlfun.__doc__)
* Instance methods   
	An instance method object combines a class, a class instance and any callable object (normally a user-defined function).   
* Generator functions    
* Built-in functions   
* Built-in methods   
* Classes   
* Class Instances   
 
###Modules  
Modules are a basic organizational unit of Python code.  
A module object has a namespace implemented by a dictionary object（在）. 定义在module中的函数，可以通过get属性\_\_globals\_\_来查看当前命名空间中的变量。也可以通过modulename.\_\_dict\_\_获取命名空间的信息。module还有其它属性：\_\_name\_\_（该属性可修改）用于获取module的名称，\_\_doc\_\_获取文档，\_\_file\_\_用于获取被载入的module所在的文件，然而，对于动态库或者C modules，该属性是相关的路径。  
###Custom classes   
与module类似，自定义的class也通过dictionary来实现命名空间，可通过C.\_\_dict\_\_来获取。当对一个变量进行查找时，若在当前class中没有找到，则往上向基类去查找。在基类中查找的方法采用C3 method resolution order ，即使是菱形继承，也能正确查询。  

* [Method Resolution Order][MRO_url]  
	 对于class C，将其ancestor由近及远进行排列（包括class C自身），由此形成的list称为 precedence list（也称the linearization of C）。MRO（The Method Resolution Order）便是用于构建该list的规则。在使用到较为复杂的继承时，通过该规则对成员名称进行决议。

	
#资料
[MRO_url]:https://www.python.org/download/releases/2.3/mro/  
[nonlocal_url]:"https://en.wikipedia.org/wiki/Non-local_variable"
[Closure_url]:"https://en.wikipedia.org/wiki/Closure_(computer_programming)"