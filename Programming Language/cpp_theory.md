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
	
	**个人理解**：关于多态，使用指针或引用而不用object，类似于浅拷贝和深拷贝的关系，即多态的话只能浅拷贝，且浅拷贝非常容易实现。
	
###The Semantics of Constructors
本章中使用比较频繁的术语：implicit, explicit, trivial, nontrivial, memberwise, bitwise和semantics。  
implicit是指编译器在某些情况下会对现有代码做一些额外处理，比如conversion；  
explicit主要是指C++中的关键字；  
trivial，是指类型为built-in类型以及与C语言中struct对等的类型（struct中不能包括具有default constructor的数据成员）。
nontrivial，与上述相反。
memberwise，bitwise和semantics直接从单词字面理解。
####Default Constructor的建构操作  
在用户没有定义default constructor时，编译器会根据情况进行default ctor的合成，并且只有在nontrivial的类型时，编译器才需要进行合成。对于nontrival类型中的trivial数据成员，或者trivial类型，则是程序员需要进行处理的（也就是需要初始化）。下面介绍四种编译器需要合成的default constructor，也就是用户没有定义default constructor且该类型为nontrival。
  
1. 带有default constructor的member class object  
	class中有带有用户已定义default constructor的数据成员。对这种class的default constructor的合成操作，只有在被调用时才会发生。   
	对于多个编译模块（不同.cpp文件），需要避免合成多个default ctor。解决方法有两种，法一是将合成的default ctor(或者copy ctor, dtor, assignment copy operator)以inline方式完成；法二则应对不适合inline的情况，合成为explict non-inline static实体。  
	**对于自动合成所产生的代码，编译器都是将其放置在user code之前。**  
	若class中包括具有default ctor的多个data member，按成员的声明顺序添加成员对应的default ctor代码。
2. 带有default constructor的base class
	若derived class没有定义default ctor，而其base class定义了，此时编译器也需要进行default ctor的合成。
3. 带有一个virtual function的class
	class声明（或继承）一个virtual function；class派生自一个继承串链，其中有一个或更多的virtual base classes。这两种情况都需要合成default ctor。  
	编译器会对所有ctor（包括用户已定义的非default ctor）进行扩张，对class object中添加数据成员vptr，用于指向虚函数表，并且虚函数表也是有编译器产生出来的。  
4. 带有一个virtual base class的class  
	在derived class中，virtual base class subobject不能沿用非virtual的base class subobject的实现方式（若沿用的话，在菱形继承链中，可能出现末端的derived class包含两次virtual base class subobject的情况），因此，涉及到对virtual base class的成员访问时，需要做特殊处理。有些编译器是这样实现的，在derived class object的每一个virtual base classes中，安插一个指针，通过该指针完成对virtual base class的成员操作。
	
####Copy Constructor的建构操作
有三种情况，会调用copy ctor，分别是赋值操作符，函数传参以及函数返回。若用户没有提供一个explicit copy ctor，编译器会按memberwise的方式实施copy ctor。对于class中的data member成员，递归的进行memberwise操作（<font color='red'>若是定义了copy ctor的data member， 在VS2010上测试是直接调用该成员的copy ctor，而不是递归的memberwise</font>）。
>一个良好的编译器可以为大部分class object产生bitwise copies，当class不展现bitwise copy semantics时，default ctor和copy ctor才由编译器产生出来。  

* 不会Bitwise copy semantics的四种情况  
	与合成default ctor的四种情况类似，注意对比。  
	（1）class中包括copy ctor的member object（不论是用户自定义还是编译器合成的，注意，要清楚编译器在何时需要合成copy ctor）  
	（2）class继承自一个base class，而后者也有copy ctor（不论是用户自定义还是编译器合成的）  
	（3）class声明了一个或多个virtual functions时。这种情况涉及到vptr的拷贝，若是对等类型（即名字一样的类型），依旧可以实行bitwise copy。但若不对等，比如将derived class object赋值到base class object，若也采用bitwise copy，便会出现问题，比如，derived class改写的virtual function中用到了derived class才有的data member。  
	（4）当class派生自一个继承串链，其中有一个或多个virtual base classes。
	>每一个编译器对于虚拟继承的支持承诺，都表示必须让"derived class object中的virtual base class subobject位置"在执行期就准备妥当。（P57）  
	
	与情况（3）类似，同类型的依旧可以bitwise，而对于derived class向base class赋值的情况，编译器必须要处理好virtual base class pointer/offset的初值。

若类的设计者提供了copy ctor（或者说是上述四种自动合成的copy ctor），而该类可以通过bitwise copy semantics进行正确的copy，那要不要调用copy ctor呢？（<font color='red'>争议话题</font>）

####Program Transformation Semantics
Named Return Value（NRV）优化，是标准C++编译器必须的功能（虽然正式标准未规定）。所谓NRV，即对于函数的返回对象，在不优化时会调用构造函数产生一个临时对象，之后再返回，在赋值完成之后，该临时对象析构销毁。NRV可以避免产生这种不必要的构造和析构行为，可以对比《More Effective C++》Item20.  
>对于memcpy()和memset()，只有在“classes不含任何由编译器产生的内部members”时才能有效运行。（P73）  

####Member Initialization List
>使用member initialization list的四种情况，①初始化一个reference member时；②初始化一个const member时；③调用一个base class的ctor，而它有一组参数时；当调用一个member class的ctor，而它有一组参数时（P75）  
>编译器会一一操作initialization list，以适当次序在ctor之内安插初始化操作，并且在任何explicit user code之前。（P77）注意，初始化顺序是数据成员的声明顺序。

###The Semantics of Data




##参考资料

[Opaque_pointer_url]:"https://en.wikipedia.org/wiki/D-pointer"