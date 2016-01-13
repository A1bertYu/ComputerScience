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
关于C++ Object所占用的空间，需要考虑如下三个因素：(1)语言本身所造成的额外负担，包括指向虚函数表的vptr和指向virtual base class subobject的指针；(2)alignment的限制；(3)编译器对于特殊情况的优化处理。比如，没有任何数据成员的object会被安插1个char类型的成员，用于在内存中占据位置。  
对于nonstatic data members，直接存放于每一个class object之中，且尽量考虑空间优化、存取速度优化以及C语言中struct数据配置的兼容性。对于static data members，则被放在程序的一个global data segment中，即使该class没有任何object实体，其static data members也已经存在（非const的static data member，需要在类的外部进行定义，若没有在外部定义，相当于只申明了，此时也不会存在）另外，函数中的局部变量在函数调用时才会产生。
####The Binding of a Data Member
对member functions本身的分析，会直到整个class的声明都出现了才开始。因此，在一个inline member function函数体内的一个data member绑定操作，会在整个class声明完成之后才发生。但是对于函数的参数列表类型(包括形参类型和返回类型)，则在第一次遇到时就决议（resolved），而不是等到class声明结束。因此，对于typedef要采取防御式编程，即class声明体内若有typedef，放在最开始，以免与全局的typedef冲突。
####Data member layout
nonstatic data member在class object中的排列顺序与其声明顺序一致。C++ Standard要求，在同一个access section（也就是public, protected, private，若声明了两个public，则算两个access section。在各access section中，data members的排列只需符合“较晚出现的members在class object中有较高的地址”，至于地址连续与否，不做要求，并且access section级别的顺序，可以不按声明的顺序。编译器合成的data members（如vptr），其放置位置是任意的，C++ Standard没做要求。在P109-P110，讲述了vptr放置位置的两种实现，cfront是将其放置在末尾，这样可以保留base class C struct，兼容C代码；MS的实现则是放在最前面，因为很少会从C struct来派生一个多态性质的class。  
关于比较两个nonstatic data members的内存地址，书中提供的程序编译不能通过（GCC与VS都不行），自己稍作修改了一番，证实可行。

        template<class T, class T1, class T2>
        int access_order(T1 T::*mem1, T2 T::*mem2)
        {
            T1 **pp1 = (T1**)&mem1;
            T2 **pp2 = (T2**)&mem2;
            T1 * p1 = *pp1;
            T2 * p2 = *pp2;
            if (p1 > p2)
                return 1;
            else if (p1 < p2)
                return -1;
            else
                return 0;
        }
对于data member的指针，除了type之外，还需要加上class前缀，比如Point3d中的float* ，则对外显示是Point3d::float* ，这样导致该类型无法直接转成float*（即普通的，无class前缀），且无法对这种类型的指针进行比较（GCC会报错），这种限制可以增加一层间接性进行解决，解法见上述代码，即双重指针之后，解引用一次，便可以获得data member指针指向的地址。上述函数的使用方法，access_order(&Point3d::x, &Point3d::y)
####Data member的存取
* Static Data Members  
对于static data members，通过指针和通过一个对象进行存取，两者效率是一样的，因为static data members并不在class object中，语法上通过'.'和'->'只不过方便书写，最终编译器是通过ObjectName::staticname来进行存取的。  
若获取一个static data members的地址，将会得到一个指向其数据类型的指针，而不是一个指向其class member的指针，因为static data member并不在class object中。

* Name mangling  
<font color='red'>此处待完成</font>
[Name Mangling][name_mangling_url]

* Nonstatic Data Members  
在存取非静态数据成员时，可以带object(object.mem，称为explicit)或者不带object前缀(mem，称为implicit，其实编译器最后转成了this->mem)。关于指向data member指针其offset加1等以后<font color='red'>再补充</font>。对于非静态数据成员（包括单一继承或者多重继承，但不包括虚基类继承），其offset在编译期间便可获知，因此，其存取效率与C语言中的struct成员是一样的。  
        
        Point3d origin;
        Point3d *pt = &origin;
        origin.x = 0;
        pt->x = 0;
上述代码用于比较通过object和pointer这两种方式的存取效率，若Point3d是一个derived class，而在其继承结构中有一个virtual base class，并且被存取的member（上述中的x）是从该virtual base class继承而来的member时，两者效率就会有重大差异，通过指针的方式必须要到执行期才能确定offset。其它情况下，二者没有差异。<font color='red'>第一，origin如何确定offset？第二，一个积极进取的编译器甚至可以静态地经由origin就解决掉对x的存取？（P99）</font>
####继承与Data member
>在C++继承模型中,一个derived class object所表现出来的东西，是其自己的members加上其base class(es) members的总和，至于derived class members和base class(es) members的排列次序并未在C++ Standard中强制指定。（P99）  

* Inheritance without Polymorphism  
	一般而言，具体继承（concrete inheritance，相对于虚拟继承 virtual inheritance）并不会增加空间或存取时间上的额外负担，并且使用者并不需要知道这些类是否有继承关系，其表现跟独立的类一样。书中讲了两点这方面的缺点，第一点说是经验不足的人可能会重复设计一些相同操作的函数，并谈了inline设计的重要性（<font color='red'>第一点真没看懂，书中的例子感觉设计比较合理，难道我经验不足？</font>）；第二点将class分为多层后，可能会膨胀，这点主要是讲了**在derived class中，需要保持base class subobject的原样性**。若不能保证，即原来在base class subobject中有些alignment的空间，为了充分利用，将derived class中新的数据成员放置于此。这样，若在进行derived class object复制时，在复制base class subobject阶段，则alignment区域的值未定义，这样复制过去会覆盖新的数据成员的值，导致bug。

* Adding Polymorphism  
 
	在继承关系中，提供virtual function，来支持OO Paradigm。这样，需要对class做一些处理，以便支持这样的弹性，数中列举了四个方面，分述如下：  
	 （1）为class导入一个virtual table, 用于存放它所声明的每一个virtual function的地址，这个table的元素数目一般而言是被声明的virtual functions的数目，再加上一个或两个slots（用以支持type_info）  
	（2）为class导入一个vptr，提供执行期链接，使每一个object能够找到相应的virtual table  （3）加强ctor，使其能够设定vptr的初值，使其指向virtual function table  
	（4）加强dtor，使其能够释放vptr。  
	<font color='red'>关于vptr放在最前端的好处，还待补充（看完4.4节后）</font>  
 
* Multiple Inheritance  
	单一继承提供了一种自然多态（natural polymorphism）形式，是关于classes体系中的base type和derived type之间的转换。若base class object和derived class object都是从相同地址开始，也就是derived class object中包含的base class subobject在最前面，此种情况下，将derived class object的地址赋值给base class pointer时，编译器不需做任何调整，而且效率也最佳。但当derived class object最开始不是包含base class subobject时（例如，base class没有虚函数，即没有vptr，而derived class中有，而编译器又将vptr放置在最前面），此时就需要编译器对指针所指地址进行调整。   

	多重继承的复杂度在于derived class和其上一个base class乃至于上上一个base class之间的非自然关系。多重继承的问题主要发生在derived class object和其第二或后继的base class object之间的转换，不论是derived class object到base class subobject的转换，还是virtual function机制做转换。

	对一个多重派生对象，将其地址指定给最左端（也就是第一个）base class（或者该class单一父类（即该class是单一继承的基类）），情况和单一继承时相同。至于第二个及以后的base classes的地址指定操作，则需要修改地址（调整偏移量），因为这些base class subobjects在中间，而不在开头。（书中P114举的例子很好，且举了一个派生类指针赋值给基类指针的例子）**注意，C++ Standard并未要求多重派生类在布局base class subobjects时，按照声明顺序来进行，但主流厂商都是遵照此种方式，因此，有了本段所述的情况。**

	<font color='red'>关于P116所说的一种调整base class布局的优化技术，使用后可少一个vptr，实在没看懂</font>

	第二个及以后的基类之data member，其位置（offset）在编译期就已经确定，存取成本只是一个简单的offset计算，就像单一继承一样简单，不管是经由一个指针、一个引用或者一个object来存取。**注意，此处没有考虑下一节所讲述的虚拟继承（Virtual Inheritance）**

* Virtual Inheritance  
	对于虚拟继承，一般的实现方法如下，class若内含一个或多个virtual base class subobject则会分为两个部分进行布局，一个不变局部和一个共享局部。不变局部不管后续如何演化，总是拥有固定的offset（从object的开头开始算起），所以这一部分数据可以直接存取。至于共享局部，所表现的就是virtual base class subobject。这一部分数据，其位置会因为每次的派生操作而有变化，因此它们只能间接存取。编译器厂商的差异就在于间接存取的方法不同。

	cfront的策略：在派生类中，为每一个直接的虚拟基类安放一个指针，通过该指针存取虚拟基类的数据成员。缺点：（1）派生类需要为每个virtual base class背负一个额外的指针，负担大小与虚拟基类的数量成正比；（2）对虚拟基类数据成员的存取时间，会随继承链的加长而变大。

	cfront策略缺点的解决之道：缺点（1）：Microsoft通过引入virtual base class table,每一个class若有一个或多个虚拟基类，则会安插一个指针，指向该表。另一种解决方法，是在virtual function table中，放置virtual base class的offset（而不是地址）。<font color='red'>注：书中说后一种方法存取成员较为昂贵，没看出来与Microsoft哪里昂贵了？</font>缺点（2）：经由拷贝操作取得的所有的nested virtual base class指针，



####Object member efficiency
##参考资料

[Opaque_pointer_url]:"https://en.wikipedia.org/wiki/D-pointer"
[name_mangling_url]:"https://en.wikipedia.org/wiki/Name_mangling"