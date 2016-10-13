CPP Theory
=

##Inside The C++ Object Model
Stanley B.Lippman （中文版，侯捷译，华中科技大学出版社，2001）

###第1章 Object Lessons

C++加入封装之后的成本并没有比C增加多少，其主要负担由virtual引起，包括virtual function机制和virtual base class.

####1.1 The C++ Object Model
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
	
###第2章 The Semantics of Constructors
本章中使用比较频繁的术语：implicit, explicit, trivial, nontrivial, memberwise, bitwise和semantics。  
implicit是指编译器在某些情况下会对现有代码做一些额外处理，比如conversion；  
explicit主要是指C++中的关键字；  
trivial，是指类型为built-in类型以及与C语言中struct对等的类型（struct中不能包括具有default constructor的数据成员）。
nontrivial，与上述相反。
memberwise，bitwise和semantics直接从单词字面理解。
####2.1 Default Constructor的建构操作  
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
	
####2.2 Copy Constructor的建构操作
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

####2.3 Program Transformation Semantics
Named Return Value（NRV）优化，是标准C++编译器必须的功能（虽然正式标准未规定）。所谓NRV，即对于函数的返回对象，在不优化时会调用构造函数产生一个临时对象，之后再返回，在赋值完成之后，该临时对象析构销毁。NRV可以避免产生这种不必要的构造和析构行为，可以对比《More Effective C++》Item20.  
>对于memcpy()和memset()，只有在“classes不含任何由编译器产生的内部members”时才能有效运行。（P73）  

####2.4 Member Initialization List
>使用member initialization list的四种情况，①初始化一个reference member时；②初始化一个const member时；③调用一个base class的ctor，而它有一组参数时；当调用一个member class的ctor，而它有一组参数时（P75）  
>编译器会一一操作initialization list，以适当次序在ctor之内安插初始化操作，并且在任何explicit user code之前。（P77）注意，初始化顺序是数据成员的声明顺序。

###第3章 The Semantics of Data
关于C++ Object所占用的空间，需要考虑如下三个因素：(1)语言本身所造成的额外负担，包括指向虚函数表的vptr和指向virtual base class subobject的指针；(2)alignment的限制；(3)编译器对于特殊情况的优化处理。比如，没有任何数据成员的object会被安插1个char类型的成员，用于在内存中占据位置。  
对于nonstatic data members，直接存放于每一个class object之中，且尽量考虑空间优化、存取速度优化以及C语言中struct数据配置的兼容性。对于static data members，则被放在程序的一个global data segment中，即使该class没有任何object实体，其static data members也已经存在（非const的static data member，需要在类的外部进行定义，若没有在外部定义，相当于只申明了，此时也不会存在）另外，函数中的局部变量在函数调用时才会产生。
####3.1 The Binding of a Data Member
对member functions本身的分析，会直到整个class的声明都出现了才开始。因此，在一个inline member function函数体内的一个data member绑定操作，会在整个class声明完成之后才发生。但是对于函数的参数列表类型(包括形参类型和返回类型)，则在第一次遇到时就决议（resolved），而不是等到class声明结束。因此，对于typedef要采取防御式编程，即class声明体内若有typedef，放在最开始，以免与全局的typedef冲突。
####3.2 Data member layout
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
####3.3 Data member的存取
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
####3.4 继承与Data member
>在C++继承模型中,一个derived class object所表现出来的东西，是其自己的members加上其base class(es) members的总和，至于derived class members和base class(es) members的排列次序并未在C++ Standard中强制指定。（P99）  

* Inheritance without Polymorphism  
	一般而言，具体继承（concrete inheritance，相对于虚拟继承 virtual inheritance）并不会增加空间或存取时间上的额外负担，并且使用者并不需要知道这些类是否有继承关系，其表现跟独立的类一样。书中讲了两点这方面的缺点，第一点说是经验不足的人可能会重复设计一些相同操作的函数，并谈了inline设计的重要性（<font color='red'>第一点真没看懂，书中的例子感觉设计比较合理，难道我经验不足？</font>）；第二点将class分为多层后，可能会膨胀，这点主要是讲了**在derived class中，需要保持base class subobject的原样性**。若不能保证，即原来在base class subobject中有些alignment的空间，为了充分利用，将derived class中新的数据成员放置于此。这样，若在进行derived class object复制时，在复制base class subobject阶段，则alignment区域的值未定义，这样复制过去会覆盖新的数据成员的值，导致bug。

* Adding Polymorphism  
 
	在继承关系中，提供virtual function，来支持OO Paradigm。这样，需要对class做一些处理，以便支持这样的弹性，数中列举了四个方面，分述如下：  
	 （1）为class导入一个virtual table, 用于存放它所声明的每一个virtual function的地址，这个table的元素数目一般而言是被声明的virtual functions的数目，再加上一个或两个slots（用以支持type_info）  
	（2）为class导入一个vptr，提供执行期链接，使每一个object能够找到相应的virtual table  
    （3）加强ctor，使其能够设定vptr的初值，使其指向virtual function table  
	（4）加强dtor，使其能够释放vptr。  
	<font color='red'>关于vptr放在最前端的好处，还待补充（看完4.4节后）</font>  
 
* Multiple Inheritance  
	单一继承提供了一种自然多态（natural polymorphism）形式，是关于classes体系中的base type和derived type之间的转换。若base class object和derived class object都是从相同地址开始，也就是derived class object中包含的base class subobject在最前面，此种情况下，将derived class object的地址赋值给base class pointer时，编译器不需做任何调整，而且效率也最佳。但当derived class object最开始不是包含base class subobject时（例如，base class没有虚函数，即没有vptr，而derived class中有，而编译器又将vptr放置在最前面），此时就需要编译器对指针所指地址进行调整。   

	多重继承的复杂度在于derived class和其上一个base class乃至于上上一个base class之间的非自然关系。多重继承的问题主要发生在derived class object和其第二或后继的base class object之间的转换，不论是derived class object到base class subobject的转换，还是virtual function机制做转换。

	对一个多重派生对象，将其地址指定给最左端（也就是第一个）base class（或者该class单一父类（即该class是单一继承的基类）），情况和单一继承时相同。至于第二个及以后的base classes的地址指定操作，则需要修改地址（调整偏移量），因为这些base class subobjects在中间，而不在开头。（书中P114举的例子很好，且举了一个派生类指针赋值给基类指针的例子）**注意，C++ Standard并未要求多重派生类在布局base class subobjects时，按照声明顺序来进行，但主流厂商都是遵照此种方式，因此，有了本段所述的情况。**

	<font color='red'>关于P116所说的一种调整base class布局的优化技术，使用后可少一个vptr，实在没看懂</font>

	第二个及以后的基类之data member，其位置（offset）在编译期就已经确定，存取成本只是一个简单的offset计算，就像单一继承一样简单，不管是经由一个指针、一个引用或者一个object来存取。**注意，此处没有考虑下一节所讲述的虚拟继承（Virtual Inheritance）**

* Virtual Inheritance  
	对于虚拟继承，一般的实现方法如下，class若内含一个或多个virtual base class subobject则会分为两个部分进行布局，一个不变局部和一个共享局部。不变局部不管后续如何演化，总是拥有固定的offset（从object的开头开始算起），所以这一部分数据可以直接存取。至于共享局部，所表现的就是virtual base class subobject。这一部分数据，其位置会因为每次的派生操作而有变化（**关于这点可以看后面中的data binding的思考**），因此它们只能间接存取。编译器厂商的差异就在于间接存取的方法不同。

	cfront的策略：在派生类中，为每一个直接的虚拟基类安放一个指针，通过该指针存取虚拟基类的数据成员。缺点：（1）派生类需要为每个virtual base class背负一个额外的指针，负担大小与虚拟基类的数量成正比；（2）对虚拟基类数据成员的存取时间，会随继承链的加长而变大。

	cfront策略缺点的解决之道：缺点（1）：Microsoft通过引入virtual base class table,每一个class若有一个或多个虚拟基类，则会安插一个指针，指向该表。另一种解决方法，是在virtual function table中，放置virtual base class的offset（而不是地址）。<font color='red'>注：书中说后一种方法存取成员较为昂贵，没看出来与Microsoft比哪里昂贵了？成本分散到了“对members的使用”上（P122）？</font>缺点（2）：经由拷贝操作取得的所有的nested virtual base class指针，即包括指向直接继承的基类和间接继承的基类的指针。
	
	>上述每一种方法都是一种实现模型，而不是一种标准。每一种模型都是用来解决“存取shared subobject内的数据<font color='red'>（其位置会因每次派生操作而有变化？</font><font color='blue'>关于该问题的原因，思考时请注意书中该页的例子，有一个非多态的class object来存取一个继承而来的virtual base class的member，通过'.'操作符来引用member时，可以在编译期决议，这种情况下virtual base class subobjects的位置不会变化。对于非指针类型的变量，C++不会对其进行动态绑定，也即所有操作都在编译期决议完成，而对于具有虚函数或者虚基类的类，在通过基类指针访问成员时（虚基类包括数据成员，而虚函数则是成员函数），是在执行期间进行决议的。关于为什么需要移到执行期决议，请看本小节后面关于dynamic binding的思考。**更新**，问题原因是基类指针可以指向派生类A，也可以指向派生类B，这样因为基类指针可以通过不同的派生操作来指向不同的派生类，导致了shared subobject位置发生了变化。）</font>”所引发的问题。由于对virtual base class的支持带来额外的负担以及高度的复杂性，每一种实现模型多少有点不同。（P122）

<font color='red'>本节图3.5b待研究</font>  
关于dynamic binding，为什么在给基类指针赋值时，编译器不能记住实际指针所指类型，从而可以在编译期间进行static binding，提高程序执行效率？应该说有时候编译器确实可以确定指针的实际类型，但并不是总是，比如给一个vector<Base*>变量，不停的push_back指向各种派生类对象的指针，此时就很难确定各元素所指向的实际类型，若真要确定，可以想象编译器实现的复杂度。而在运行时，dynamic binding的实现就很为简单统一。
####3.5 Object member efficiency
书中测试了对于local float类型的变量读写，local float数组元素的读写，struct中float类型的数据成员的读写，OB对象中float数据成员通过set/get函数的访问。两个编译器比较结果，不开优化有差异，开了优化所有编译器的所有情况效率一样。  
随后测试了虚拟继承下的数据成员访问，编译器未能识别出通过一个非多态对象来存取元素（虚拟基类的数据成员在这种情况下的offset可以在编译期决议完成，因此可以直接存取），依旧采用运行时的间接存取，压制了优化能力，导致了效率问题。
####3.6 Pointer to Data Members
>取一个nonstatic data member的地址，将会得到它在class中的offset（例如，&Point3d::z）；取一个绑定于真正class object身上的data member地址，将会得到该member在内存中的真正地址（例如 &origin.z，注意该值的类型是float* ，而不是float Point3d::* ，这算是实例化了吧）（P132）

根据上面中引用的定义，那么class中第一个data member（若有vptr时，vptr不放在最前面的话）的offset是0，那么若与这个data member同类型的指针也赋值0的话，二者将无法区分（可以看P131下半页的例子）。为了解决这个问题，编译器将每一个member offset都加上了1。因此在使用指向member的指针时需要注意该问题。（P132上半页例子）【注意，P174写到，offset这种非完整的值，需要绑定于某个class object的地址上，才能够被存取，即offset需要有基准地址】（<font color='red'>在VS和GCC上还待验证</font>）另外，在多重继承时，指向基类的数据成员指针，与派生类中该基类对应的数据成员的指针类型是相同的。若该用后者的地方若使用了前者，因为是多重继承，基类中的数据成员，在基类的offset与在派生类中的offset不同，若派生类通过该offset来获取具体的数据成员的话，可能会导致非预期结果。（P132下半页例子）

文中最后对指向Members的指针的效率问题进行了测试和讨论。主要用四种方式使用了指向member的指针，方式一：通过指向实例化对象的成员的指针，此时的类型已经不是指向member的指针类型，而是member类型的指针（比如，不是float Point3d::*，而是float *）；方式二：通过声明指向member之指针（如float Point3d::* bx），再通过实例（如pA.*bx）来引用该成员；方式三：非虚拟继承之后，来使用方式二来操纵数据成员；方式四：虚拟继承之后，使用方式二来操纵数据成员。虚拟继承因为增加了间接性（虚基类数据成员），导致优化能力降低，因此效率最低。

###第4章 The Sematics of Function
成员函数分为三种类型：static, nonstatic和virtual。其中，static不能为virtual，也不能用const修饰。
####4.1 member的各种调用方式
* Nonstatic Member Function  
	C++设计准则之一就是：nonstatic member function至少必须和一般的nonmember function有相同的效率。  
	nonstatic member function会被内化为nonmember function，也就是编译器会在前者的形参中，插入this指针，函数体内对nonstatic data members的存取操作，经由this指针来完成，而内化成的nonmember function的名字会进行“mangling”处理，编译器保证该名字唯一。
* Name Mangling  
	编译器对data members和member functions（包括static member function，其实对于nonmember function也会进行）都会进行Name Mangling处理。各编译器采用不同的Name Mangling，目前还没有统一标准出现。关于书中P146说道，Name Mangling用到了函数名称、参数数目和参数类型的信息（三样合起来称作signature），而没有用到返回类型，但若只有返回类型不一样，而signature一样的话，这不满足函数重载条件吧，也就是不会出现这种情况。<font color='red'>此处求证伪</font>  
	对于非静态的成员函数，其Name Mangling之后，其地址类型是指向成员函数的指针类型。对于静态的成员函数，由于没有this指针，其经Name Mangling之后，地址类型是一个"nonmember 函数指针"，因此，其与nonmember函数差不多，这带来的好处是可以用作callback函数（P152）。
* Virtual Member Functions  
 	对于class object（即不是指针），其调用虚函数时，编译器应该像对待一般nonstatic member function一样，在编译期完成决议。
* Static Member Function  
	在未引入static member function之前，若要不通过建立class object来使用其nonstatic member function，有一种方法如下：

		((Point3d*)0)->object_count();
	上述方法在VS2010测试通过，包括具有vptr的class。编译器对上述代码的转换为：

		object_count( (Point3d*) 0 );
	static member function的主要特性是没有this指针，其不能直接存取nonstatic data members，不能够被声明为const, volatile或virtual，不需要经由class object才能被调用。
		
		foo().object_count();
	上述代码中，若object_count()为static member function，前面的foo()依旧要被evaluated。  
####4.2 Virtual Member Functions
本节的论述与我之前关于dynamic binding的一些思考相吻合（本文档前面）。

	ptr->z()
编译器对上述代码做如何处理，以便找到并调用z()的适当class object，是本节要讨论的问题。  
方法一：  
（1）主要思路就是将必要信息加在ptr身上，这些必要信息包括两点，① 对象的地址；② 对象类型的某种编码，或是某个结构（内含某些信息，用以正确决议出z()函数实例）的地址。  
（2）该方法的缺点：① 明显增加了空间，即使程序并不适用多态（因为所有类型的ptr都要加这些信息）；② 打断了与C程序的链接兼容性。  

那么方法一（1）②中的信息放在哪里好呢？该信息也就是P155提炼出的两点：ptr所指对象的真实类型（以便正确的选择z()实体），z()实体的位置（以便调用）。  

* 关于消极多态（passive polymorphism）和积极多态(active polymorphism)  
	P154叙述了这两个术语，在bing.com搜索了一下，并没有找到这方面的讨论，所以只好自己理解一番，记录于此。  
	对于消极多态，我想作者的意思应该是基类指针指向派生类对象的地址，之后在其它地方可以通过强制转换为派生类指针进行使用（故编译期可决议），此时基类指针有点类似C语言中void*。有virtual base class时候是例外（编译期无法决议），难道是与P122中shared object会改变位置同样的原因？<font color='red'>待核实</font>。  
	对于积极多态，也就是通过基类指针使用虚函数，另外，还有dynamic_cast的运用。无需多言。

要识别一个class是否支持多态，唯一适当的方法就是看其是否有任何virtual function，若有，则该class需要这份执行期信息。

实现上，对每一个多态class object，编译器会对其增加两个data members：一个是字符串或数字，用于表示class类型（type_info信息）；一个指针，指向虚函数表。  
virtual functions可以在编译期间获知，且这一组地址是固定不变的，执行期不能新增或替换，也就是表的大小和内容都不会改变，其构建和存取皆可以由编译器完全掌握，不需要执行期的任何介入。

虚函数表中的内容：  
（1）被继承的class覆写了的那些基类虚函数的覆写版本；  
（2）未被覆写的基类虚函数；  
（3）纯虚函数，用于占位，也可以用于执行期异常的处理函数（<font color='red'>异常处理这种情况待确认</font>）。    
每一个虚函数都被指派一个固定的索引值。当然，派生类可以增加新的虚函数，新增的虚函数信息也会放入虚函数表中。  

	ptr->z()
编译器将函数指针转换为虚函数表中的虚函数指针（解引用），处理过程等价于将语句改写为如下形式：

	(*ptr->vptr[4])(ptr)

上述语句中，slot 4中安放的具体函数地址，则需要到执行期才能确定。

* 多重继承下的Virtual Functions  
	本小节可以与第3章的多重继承（P112）对比学习。  
	派生类支持虚函数的难度，统统落在第二个基类的subobject（书中例子的Base2 subobject）身上（也即对this指针进行调整）。注：下面笔记都是基于P160的示例代码。

		Base *pbase2 = new Derived;
	编译器需要对此进行调整，上述代码被转换为（编译期完成决议）：
		
		Derived *temp = new Derived;
		Base *pbase2 = temp ? temp + sizeof(Base1) : 0;
	若不调整，请思考如下的数据成员访问是否可行；若调整，又是否可行？

		pbase2->data_Base2;
	在删除pbase2时，也许要做调整，确保调用正确的dtor（执行期才能完成）

		delete pbase2;

	对比语句Base *pbase2 = new Derived和语句delete pbase2的调整，前者可以在编译期间决议，而后者的offset却只能在执行期决议。offset书中提到了几种方法，最初的是将给每个虚函数附加一个offset，而这样对那些不需要调整的虚函数不公平；还有一种是thunk方法，思想是需要调整this指针的虚函数，将其在虚函数表中对应的slot先进行指向thunk代码块（该代码块可以根据指针类型来调整this），而无需调整this指针的虚函数对应的slot中继续存放虚函数的地址。  

	在多重继承下，一个derived class内含n-1个额外的virtual table（n表示其上一层base classes的数目）。针对每一个虚函数表，derived class中都有对应的vptr。对于一个class拥有多个虚函数表的情况，编译器的一种实现是给每一张虚函数表赋予唯一的表名称，在执行期，通过符号链接来寻找对应的虚函数表【书中是指通过动态链接的形式来实现多重继承下的派生类虚函数，故涉及到符号的解析】。Sun公司的实现是这样，将所有虚函数表合为一张表，第二张及以后通过第一张表的地址加上offset来获取就，据称性能改善明显。

	* 三种情况使得多重继承的第二及后续基类影响对虚函数的支持  
		（1）情况1：通过第二及后续基类类型的指针调用派生类的虚函数	  
			如上述delete pbase2;所对应的情况。具体解决方案也如上述。  
		（2）情况2：派生类型的指针，调用第二个及后续基类所定义的虚函数  
			此处也需要做offset调整  
		（3）情况3：虚函数返回类型有所变化，可能是base type，也可能是publicly derived type  
			返回类型是base指针，派生类将其返回类型为派生类型指针，那么，首先要确定返回的指针类型，再进行offset调整。（书中P167的例子值得参考）  

	对于情况3，sun公司提出了一种split functions的技术，即提供两个版本的函数，一个加offset，一个不加，再根据指针类型调用不同的函数。当split function较大时，就不产生两个了，而是提供多个进入点（entry points），根据情况选择不同的进入点。

	IBM将调整放入被调用的虚函数中，若需要调整，则调整this指针，之后再执行所写代码。  
	微软公司则引入了"address points"，以取代thunk策略，<font color='red'>具体没看懂(P167)</font>

**关于虚函数表的一些思考**：书本P155说道一个class只有一个virtual table，这其实是针对单一继承来说。在P164讲到多重继承时，自然一个class有多个virtual tables。我们可以对比图4.1和图4.2，派生类的vptr自然与基类的vptr指向不同的表，留意观察表中各slot在虚函数改写前后的区别。	

* 虚拟继承下的virtual functions
	因为虚拟继承有virtual base subobject的指针，再加上vptr，使得编译器对此的支持非常复杂，书中也没有细说，作者建议，不要在virtual base class中声明nonstatic data members
####4.3 函数的性能  
	
书中对比了inline member, nonmember, nonstatic member, static member和virtual member（又分单一继承、多重继承和虚拟继承三种）这些类型的执行效率。对于nonmember, nonstatic member和static member，效率是一样的（优化前后都一致）。所有类型中，inline的效率最高，特别是优化后，这表明inline不只是能够节省一般函数调用所带来的额外负担，也提供程序优化的额外机会（优化前相对于其它版本，是节省了函数调用带来的负担，而优化后，则是获得了额外的优化机会）。  

对于虚函数，因为要涉及到执行期决议和this指针调整（thunk技术可只对需要调整的this指针进行调整），而书中测试的两个编译器不支持thunk技术，使得单一继承的虚函数也会有this调整（调整值为0）。在这种情况下，单一继承和多重继承就应该有同样成本，那为什么结果是多重继承性能稍差呢？书中分析发现用到了局部变量，多次ctor和dtor导致了性能的差异。在ctor对vptr进行设定时，古老的编译器的插入代码会对this进行非空测试【若this为空则在ctor进行new操作，现代编译器将new和ctor进行了分离，不会有这种情况】，当测试次数过多，会影响性能。  
文中最后不使用局部对象（也就不会在每轮循环中调用ctor），发现频繁的ctor确实会影响性能。
####4.4 指向Member Function的指针  
取一个nonstatic member function的地址，若为非虚函数，则其为内存中的真正地址，然而，因为其需要this形参，故也必须绑定一个class object才能使用（这其实跟普通函数无异，只要是需要形参的函数，肯定是有参数才能使用）。  
指向member function的指针声明、赋值和调用（<font color='red'>未测试</font>）：  
	
	double (Point::* pmf)();
	pmf = &Point::y
	origin.*pmf();
	(ptr->*pmf)();
P175中所述指向member selection运算符是否是指"->"和"."？<font color='red'>待确认</font>  

* 指向Virtual Member Functions之指针  
	若之前的pmf被赋值为虚函数的地址，那么在调用它时，是否也是适用虚函数机制呢？答案是肯定的。虚函数在编译期，其地址是未知的，仅知道其在virtual table中的索引，也就是说，对一个虚函数取地址，只能获得其索引。这样，对一个成员函数指针的evaluated，会因为该值有两种意义（指向nonvirtual和指向virtual）而复杂化，即编译器要识别出来到底指向那种类型。书中讲述了cfront的一种实现，即假定nonvirtual的地址至少大于127，而virtual的索引值不超过127.毕竟只能用值的范围来进行区分。  
* 多重继承之下，指向member functions的指针  
	这种情况下，指向成员函数的指针需要记录更多的信息，如指向的哪张虚函数表的哪个slot，为此，Stroustrup的实现是定义了如下的数据结构  

		struct __mptr
		{
			int delta;/* this指针的offset值 */
			int index;/* -1时表示不指向虚函数表，否则，表示虚函数表中的slot */
			union 
			{
				ptrtofunc faddr;/* 表示非虚函数地址 */
				int v_offset;/* 放置虚基类或者多重继承下的第二级后续基类的vptr位置 */
			};
		};

	编译器所做的调整：

		(ptr->*pmf)();
	调整为

		(pmf.index < 0) ? (*pmf.faddr)(ptr) : (*ptr->vptr[pmf.index](ptr))
	在上述的实现中，不论指针是否指向虚函数，都需要付出判断成本。微软的一种实现是将faddr表示为非虚函数地址或者vcall thunk的地址，因此，不论是否为虚函数，直接进行(*pmf.faddr)(ptr)调用即可。  
	虽然各编译器厂家实现不尽相同（反映在__mptr的定义和使用），但多重继承下的成员函数指针的复杂性可见一斑。  
* 成员函数指针的效率  
	涉及多重继承和虚函数的成员函数指针的使用，会有较差的性能。
####4.5 Inline Functions
> cfront有一套复杂的测试法，通常是用来计算assignments、function calls、virtual function calls等操作的次数。每个表达式（expression）种类有一个权值，而inline函数的复杂度就以这些操作的总和来决定。（P183）

编译器处理一个inline函数，有两个阶段：  
（1）分析函数定义。若函数因其复杂度和建构问题（<font color='red'>什么意思</font>），被判断不可成为inline，它会被转为一个static函数。（<font color='red'>此处的static是什么意思？文件作用域，有和用处？static member function，不可能吧?待核实</font>）  
（2）函数确实被inline了，在扩展时，会带来参数的求值操作以及临时性对象的管理。  
在cfront中，inline函数只有一个表达式(expression)，其第二或后继的调用操作不会被扩展出来（即如果一个表达式包含多个inline，则只有第一个会被inline）,看下面的例子：
	
	new_pt.x(lhs.x() + rhs.x());/* x()为inline函数 */
	new_pt.x(lhs.x() + x__5PointFV.x(&rhs));/* cfront的扩展 */
**注意**，上述为cfront的方法，书中也说其它编译器厂商对inline支持没兴趣，不知道是否也是如此扩展。<font color='red'>待验证</font>  

* inline扩展  
	对于inline函数的形式参数，若形参无需evaluate（无副作用），比如直接是变量或者常量，则会产生较好的结果。否则，需要构建临时变量（比如，参数是函数的返回值foo()）。当然，若inline函数中有局部变量（非形参），则也会导致临时变量产生。这些会导致代码的增长。**思考**，其实，将inline与宏对比，宏的替换也会产生局部变量，感觉是不可避免的，确实如很多编译器厂商所想，无需在inline上做过多工作。
###第5章 构造、解构、拷贝语意学  
对于virtual base class中的pure virtual functions，可以只声明不定义，但若纯虚函数是析构函数，则必须要进行定义，因为派生类的dtor会被编译器扩展，其中会扩展为以静态调用的形式调用上一层base class的dtor。不过一般不要把virtual dtor声明为pure。  

书中“Presence of a virtual specification”，探讨了如下问题，若定义了一个virtual函数，而派生类不会改写该函数，编译器能否将其优化为静态调用。书中举了一个例子，若当前确实没有改写，而转换成了静态调用，但若有新class加入而改写了呢？那么该函数就必须重新编译（或者产生第二个版本，多态实体），这也增加了实现复杂度。不要把所有成员函数都声明为virtual function，而想着依靠编译器的优化！  

关于const修饰，要记住这点，const对象只能调用const修饰的函数，而非const对象可以调用的函数不限制是否为const修饰。这点确实比较难以考虑周全。  

####5.1 “无继承”情况下的对象构造   
三种不同的对象产生方式：global内存配置、local内存配置和heap内存配置。  

关于Plain Old Data的定义，可以参考草案N3242的第9章Classes的开头。<font color='red'>PoD定义待更新至此处</font>  
对于PoD类型，其ctor和dtor一般不会自动产生，或者产生了也不会被调用，其处理方式与C语言一致【但若是globa类型变量，则需要调用ctor，即使是PoD，原因见下面】。  
关于global的初始化，C语言中并不会明确的进行初始化，globa放置在BSS段，C语言对global变量只能用常量设定初值（注意，常量是指在编译期便可求出值。C++中的class object，并不是常量，因为其ctor只有在程序激活时才会实施）。在C++中，全局变量放置在程序的data segment中，若明确给其一个值，则初始化为该值，否则，全局变量所配置到的内存内容为0（这些内容很多参考了P242）。

* 抽象数据类型  
	对于对象的初始化，有explicit initialization list和ctor的方法，前者需要成员都是public，因此比较少用。  
	对于inline ctor，且若在ctor中，利用显示语句对data members进行常量指定操作，则编译器可以将这些赋值优化为explicit initialization list的方式。
* 为继承做准备  
	书中所说提供copy ctor会触发named return value(NRV)，其实编译器自动合成的copy ctor也可以触发吧，也就是说会不会触发NRV可以跟程序员无关了。<font color='red'>P221又说需要提供copy ctor，才能出发NRV，这个待核实！</font>
####5.2 继承体系下的对象构造  
编译器会对ctor进行扩充，程度随继承体系而定，一般包括如下扩充：  
（1）若对象有vptr，则在ctor最开始设定其初值；  
（2）若对象有基类，则基类的构造函数先被调用。若有多个基类，则按声明时的顺序进行调用（即声明时的从左到右顺序）。若基类没有默认ctor，则需要明确在成员初始化列表中对基类进行初始化，当然，若派生类不定义ctor，则编译器在给派生类自动合成时会调用基类正确的ctor（即使基类没有默认的ctor，这点可与（4）和（5）中的data member初始化进行比较学习）。  
（3）所有的virtual base class ctors必须被调用，从左到右，从继承链的最深到最浅。  
（4）member initialization list中的data members的初始化操作会被放进ctor函数本身。**注释：**在成员初始化列表中，按指定的构造函数来生成具体的数据成员；若是直接数据成员的赋值写进ctor函数体，则是先调用默认ctor（若数据成员没有默认ctor，则无法采用函数体内赋值这种方式初始化。这种情况下，只能用初始化列表进行，当然，你若对包含该数据成员的class，不给其定义ctor也可以，这样编译器在合成ctor时会自动调用该data member的ctor进行初始化。），之后再赋值。   
（5）若data member没有在初始化列表中，则在ctor里面会插入其默认构造函数（若数据成员没有默认ctor，是无法这样的。）。

* 虚拟继承  
	以P211中的图示继承关系为例，语句  
		
		Vertex3d cv;
	由于Vertex3d继承自Point3d和Vertex，而后两者都有虚拟基类Point，若后两者都在自己的构造函数扩充中，调用Point的构造函数，那么相当于Point的经历了两次构造，因此，编译器必须对此进行处理。目前的解决方案是对虚基类subobject的初始化扩充，放在对most derived对象的ctor中。现在更有效的编译器是将ctor做两个版本，一份是肯定要调用虚基类的ctor，另一份则不会调用，根据代码情况选择其中一个进行链接。  
	**实践测试：**VS2010环境下，Point3d和Vertex，若在其构造函数的初始化列表中明确调用了虚基类的构造函数（两者可以调用虚基类的不同的构造函数，虚基类必须要有默认的构造函数，不然VS2010会报错），语句Vertex3d cv; 也只对Point的默认构造函数调用一次。

* vptr初始化语意学  
	vptr在ctor中的初始化代码，是在base class ctor调用操作之后，程序员代码或者member initialization list操作之前。为什么要放置在这个位置？以单一继承为例（可以参考P157的图4.1），若base class ctor之前初始化vptr，那么在调用base class ctor时vptr值会被覆盖，导致派生类中不正确的vptr。若在派生类ctor函数体最后才初始化vptr，这种情况下，无法解决“在class中限制一组virtual function名单”的问题，因为如前文举例分析，要求虚拟机制本身必须知道对某个虚函数的调用是否源于ctor中，而虚函数需要通过vptr来获取，而且派生类可能会添加新的虚函数。  
	在ctor中调用虚函数，会被静态决议为当前class所定义的虚函数，若虚函数函数体内又调用一个虚函数，这个也是要静态决议为当前class所定义的虚函数。  
	<font color='red'>书中P218没看出头绪，这个例子与前述有何不同，不就是初始化列表调用了虚函数么？而且也不影响前述的解决方案的适用性啊。</font>
####5.3 对象赋值语意学  
本节是讲copy assignment operator的，感觉书中标题取错了，故笔记中将复制改为了赋值。  
一个class对于默认的copy assignment operator，在以下情况不会表现出bitwise copy语意：  
（1）class中有带copy assignment operator的data member   
（2）class的base class有copy assignment operator  
（3）class声明了virtual functions  
（4）class继承自virtual base class（不论该base class有无copy assignment operator）
copy assignment operator是不支持成员初始化列表的。  
在派生类的operator=中，会调用基类的operator=（也就是编译器进行了扩充）。对于虚基类，也会出现可能被调用两次的情况（参考ctor的扩充），如何避免这种情况呢？也就是书中所说的要压制虚基类的operator=调用。书中讨论了两种途径（貌似也没解决问题）：  
（1）因为可以获得到operator=的函数地址，因此不能像ctor那样，给ctor加上额外的形参，来保证虚基类构造函数的单次调用。试想，若将operator=附加了额外参数，那些通过函数指针进行调用的情况如何处理，看看P223中关于成员函数指针的typedef就清楚了，ctor的方法行不通。<font color='red'>书中说6.2有解决方案，待添加，不过也说那种解决方案不行</font>  
（2）编译器通过产生两个版本的operator=来分别应对中间的base class中是否需要调用虚基类的operator=，但若是程序员自己定义了operator=，则就无法分化了（因为你只能调用该版本）。不过C++标准中也没有说不能调用多次，而且书中说很多编译器就是调用了多次。**实践测试：**在VS2010中，只调用了当前class的operator=，其基类的都没有调用（也就是没有扩充），更不用说虚基类了。

作者建议不要在虚基类中声明数据成员。
####5.4 对象的效率  
本节标题翻译也有误，笔记已改。
测试了在PoD、ADT、单一继承、多重继承和虚拟继承下的对象构造和拷贝成本。
这个具体比较参考书本，此处略去。
####5.5 析构语意学  
若class没有定义dtor，则只有class内带的member object（或者class自己的基类）有dtor的情况下，编译器才会自动合成出一个来，否则，被视为不需要。
dtor的扩展按如下顺序进行：  
（1）dtor函数本身被执行；  
（2）class中具有dtor的data members，会以其声明的顺序反向调用；  
（3）若class带有vptr，则重新设定vptr，使其指向基类的虚函数表；  
（4）继承链上的非虚基类，若有dtor，按声明顺序反向调用；  
（5）任何虚基类，且当前class是most-derived（最尾端），则会调用虚基类的dtor，多个虚基类的话，会以构造顺序的逆序进行。
  
虚基类的dtor可能调用多次的问题：   
类似ctor的解决方案，维护两份dtor，一份总是调用虚基类的dtor和设定vptr(s)，而另一份一般不调用虚基类的dtor，也不设定vptr(s)，除非在dtor函数体中调用了。<font color='red'>第二份没看懂，dtor中调用虚函数就需要设置吗？虚函数的调用静态扩展不就搞定了吗？</font>  
关于P234页最后说dtor中vptr的代码扩展是在程序员代码之前，用vs2010测试不是的，因为类似ctor中的静态调用，dtor对虚函数也是静态调用的，试想，若vptr已被改写，还能正确的静态调用吗？其实侯捷老师改正了前面的扩展顺序，但没改正这里，按照那种扩展顺序来看的话，vptr扩展代码就不会是在程序员代码之前了。 

###第6章 执行期的语意学  
开篇的例子，就是说明，不太容易从程序代码看出表达式的复杂度。  
####6.1 对象的构造与析构  
因为C++编译器会产生扩展代码，比如局部变量若有dtor，而程序在多处有return，这可能导致每一处return前面都要扩展该局部变量的dtor调用。  
> 一般而言，要把object尽可能放置在使用它的那个程序区段附近，这样做可以节省不必要的对象产生操作和摧毁操作。许多Pascal或C程序员使用C++时，仍然习惯把所有object放在函数或某个区段起始处。（这种习惯是不好的）P241  

* 全局对象  
	C++保证，对于全局变量，会在main()函数调用其之前，将其构造出来；在main()函数结束之前，会将其析构。   
	书中提到一个“静态初始化”的概念，可对比“静态调用”进行理解，也就是进行代码扩展以初始化全局变量，最后程序退出前扩展代码以释放全局资源。  
	书中关于cfront对于不同平台支持的实现方式，没看懂，其实这个也不需要懂。另外，关于静态初始化的objects的缺点，不能放置于try不懂（放置于try了还是global吗？）<font color='red'>这里有疑问</font>
* 局部静态对象  
	只有函数在调用时，local static objects才会被构造出来。
* 对象数组  
	
		Point knots[10];
	通常的实现定义一个创建数组的函数,vec_new()，对于有virtual base class的class，则定义另一个版本的函数vec\_vnew，函数的类型通常如下：

		void* vec_new(void *array, /*若是new出来的数组，此处为0；否则，为数组地址，如上面的数组名称knots，*/
					size_t elem_size,/* 每个object的大小 */
					int elem_count,/* 数组元素个数 */
					void (*constructor)(void *),/* 默认构造函数 */
					void (*destructor)(void *, char) /*若new的过程中有异常，会调用dtor*/
					);
	对于数组空间资源的回收，也会通过使用扩展的函数vec_delete（对应的一个版本vec_vdelete），类型如下：  
		
		void* vec_delete(void *array,
						size_t elem_size,
						int elem_count,
						void (*destructor)(void*, char));
	那么，对于那些有初始化值的数组如何处理，比如，前面的Point knots[10]，前三个元素有初始化值，而后面没有。编译器的处理方式是，对前三个元素调用copy ctor，而后面7个则与前述处理方式相同。
* default ctor 和数组  
	考虑当default ctor是具有默认参数形式的object的数组时，如何将参数传到前面的vec_new()中呢？因为vec_new()中可没法传这些参数。书中所述的实现，是通过编译器在创建一个无参数的默认ctor（相当于违背了C++中，默认ctor只有一个的规定），在这个无参数的默认ctor中，再调用程序员定义的那个默认ctor。加了一层包装。
####6.2 new 和 delete运算符  
new运算符由两个步骤组成：配置内存和设立初值（调用构造函数）。若有exception handling，则设立初值会在try区块进行。对于delete，也是分为两步操作，第一步先析构，第二步则释放空间。对于分配空间和释放空间的实现，一般的library会调用malloc和free。<font color='red'>对于new_handler，这块还需要学习。</font>

* 针对数组的new语意  
	 我们知道，删除数组的语句是delete [] a;那么，如何来记录数组的元素数量呢？书中提到了两种方法，其中详述了有缺陷的一种。  
	 方法一： 在vec_new()所传回来的内存区块，配置一个额外word，用于存储元素个数。  
	 该word值可以通过array\_key（即指向分配的内存的指针）来得到，调用函数

		int __remove_old_array(PV array_key)/* 不知道该函数如何实现 */
	 在获取数组大小后，循环delete数组中的元素。这种方法是有缺陷的，从前面的vec_delete()中的dtor指针形参，其dtor类型是与数组指针类型一样的，也就是说若是派生类对象的数组，但由基类指针去删除，那么每次dtor后，删除的大小是基类的大小。**注意：**此书可能时间久远，所述与当前主流编译器实现<font color='blue'>结果不符</font>。用VS2010测试，通过基类指针删除派生类，只要dtor声明为virtual，是没有问题的，包括通过虚基类的指针来删除派生类（按P46的图示继承关系进行测试，但在dtor中调用虚函数会出现问题，当时在四个类中定义了printClassName()的虚函数，并在各自的dtor中进行了调用，此时会出现问题，<font color='red'>原因待查</font>）。

		Point *ptr = new Point3d[10]
	方法二：维护一个联合数组，放置指针及大小，也把dtor的地址维护于此数组中。
* Placement Operator new的语意  
	<font color='red'>待添加</font>
####6.3 临时性对象  
	T c = a + b; /* 编译器一般不会对此产生临时性对象 */
	
	/* 下面赋值语句 */
	c = a + b;
	/* 转换为 */
	T temp;
	temp.operator+(a, b);
	c.operator=(temp);
	temp.T::~T();
	/* 书中还探讨了另一种转换可能，但此种方式与上一种不一定等价，因为dtor和ctor是用户定义的 */
	c.T::~T();
	c.T::T(a+b);
	
	/* 没有出现目标对象 */
	a + b;
	printf("%s\n", a + b);/* 临时对象的声明周期 */
对于上述代码块最后一种情况，临时对象的声明周期在C++标准中没有指定（现在的标准不知道是否修改，<font color='red'>待查</font>）,因此，可能printf在使用该临时性变量时，已经失效，因而不安全。

>临时性对象的被摧毁，应该是对完整表达式（full-expression）求值过程中的最后一个步骤，而该完成表达式造成临时对象的产生。（P271）

考虑一种短路求值情况：

	if( xx.foo() || yy.foo())
若第二个算子被短路，还需要产生临时对象吗？书中描述了一种方法，在每一个子算子（相当于每一个条件）的求值过程中，产生临时对象，求完之后就将临时对象dtor，这样就不需要在销毁临时对象时做判断了（比如，可能需要判断有些临时对象是否产生出来）。但这种方式与标准不符，标准要求临时性对象的销毁需要在完整表达式求值完成之后（最后一步）。这样，第二个算子产生出的临时对象是否被销毁就需要一些条件测试安插进来（因为需要判断是否产生了临时对象）。

考虑如下情况：

	bool verbose;
	String progNameVer = !verbose ? 0 : progName + progVersion;
	const char* progNameVer = progName + progVersion;/* 此时可不算object初始化哦，不要这样写  */
若progNameVer是通过调用copy ctor初始化，那么产生的临时对象就是参数，若临时对象在copy ctor之前析构，会怎么样？因此，C++标准规定，临时对象被用作object初始化时，需要等初始化操作完成之后在析构。  

临时对象被绑定于一个reference，对象将残留，直到被初始化之reference生命结束，或者直到临时对象的scope结束，看哪种情况先达到而定。<font color='red'>后一种情况没看懂，若reference绑定到临时对象，必须是引用初始化的时候，那么临时对象又何来scope之说？</font>

* 临时性对象的迷思（神话、传说）  
	
		a[i] = b[i]+c[i] - b[i]*c[i];
书中说这段代码产生了5个临时对象，我觉得应该把+和*操作中，函数体内产生的临时对象算上了。临时对象的产生确实会影响效率，但编译器也能进行优化。
###第7章 站在对象模型的尖端  
####7.1 template  
 


	
##参考资料

[Opaque_pointer_url]:"https://en.wikipedia.org/wiki/D-pointer"
[name_mangling_url]:"https://en.wikipedia.org/wiki/Name_mangling"