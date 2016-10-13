把会变化的部分取出来并封装起来，以便以后可以轻易地改动或扩充此部分，而不影响不需要变化的其它部分。那么，哪些是需要改动的，哪些又是无需改动的，这个需要界定清楚。  
##策略模式  

* 问题描述    
	对于最初的设计，刚开始只有少量的派生类，随着需求的增加，派生类型越来越多，对此基类需要增加越来越多的虚函数，而原有的派生类可能就被动的增加了本不需要的虚函数（而且这些虚函数对某些类不一定有意义，比如，并非所有鸭子都会飞，若有的类型会飞，那么基类就要定义这种飞这种行为，然而，不会飞的鸭子也能调用这种行为了，这样就会出现常识问题）。此外，改动基类时影响范围太广。综上，通过派生的方式来解决新的需求问题，是行不通的。  
	通过Java的接口（interface），确实可以解决程序意义问题（调用的方法有意义，不会飞的鸭子不会有fly()函数），但会有代码重复问题（接口中没有实现代码，只能在继承中实现）。  
* 如何理解针对接口编程，而不要针对实现编程  
	针对接口编程的真正意思是针对超类型编程（C++中的基类），也就是利用多态，不同的派生类对象赋值给基类指针，再调用具体的虚函数。若不是使用基类指针，则就是“针对实现编程”。  
* 有一个（HAS-A）和是一个(IS-A)  
	有一个是组合，例如鸭子有飞行行为，现实生活中不同的鸭子飞行行为不同，那么将飞行行为单独用一个类实现，让其成为鸭子类的数据成员，这样，不同的飞行行为采用不同的派生类，而鸭子类只需要通过基类指针调用飞行方法，就可以动态的实现多种飞行行为。鸭子有一个飞行行为，这就是HAS-A.  
* 定义  
	策略模式，定义了算法簇，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。  
	理解：鸭子Duck类，若要实现多种飞行行为，可以通过在Duck中定义FlyBehavior指针类型的数据成员，通过赋值指向不同的派生类，来实现功能。  
##观察者模式    

* 定义  
	观察者模式，定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。  
* 具体实现  
	以Java为例，有两个组成部分，接口Subject和接口Observer，Subject只有一个，而Observer有多个。
		
		Subject  
		registerObserver()
		removeObserver()
		notifyObservers()
		
		Observer
		update()
* 依赖关系  
	主题（Subject）是真正拥有数据的人（因此，数据源只有一处），观察者（Observer）是主题的依赖者，在数据变化时更新。  
* 为什么是松耦合  
	首先，看如下的Java代码

		/* 在Subject中，有如下方法 */  
		public void registerObserver(Observer o);  
		/* 在Observer中，有如下方法 */  
		public class CurrentConditionsDisplay implements Observer{
			CurrentConditionsDisplay(Subject weatherData){
				weatherData.registerObserver(this);
			}
		}
	由上，我们可以看出继承自Subject和Observer**接口**的具体类中，需要分别知晓Observer和Subject接口的信息，但不需要知道继承自这些接口的具体类的信息，所以这也算一种松耦合。若要移植到C++中，我觉得必须要用一个纯虚基类，才能模拟出Java接口。  
* 扩展，观察者pull数据  
	关于pull，可以看一下观察者的update代码便知：  

		/* Observable为Java的内置实现，等同于Subject，但是是类而不是接口 */
		public void update(Observable obs, Object args){
			if (obs instanceof WeatherData){
				WeatherData data = (WeatherData)obs;
				/* 从主题中pull数据 */
				this.humidity = data.getHumidity();
			}
		}
	
	Java中Observable的实现采用类，而不是接口，若要使用，必须继承，而Java中不支持多重继承，这也限制了其使用，有时候你可能已继承了某个类，但又需要Observable，那么，就无法解决。
##单例模式  
其实就是全局变量（Providing a global point of access, a singleton is global state — it’s just encapsulated in a class），封装在类里面的全局变量。

	class FileSystem
	{
		public:
	  	//写法1，可以实现延迟初始化，Lazy initialization
		static FileSystem& instance()
		{
			static FileSystem *instance = new FileSystem();
			return *instance;
		}
		//写法2 可以利用多态性质
		FileSystem& FileSystem::instance()
		{
		  #if PLATFORM == PLAYSTATION3
		    static FileSystem *instance = new PS3FileSystem();
		  #elif PLATFORM == WII
		    static FileSystem *instance = new WiiFileSystem();
		  #endif
		
		  return *instance;
		}
		//写法3，无法使用多态（其实感觉也可以），不会延迟初始化，但某些场景下
		//比如游戏中，延迟初始化反而是不好的。
		static FileSystem& instance() { return instance_; }
	  	static FileSystem instance_;
	
	private:
	  FileSystem() {}
	};
若单例变量可以在全局访问，那么当以后需要修改该类时，因为代码散落各处（甚至有可能被当做library使用），破坏了模块化，使得修改成本很高。  

* 缺点（主要因为其就是一个全局变量）  
	（1）They make it harder to reason about code  
	（2）They encourage coupling. 顺便提及关于耦合的一些看法：By controlling access to instances, you control coupling.  
	（3）They aren’t concurrency-friendly. 这是全局变量的通病。

* 在使用单例模式前先考虑其它方式  
	（1）能否使用静态类解决  
	（2）因为是单例是全局可访问，那么能否限制其只在某个代码区块可被访问呢？如下：  

		class FileSystem
		{
		public:
		  FileSystem()
		  {
		    assert(!instantiated_);
			//控制该变量的值，使得访问并不会是全局的，只会在某一个区域
		    instantiated_ = true;
		  }
		
		  ~FileSystem() { instantiated_ = false; }
		
		private:
		  static bool instantiated_;
		};
		
		bool FileSystem::instantiated_ = false;  
	（3）通过传参的形式来使用，包括函数的形参，或者通过基类的数据成员形式（当然，后者与普通的函数传参的形式不同）  
	（4）若当前已经有了全局模块，那么，能否放在该全局模块实现调用。

##装饰者模式  
* 开放关闭原则：类应该对扩展开放，对修改关闭  
	举个例子，对于观察者模式，通过加入新的观察者，可以扩展主题（对于主题来说，无需修改代码，而观察者变多，功能扩展了）。  
	由于该原则会增加抽象层次，增加代码复杂度，因此，在设计中最有可能改变的地方，才应用该原则。  
* 具体实现  
	装饰者和被装饰者都有相同的基类，装饰者可以在被装饰者的行为之前与/或之后，加上特定的行为，以达到特定的目的。当然，装饰者也可以作为用作被装饰者，但被装饰者不一定能用作装饰者。请看下面代码：

		/* 都继承于该基类 */
		public abstract class Beverage{
			String description = "Unknow Beverage";
			public String getDescription(){
				return description;
			}
			public abstract double cost();
		}

		/* 此乃被装饰者 */
		public class Espresso extends Beverage{
			public Espresso(){
				description = "Espresso";
			}
			public double cost(){
				return 1.99;
			}
		}

		/* 具体的装饰者与基类之间又加了一个间接层，此间接层的目的
			是要求具体装饰者必须从新实现getDescription()，原本基类
			实现了该函数，但是该间接层又将其变为abstract，强制具体
			装饰类实现。在VS2010下，测试表明C++也支持该种方式 */
		public abstract class CondimentDecorator extends Beverage{
			public abstract String getDescription();
		}

		/* 具体的装饰类 */
		public class Mocha extends CondimentDecorator{
			/* 注：该成员即为被装饰者，显然，装饰者可以当做被装饰者
			（因为其基类为Beverage）；但被装饰者不一定可以当做装饰者
			因为其不一定包含Beverage成员，参考上面被装饰者的实现代码 */
			Beverage beverage;
			public Mocha(Beverage beverage){
				this.beverage = beverage;
			}
			public String getDescription(){
				return beverage.getDescription() + ", Mocha";
			}
			public double cost(){
				return 0.20 + beverage.cost();
			}
		}

		/* 外部使用 */
		Beverage beverage = new Espresso();
		beverage = new Mocha(beverage);/* 加一份Mocha */
		beverage = new Mocha(beverage);/* 再加一份Mocha */

* 总结  
	装饰者和被装饰者必须是一样的类型，也就是有共同的基类，这是相当关键的地方，我们利用继承来达到“类型匹配”，而不是利用继承获得“行为”。当我们将装饰者与组件组合时，就是在加入新的行为。所得到的新的行为，并不是继承自基类，而是由组合对象得来的。缺点：该模式会造成大量的小类，比如多种饮料和多种调料。
##工厂模式  

* 工厂方法模式和抽象工厂模式    
	**工厂方法模式**：定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类把实例化推迟到子类。实现（子类）和使用（基类）完成了解耦。通过继承实现。    
	**抽象工厂模式**：提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。通过对象的组合。例子可参考《HeadFirst 设计模式》中文版P149-P150的代码，以及P161图解。 
	
* 具体实现  
	* 工厂方法模式  
				
			public class Creator{
				/* 留待具体子类实现，当然，也可以实现默认的Productor*/
				public abstract Productor factoryMethod(int condition);
				void doSomething(int condition){
					Productor p = factoryMethod(condition);
					p.anOperation();
				}
			}
			/* 通过继承来改写工厂方法，返回不同的Productor，这就好比工厂制造产品一般，
			而基类无需知道子类制造出的具体类型 */
			public class Concrete extends Creator{
				public Productor factoryMethod(int condition){
					if(condition == 0){
						return ProductorA;/* 产品的具体实现 */
					}
					else {
						return ProductorB;
					}
				}
			}
* Dependency Inversion Principle（依赖倒置原则）  
	要依赖抽象，不要依赖具体类。低层组件依赖高层的抽象。  


##命令模式  
**定义**：将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其它对象。命令模式也支持可撤销的操作。命令模式可将“动作的请求者”与“动作的执行者”分离解耦。  
**具体实现**：三个组成部分，Client, Command和Invoker. 一般在Client类中存储了Command类型的引用成员（或者Command引用类型的列表），这样，Client就可以调用Command引用成员的execute()方法，从而执行具体命令。至于execute()中执行了什么语句，Client无需了解。那么，不同的Command对象，实现不同的execute()方法，一般是在Command对象中，定义一个Invoker类型的数据成员，而execute()中则调用该Invoker类型的某些成员方法。以上，便实现了Client和Invoker的解耦。  
**撤销**：在Command中加入undo()实现，这样在Client中每一次操作后记录当前执行的Command成员（若要支持多次撤销，可用一个Stack将每一次的Command类型成员装入），在撤销的时候，调用这些刚才保存的那些Command成员的undo()即可。  
**宏命令（Party Mode）**：若要一次执行多个命令，则可以建立这样一个Command对象，而在该Command对象中，包含多个Command对象的引用，在该Command对象的execute()中，只需要简单调用其包含的Command对象引用即可。例如，按下遥控器的一个按钮，就能够执行调暗灯光、打开音响和电视。    
**注意事项**：要注意，对于每一个请求者，都需要建立一个Command类型对象。例如，开灯和关灯，若是有两个按钮（两个请求者），则要分别创建开灯的Command对象和关灯的Command对象。  
##适配器与外观模式  
* 适配器模式的定义  
	将一个类的接口，转换成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。适配器又分为类适配器和对象适配器，前者利用了多重继承，而后者使用了组合。前者不需要重新实现整个被适配者，特别当方法较多时；而后者则可以适配被适配者的任何子类。  
* 外观模式的定义  
	提供了一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用。
 
##模板方法模式  
**定义**： 在基类的某个方法中定义一个算法的骨架，而将一些步骤（方法）延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。  
**具体实现**:两个组成部分，基类和子类，基类中定义了某算法（一个方法），而该算法包含一系列步骤（多个方法）。子类不能修改基类定义的算法，但可以修改该算法包含的某些步骤（公用的步骤已经在基类中实现，无需子类修改），这些步骤被定义为抽象方法，子类必须继承实现，这样子类就能够控制算法。  
**钩子**：钩子可以让子类实现算法中可选的部分，也可以让子类能够有机会对模板方法中即将发生的步骤做出反应。钩子一般是基类实现的具体方法，而子类可以选择覆盖或者不覆盖。  
**循环依赖**：为避免循环依赖，高层组件（例如基类）对待低层组件（例如派生类）的方式是“别调用我们，我们会调用你”。低层组件绝不可以 **直接**调用高层组件（其实，也不绝对，一般上是这样）。  
**演进**：（1）在C++ STL中，对于排序，通常可以提供谓词(Predictor)，实际上这种排序也算模板方法模式，Predictor就是我们可以修改的步骤。此外，排序的数据类型，也可以指定，也算可以修改的步骤。由此可知，模板方法不一定要求基类子类这种模式。  
##迭代器与组合模式  


##状态模式  
* 定义  
	允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类。  
##MVC模式  
[MVP与MVC的区别][mvp_mvc_difference_url]  
###MVP  
Presenter相当于View和Model的中间人（middle-man）。In MVP, all presentation logic is pushed to the presenter. View只是一个passive接口，工作都转给presenter，当然，这与具体实现有关，UI的逻辑也有可能view直接处理。一般一个View对应一个Presenter（View中有Presenter接口的数据成员），但也有一个View对应多个Presenter的情况。     
优点：Easier to unit test because interaction with the view is through an interface.  
###MVC  
主要目的是将UI和业务分离，各模块的功能：  
View: the view is responsible for rending UI elements.  
Control: the controller is responsible for responding to UI actions.  
Model: the model is responsible for business behaviors and state management.
在很多实现中，MVC这三者可以相互访问彼此接口，在有些实现中(Front Controller Pattern),Control可控制显示哪个View。  

http://martinfowler.com/eaaDev/uiArchs.html  

###Proactor Pattern  
* Conventional Concurrency Models  
	最常见的两种方式：Synchronous multi-threading and reactive programming  
	（1）Synchronous multi-threading  
		每一个线程以阻塞方式来进行连接建立、文件传输等操作。  
		优点：编码简单，各线程之间基本可互不干扰。  
		缺点：  
		① Threading policy is tightly coupled to the concurrency policy（这点没看明白，说是只能根据可用资源进行优化，而不是根据连接数进行优化）  
		② Increased synchronization complexity: 在使用服务器的共享资源时，会要考虑多线程的同步问题  
		③ Increased performance overhead：context switching, synchronization, and
		data movement among CPUs  
		④ Non-portability:不同的OS会有不同的线程实现  
			
		  
		
policy:
##参考  
[mvp_mvc_difference_url]:[http://stackoverflow.com/questions/2056/what-are-mvp-and-mvc-and-what-is-the-difference]


