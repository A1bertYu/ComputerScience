CPP Theory
=

##Inside The C++ Object Model
Stanley B.Lippman （中文版，侯捷译，华中科技大学出版社，2001）

###Object Lessons

C++加入封装之后的成本并没有比C增加多少，其主要负担由virtual引起，包括virtual function机制和virtual base class.

####The C++ Object Model
C++中有两种成员，数据成员（data member）和成员函数（member function），前者可分为static和nonstatic，后者则分为static，nonstatic和virtual.关于这两种成员的布局方式，文中对比了三种模型，以及模型对程序的影响。

1. A Simple Object Model  
	一个Object是一系列的slots，每一个slot指向一个member，即放在object中的是指向member的指针。这样Object的大小就是指针大小乘以member数目。
	
	**继承模型**：对于base class，也是在一个slot中放置指向base class subobject地址的指针（继承链上的每一个base class，不论直接还是间接）。  当然，也可以使用另一种模型，例如类似虚函数表那样，使用基类表，再用bptr指向该表，而基类表中放置直接继承的类对象地址，再通过这些地址，访问其对应对象的bptr，进而可以获取间接继承的对象的地址。
2. A Table-driven Object Model  
	一个Object有两个指针，分别指向data member table和member function table。在数据成员表中，依次存放data本身；在成员函数表中，则是一系列solt，存放指向成员函数的指针。
	
	这种模型的优点就是data member的增加、移除或者修改之后，使用该class的代码无需重新编译。从封装的角度来说，一般data member是隐藏的（private），而且最有可能修改的也是data members（增删改），所以这种模型，通过增加一层间接性，使得依赖性降低。QT中的d指针（[Opaque pointer][Opaque_pointer_url]）就是从代码层面上将Object Model变成了A Table-driven Object Model。
3. The C++ Object Model  

	该模型是当前主流编译器普遍采用的模型。Nonstatic data members放置于每一个Object之内，static data members则存放于所有Object之外，对于非virtual的member function，则不论static或nonstatic，都是存放于所有Object之外。virtual function（OO[OO paradigm请参考后面章节]的class才有），在Object中会添加一个vptr（data member），用于指向虚函数表，表格中的每一个slot存放指向不同虚函数的指针（即A Table-driven Object Model中的概念）。关于vptr的维护，则由编译器自动完成。

	对于OO型class关联的type\_info object（用于支持RTTI），是放在虚函数表中的某个slot的。注意，非OO型的Object（或者built-in type），则不需要RTTI，因为RTTI的操作符typeid在编译时期就可以计算出非OO型的type_info，故不需要额外的空间来存储非OO型的RTTI。可以这么理解，RTTI对OO型对象才有意义。

	这种模型没有双表格模型（Table-driven）有弹性，对data members进行更改后需要重新编译使用了该class的程序。

	**继承模型**：直接将base class的data members放置于derived class object中。优点是存取效率高，缺点是缺乏弹性，base class的data members的任何改变，derived class的object需要重新编译。关于虚继承，最初模型。。。，演进的模型。。。<font color='red'>（待完善）</font>

####An Object Distinction
C++程序的三种programming paradigms，分别为procedural model, abstract data type model和object-oriented model。

1. procedural model  
	过程模型，即C++中的C语言部分，过程型。

2. abstract data type model，ADT（也称object-based）  
	将数据和相关函数进行封装，并通过提供public接口，供客户使用。

3. object-oriented model  
	在上述ADT基础上，增加了继承关系，并通过使用虚函数来支持多态。在C++中，需要借助指针和引用才能使用多态，且多态只存在于public class体系，nonpublic派生行为和void *指针可以说是多态，但它们并没有被语言隐式支持，使用时需要显式的转型操作（多态的准一线选手）。相比于ADT的编译时确定，OO的多态机制需要在运行期才能确定。

4.  指针与多态  
	指针不论何种类型，占用内存空间都是一样的，其类型只是告诉编译器如何解释其指向地址中的内存内容以及大小。因此，转型（cast）是一种编译器指令（编译期确定），也就是影响解释方式。

	多态通过基类指针或引用完成，通过vptr所指之virtual table，来调用具体的虚函数。为什么不是通过object本身呢？比如直接将derived class object的vptr赋值给base class object的vptr，这样赋值之后便可通过base class object实现多态，从而无需指针或引用。为什么不这样呢？这是因为编译器必须确保derived class object的vptr（一个或者多个）的内容（即指向的虚函数表）不会被base class object初始化或改变。若是直接赋值给基类，那要是基类调用了派生类的虚函数，而该虚函数中使用了派生类才有的数据成员，就会出现问题。并且，派生类的虚函数表可能会增加虚函数，也就是基类没有的虚函数。（P56）<font color='red'>不赋值就不会被改变。</font>
	>一个严格的编译器可以在编译时期解析一个“通过该object而触发的virtual function调用操作”，因而回避virtual机制。（P34）  
	>对于这句话的理解，应该是说可以在编译期间确定的virtual调用，就放在编译期。
	
###The Semantics of Constructors
本章中使用比较频繁的术语：implicit, explicit, trivial, nontrivial, memberwise, bitwise和semantics。  
implicit是指编译器在某些情况下会对现有代码做一些额外处理，比如conversion；  
explicit主要是指C++中的关键字；  
trivial，是指类型为built-in类型以及与C语言中struct对等的类型（非OO的ADT）。
nontrivial，与上述相反。
memberwise，bitwise和semantics直接从单词字面理解。
####Default Constructor的建构操作
1. 编译器需要以及程序员需要
	
##参考资料

[Opaque_pointer_url]:"https://en.wikipedia.org/wiki/D-pointer"