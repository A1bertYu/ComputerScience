# Beginning Database Design  
Clare Churcher, 2nd, Apress  
##chapter 01 What Can Go Wrong  
###Mishandling Keywords and Categories  

* 例1-1  The Plant Database  
	对植物分类，图1-1中，每一种植物，其属性都在一张表中。其中，对于“用途(use)”属性，还占了多个列，而且当要添加新用途时，又需要增加新的列。若是通过用途来寻找植物，则增加新列后还会涉及到逻辑修改。  
	对于图1-1的设计，作者说用途分为多列比仅用一列来进行存储好（存为一列时，用','进行分隔）。  
	数据库的设计，应该要思考解决什么问题，比如，某种植物有哪些用途？（图1-1能很好的解决）对于某种特定用途，有哪些植物？（图1-1不太好解决）。  
	图1-1，最初设计为class Plant，考虑将其拆分为Plants和Uses，并考虑二者的联系。图1-2便是一种解决方案。  
* 例1-2 Research Interests  
	与例1-1类似，也是将属于多个class的数据塞进了一个class，从而形成了一张表。  
	但是，关于专业领域的分类，不像例1-1中的用途，那么容易划分，因为专业领域是个人主观所写，即使同一种专业领域，描述也可能不同，因此这方面问题需要解决。  
###Repeated Information  
重复字段，比如多个表中存储了用户的联系方式。

* 例1-3 Insect Data  
	图1-4中，域ADhc对应的farm是为1的，而在图1-4中，第269行却显示其对应的farm为2。日期那一列，也有可能存在这种问题。  
###Designing for a Single Report  

* 例1-4 Academic results  
	图1-6中关于学生各科目成绩的统计，没有考虑的包括，若学生有其它选修课（虽然当前所有学生的课程相同）；若学生某一门有补考行为（这样需要记录多个分数）。这又涉及到类设计的问题，此处也只设计了Student，实际上划分为Student and Subject会更合理。  
##chapter 02 Guided Tour of the Development Process  
try to understand the data that are going to support that task and other likely tasks.
Designing a database to reflect the type of data involved, rather than what you currently think is the main use for the data, will be more advantageous in the long term.    
Arriving at a good solution for a database project requires some abstraction of the problem so that the possibilities become clear.  
###Initial Problem Statement  
通过用例（use cases）对问题进行描述，用例也就是用户如何与系统进行交互，也可以理解为用户想要完成什么样的任务。
###Analysis and Simple Data Model  
* Classes and Objects  
	class的定义：Each class can be considered a template for storing data about a set of similar things (places, events, or people).可以对比面向对象中的类的抽象。 
* Relationships  
	在设计数据库时，对于每一个class，通常对应一张表，而class之间的关系，也会设计为一张表。  
	图2-10中的表，关于class之间的对应关系，采用了UML的表述，可以看看。  
###Further Analysis: Revisiting the Use Cases  
分析了第一章中例1-1，建立了Plant和Use两个类，并使用了UML中的表述，描述了两者关系。  
讨论了若按照Genus进行筛选，可以考虑再增加类Genus，这比作为Plant的一个属性好点，因为作为属性时取值是每个Plant对象进行取值，可能会出现比如大小写不一致等情况(统一性差，类似于Repeated Information 例1-3中的情况)，这样会影响按Genus筛选的结果。  

* 例2-2 Revised use Cases for the Plant Database  
	对例2-1的更进一步分析，建立用例。  

迭代式的对问题进行分析，产生更为清晰的用例和class。  
###Design  
一个类对应一张表，类的关系也表示为一张表（也用类表示），外键表示表之间的关系  
###Implementation  
在关系数据库中，依据图2-14建立对应的表。  
###Interfaces for Input Use Cases  
用户最终接触到的部分。
###Summary  
四个阶段值得一看，本书以抽象化（第2阶段）为主。  
##Chapter 03 Initial Requirements and Use Cases  
理清楚两个问题：（1）软件的所有目标用户需要用软件完成什么事情；（2）为了支持（1）需要存储哪些数据。  
两种视角：（1）用户角度出发，真实世界存在的问题；（2）开发人员角度出发，对问题进行抽象。  
###Real and Abstract Views of a Problem  
* Data Minding  
	要搞清楚数据的用途，会被怎么用。预测可能需要的输出（用户当前可能没有或者根本没提这种要求）是难点。  

* Task Automation  
	将用户的任务同需要被记录和被报告的数据分开。书中列出了问题提纲，以解决这种问题。  

* 例3-1 Meal Deliveries  
	按照Task Automation列出的问题提纲，进行了分析。  
	问题为送餐员送餐的问题，旅馆客户订餐后，送餐员负责送过去并收取餐费。送餐公司想对这些数据进行统计。  
	（1）用户要做些什么？  
	送餐公司客服会记录订单信息，地址、电话、餐品和价钱；  
	客服将订单分发到具体送餐员，并告知订单信息；  
	送餐员取到分发过来的餐品；  
	送餐员送到用户并将送达信息通知客服；  
	送餐员将工作情况汇总成表以提交给公司；  
	公司对送餐情况进行周总结或者月总结。  
	（2） 什么是相关数据？  
	对一个典型用例涉及的执行过程进行分析。  
	（3）系统的目标是什么？  
	不要想着全盘自动化，很多时候需要人的参与，这样效果会更好。    
	多想想具体的问题，而不要太过笼统。此外，为了解决这些问题，检查是否已经获取了足够的数据。设计时要考虑能获取准确数据的途径（比如需要用户输入时，因为某些压力或者麻烦，用户可能会输入不准确的数据），对准确数据进行分析才用意义。  
	（4）需要什么数据  
	需要对用例进行更精确描述，才能得到。  

* 例3-2 Restatement of Meal Delivery Problem  
	为了保证后续的可扩展性，对例3-1中的问题进行重新思考。  
	（1）What are the Input Use Cases?
	从任务的角度（user task level），来设计用例。用例设计既不能太宽泛，也不能太细。  
* 例3-3 Initial use cases for Meal Deliveries  
	对上面的总结，得到用例。  
* What Is the First Data Model?  
	首先得到类Order和Meal  
* What Are the Output Use Cases?  
	对数据进行分析，不同类型的用户有不同需求。
###More About Use Cases  
* Actors  
	不同类型的用户，与数据库交互的方式不同。从各种用户与系统交互的角度，要求解决哪些问题。 
* Exceptions and Extensions   
	要考虑意外情况，比如，送餐的例子，订单取消、午夜跨天送餐（时间临界点）  
* Use Cases for Maintaining Data  
	管理数据包括：增加、删除和修改。这三个方面一般是分开的。  
* Use Cases for Reporting Information  
	输出，guided by readability
###Finding Out More About the Problem  
对于用户提出来的问题，目前用户是如何应对的，从这些已有的解决方案中，进行借鉴。  
###What Have We Postponed?  
对于例3-1的问题，迭代过程中可能还有新的需求需要考虑，因此，迭代开发，不断完善新功能。书中列出了几点，价格发生变化后，以前的订单查看时如何处理价格问题？若有些餐品现在不再外卖，需要删除，那以前的订单又如何处理？每一种餐品外卖时的每一订单所订数量，也是需要考虑的。  
##Chapter 04 Learning from the Data Model  
为了解决真实世界中的问题，需要存储相关数据，而数据模型（data model）是对这些数据的一个描述，因为不可能完全或者精确的进行描述，故一般会基于某些假设和定义（这样只需关注最本质的信息）。It is a model of the relationships among the data items that are being stored about a problem.  
###Review of Data Models  
本节举了一个旅馆登记系统的例子，该系统首先考虑以组为单位进行登记，组内所有人员到店和离店日期相同，但是，当组内有人员因为某些原因要提前离开，或者独自旅行的客人需要住宿时，这种模型都不能很好的解决问题。  
后面几节主要讨论两个class之间的关系，共有如下四种情况：  
（1）Optionality: Should it be 0 or 1?  
（2）Cardinality of 1: Might it occasionally be 2?  
（3）Cardinality of 1: What about historical data?  
（4）Many–Many: Are we missing anything?  
###Optionality: Should It Be 0 or 1?  
两个对象形成的关系中，一个对象关联另一个对象实例的数量可以指定，而最小数量称为Optionality，且通常为0或者1.  
Optionalities can provide a great deal of information about the definitions of classes and the scope of the problem.  

* Student Course Example   
	考察学生和课程之间的关系，Optionality的选取对数据建模和问题分析有重要影响。例如，有些刚报到的学生还没有开始进行选课，这时候就不会有课程； 这样会使得问题分析更为全面。 
* Customer Order Example  
	客户与订单的关系，客户必须对应有订单吗？那么如何记录潜在客户信息？反之，订单必须关联用户吗？若有些情况下不知道该订单的客户信息呢？  
* Insect Example  
	若很多以前的Sample并没有详细的记录Visit，若何处理？  
###A Cardinality of 1: Might It Occasionally Be Two?  
* Insect Example  
	每一次访问（Visit）的时候，会采集多个样本（Sample）。考虑天气（Weather）情况对试验的影响，那么天气关联到Visit的时候，会出现这种情况，此次Visit时，开始天气都没有变化，但在采集最后几个Sample时，天气发生了变化，如何处理？若天气关联到Sample，那么若Visit天气不变，则此次的所有Sample都有相同Weather，这导致数据重复。  
	解决方法，重新定义Visit，如下：  
	A visit is a time spent on a farm during constant weather conditions on a single day. It is possible to have more than one visit to a farm per day.  
* Sports Club Example  
	俱乐部通过Team来管理Member  
###A Cardinality of 1: What About Historical Data?  
* Sports Club Example  
	为了保存历史记录，每个人可以对应多个组，而一个组包括多个人。  
* Departments Example  
	一个部门有一个经理，但为了保存历史记录，一个部门对应多个经理（不同时期）。  
* Insect Example  
	若一个农场的类型随着时间会变化，那么为了保存历史记录，应对应多个类型。  
###A Many–Many: Are We Missing Anything?  
对于有些n-n的关系，可以通过引入中间类来进行实现。（图4-14）  

* Sports Club Example  
	关于球员和球队之间的关系，若球员随着时间的推移效力了多个球队，那么球员和球队之间的关系就是n对n，但是，通过引入一个Contract类，里面只有球员签约球队的日期信息，而球员可以有n个合同，球队同样也有n个合同，这样就形成了球员与球队之间的Many–Many关系。  
	图4-17是对图4-16的解读，每个Contract实例只有一个球员和球队，而一个球员可以有多个Contract，一个球队也可以有多个Contract。  
	图4-18是数据库中的一种具体实现。  
* Student Course Example  
	如何登记学生的课程成绩？成绩不能放入Student，因为其与Course有关；当然，也不能放入Course，因为其与Student有关。综上，引入中间类Enrolment.  
	A student and a course can each have many enrollments, and a particular enrollment is for exactly one student and one course.  
	三者的关系同例子“Sports Club Example”中的三者关系。  
* Meal Delivery Example  
	Order和Meal之间，也有Many–Many关系，对于Meal的数量，放哪里呢？这也需要一个中间类。对于变动的价格，也可以放入中间类。
###When a Many–Many Doesn’t Need an Intermediate Class  
Problems that involve categories as part of the data often do not require an additional class.  
A Many–Many relationship that doesn’t require any additional information often occurs when we have something that belongs to a number of different categories;  
###Summary
本章总结很到位，值得细看。  
##Chapter 05 Developing a Data Model  
###Attribute, Class, or Relationship?  
* Sports Club  
	分析了是将数据存储为某个类的属性、或者成为一个新类、或者为两个类之间的关系。  
	这三种要根据具体应用来选择。本节中的最后两点总结很好，可以看看。  
###Two or More Relationships Between Classes  
* Sports Club Example  
	图5-4描述了Member和Team之间的两种关系。从图5-4中，并不能确认一个Team的Captain是否Plays For这个Team，也就是说图中的两种关系是独立的，A推不出B.   
* Small Hostel  
	当有历史数据时，原先只有一种关系的两个class之间，可能需要再添加一种关系。对于前面的住宿问题，若要找出正住在房间A的客人，根据之前的设计，需要搜寻所有登记到该房间的人（包括历史记录），同时检查该客人所关联的组的离店日期，检查离店日期是否还未到来，若是，则该客人就是当前住在房间A的客人。若要找出所有的空房呢，也很麻烦。  
	给Guest和Room增加一种关系，Guest当前是否在Room，便可容易地解决上述问题。当然，增加关系后，数据的一致性同步工作也需要增加。 
	it is best to avoid the situation where we have information stored more than once.   
###Different Routes Between Classes  
某些信息可能从两种或以上的途径得到（也就是冗余），而由于数据维护时没有保持一致性，导致各途径得到的结果并不相同。  
当增加关系以带来查询上的方便性的同时，数据的维护变得复杂了，两者间需要有所权衡。  
  
* 例5-3 Startup Incubator   
	以图形所示的data model中，若关系链有闭环，则要注意可能存在的数据冗余。  
* Routes Providing Different Information  
	图5-8所示，若Employee可以为多个Group工作（注意，图5-7限定为1个），那么通过Group来查找Employee的地址，则无法实现。因此，图5-9引入了Employee与Room的关系，在图5-9中，虽然闭环，但并不是数据冗余。  
	The important thing is to ensure that two routes do not contain what should be identical information so we do not introduce avoidable inconsistencies.   
* False Information from a Route (Fan Trap)  
	图5-10和图5-11示例了扇形陷阱，也就是左右两边的类，其关系反映在两个类的联合上，而我们只能得到这些联合的信息，比如图5-11可能只会得到Jim A, Jim B, Sue B, Jane D这些信息.  
	这种关系的特点：The feature that alerts us to a fan trap is a class with two relationships with a Many cardinality at the outside ends.反应在图5-11中，中间Division与两边的类都构成扇形形状。  
	fan trap会使一条路径在某个中间结点分散出多个（Check to ensure you are not inferring more than you should from a route;）
* Gaps in a Route Between Classes (Chasm Trap)  
	本节举了一个层级结构的例子（hierarchical），Employee, Group, Division逐步往上。要是查询Employee所属的Division，则通过Group即可，但是，当有些Employee没有Group时，则就无法查询，称该问题为chasm trap.  
	解决方案一，建立Employee与Division的直接关系，那么，此时有些用户可能有两种方式到达Division，会出现数据冗余。  
	解决方案二，增加一个特殊组，也就是所有Employee必须有一个组进行关联，这样可能要重新对数据进行建模。  
###Relationships Between Objects of the Same Class  
俱乐部中，老球员推荐新球员入会，那么这是Member之间的关系，需要建模为图5-15，而不是图5-14，因为后者会造成数据冗余。此外，图5-15中的模型，并没有反应出Member能否自我推荐，这一般在用例设计中体现。  
Animal类中，is mother的关系，如图5-16中所示，为什么Animal的Mother是0或者1呢，而不就是1呢，因为现实中，确实Animal必有Mother，但数据库中，是存储了才有，而不存储是不会有的。  
此外，图5-16中，若有些Animal是Male，则因为存储错误会使该Animal成为Mother，所以必须考虑这种情况以防出错。  
###Relationships Involving More Than Two Classes  
图5-17两侧都是n，因此会形成fan trap。从图5-17中，对于某场比赛，某个球员是否会参与，我们无从得知（cannot deduce），我们只能得出某球员在某个Team，而对于某场Match，即使包含该Team，但也无法推知该球员是否参与该Match。当然，一般认为，若球员不在Team A中，那么球员是不会参与不包含Team A的Match的。  
为了解决该问题，Member和Match之间添加了Plays in关系，如图5-18，这样会造成数据不一致。John是Team A的，那么理应不会参加Team B和Team C的Match（图5-18即这种意思），然而，实际中却可能会出现这种情况，比如John作为外援（图5-18其实是不允许这种情况的，若发生则就造成了数据不一致）。此外，John即使可以为Team B或者C当外援，那么到底是B还是C呢？图5-18的模型也无法表示。  
对于每一场Match，需要知道哪些Team的哪些Member参加，图5-18的模型难以解决该问题。因为，我们需要同时知道Member, Team和Match的信息，这导出了图5-19的模型。  
图5-19中的连接类，一般只记录三者有效的连接信息。当然，也可以自定义属性，但属性应该是包含了三者关系的数据。  
从图5-19是可以推出图5-18的，也就是Member, Team和Match两两之间的关系，若在图5-19之间直接建模了三者两两之间的关系（即不是通过中间的连接类），这种闭环会造成数据不一致。当然，书中也说了，有些关系还是需要在两两之间添加关系来进行表述。例如图5-20便是对图5-19的补充。  
图5-20下面的一段话，提出了几个问题，反映了某两个类之间的关系，而与第三个类无关（independent），这时也需要添加两个类之间的关系。  
##Chapter 06 Generalization and Specialization  
###Classes or Objects with Much in Common  
本节举了一个职员类Employee的例子。因为各种岗位的属性不同，比如技术岗位有技能证书的失效日期，而其它岗位没有，若所有岗位都采用同样字段的话，则Employee并不会用到所有字段（例如，非技术类岗位就不是用到技能证书的失效日期字段），这样就有不完整数据。当然，我们可以通过条件判断的方式，来进行字段的访问，比如，只有技术岗位才访问技能证书的失效日期字段，若增加新的岗位信息后，则又要修改判断条件	。  
图6-1显然不是一个可靠的设计。    
###Specialization  
类似于C++中基类派生的设计方式，将公共部分抽象出来（as general as possible，因为后续若改动base会影响早前的派生类，影响范围很大）。  
###Generalization  
图6-6的模型，在分配车位时，车位可能会被同时分配给老师和学生；图6-7是解决方案，在这种模式下，车位是与Person交互，而Person要么是学生要么是老师（或的关系），因此，此时车位只会分配给一个人。  
###Inheritance in Summary  
从本章前面两节（Specialization和Generalization）来说，前者解决某种问题是在派生类中进行（各派生类可以有不同属性，故称Specialization），后者则是在基类中解决(故称为Generalization)。  
在是否使用继承结构时，可以检验这两个问题：  
Is an object of SubClassA a type of SuperClass? (always)  
Is an object of SuperClass a type of SubClassA? (sometimes)  
若是括号里的回答，则适合采用继承结构。  
###When Inheritance Is Not a Good Idea  
* Confusing Objects with Subclasses  
	如何来区分class与object，这个书中对此也提炼出了一个问题：  
	Am I likely to have several Corgis（所要解决的问题中的某个对象名称） and am I interested in them as a group?  
	上述问题，若肯定回答，则是class；否则是object. 其实，对此，采用面向对象的设计概念，还是很容易解决的。  
* Confusing an Association with a Subclass  
	对于上小节提到的问题，首先给出了图6-9中继承方式的解决方案，但若有新的品种，那岂不又要添加一个subclass？此方案明显牛刀小用。因为该问题中只有品种的值不同，故可以考虑单独建立一个品种类Breed，如图6-10，这样扩充新品种就非常简单了。  

We only need to consider specialized classes if we have different attributes or relationships(not just different values for an attribute).  
###When Is Inheritance Worth Considering?  
The decision of whether to use inheritance or not depends on how important the completeness and accuracy of the data are to the objective of the project.  
###Should the Superclass Have Objects?
类似于C++中的抽象基类，这种基类不要实例化。  
It is generally advisable that any class which has subclasses should be an abstract class, meaning that it cannot have objects.  
###Objects That Belong to More Than One Subclass  
有时候会用到多重继承，但是，当添加多重继承中的基类时（比如二重继承变为三重继承），其子类可能也要相应添加，这有时候会使得模型非常复杂，难以处理。这时，就需要改变思路，不要多重继承，而将多重继承的部分（即每个基类各自涉及到的部分）抽出来，另行建模。  
例如，图6-16中，老师和学生，老师有时候也需要学习，而学生则可以做老师（如家教），那么图6-16中是使用了多重继承的解决方案。试想，若，
再增加其他种类的职业，而其也可能与老师、学生交叉，如何解决？
换一种思路，得到图6-17的解决方案。不过此种方式，缺少一些限制条件，比如一个人会有多份student的contract（不过也可能会发生此种情况，这就要具体分析了）。整体来说，图6-17的解决方案远远好于图6-16.  
###Composites and Aggregates  
参考了设计模式中的组合模式，多用组合，少用继承。数据库设计虽然主要是处理数据，而设计模式则主要是关注行为（behavior），但前者也可以像后者借鉴很多思想。 
##Chapter 07 From Data Model to Relational Database Design  
###Representing the Model  
一个好的模型的判断标准：即尽量用关系数据库中的标准技术，而少编程或者进行复杂接口的设计。
A good model, implemented using the standard techniques, allows us to capture many of the constraints implied by the relationships between classes without recourse to programming or complex interface design.   
表7-1对模型到实际表的映射，做了一个很好的总结。  

* Representing Classes and Attributes  
	Class对应一张表，而Attribute对应一个Field，一行数据代表一个object  
* Creating a Table  
	通过语句或者图形化前端创建。  
* Choosing Data Types  
	数据类型大致可分为四类：字符、整型、分数和日期。  
	为什么要划分出这些数据类型，字符串不是什么都可以记载吗？  
	原因有三：  
	（1）对于取值的限制。例如，整型的数据就无法输入字符，但是，若用字符存储整型，则有误输入可能。  
	（2）排序。字符和整型排序方法不同。  
	（3）计算。只有存储的值合乎要求，才能进行计算。  
* Domains and Constraints  
	A domain is a set of values allowed for an attribute or field.  
	The difference between a constraint and a domain is that the former has to be specified in every table whereas the latter only needs to be declared once in the database.  
* Checking Character Fields  
	A good rule of thumb is that any data that you are likely to want to search for, sort by, or extract in some way should be in a field all by itself.  
	对有准确度要求的field，比如，用户的城市不能随意拼写（如Paris, paris），则可以专门建立一个City类，用于列出合乎要求的名称。
###Primary Key   
	
* Determining a Primary Key  
	A key is a field, or combination of fields, that is guaranteed to have a unique value for every record in the table.  
* Concatenated Keys  
	怎样去确定数据库中表的key，有无满足unique的field，若没有，那么多个fields是否能满足unique，进行这些思考，可以对待解决的问题有更为准确的认识。  
	当需要多个field才能构成key时，考虑再增加一个自增的field（假设该field名称为auto\_increament），而且该field可以保证unique，那么添加该field（auto\_increament）有何利弊？优点，更短了使用方便。缺点，需要额外的限制来确保数据不重复。就拿书中例子来说，(student, course, semester, year)可以构成一个key，若再增加auto_increament，那么(student, course, semester, year)就有可能会重复。  
###Representing Relationships  
前述一个table代表一个class，一个field代表一个数据成员，具有unique性质的多个filed的联合（或者一个filed独自）便为table的primary key. 各class的关系便可以用primary key来表述。

* Foreign Keys  
	关于外键的定义：  
	A foreign key is a field(s) (in this case captain) that refers to the primary key field(s) in some other table (in this case it contains a value of the key field member_ID from the table Member). In this way, we establish the relationships between objects of different classes.  
	主键和外键的关系：  
	Formally, a foreign key and the primary key of the table it references must have the same domain.  
	在实际的数据库产品中，一般要求主键和外键必须是相同的（或者兼容的，对于兼容性的要求，需要查看具体数据库产品）数据类型。  
* Referential Integrity  
	Arm-in-arm with the idea of a foreign key is the concept of referential integrity.  
	引用完整性，外键中的值，必须是另一个表的主键值中的某一个（在有些情况下可以是null）。因此，主键的值被别的表当做外键值时，该主键值对应的记录是不能够被删除的。  
* Representing 1–Many Relationships  
	For a 1–Many relationship, the key field from the table representing the class at the 1 end is added as a foreign key in the table representing the class at the Many end.  
* Representing Many–Many Relationships  
	需要引入一个中间类（a new intermediate class），这样将一个many-many关系转换为两个1–Many，从而可以采用前述的外键描述。  
	This combining of foreign keys to form a primary key is often the case in the situation where an intermediate table has been introduced in a Many–Many relationship.  
* Representing 1–1 Relationships  
	具有这种关系的两张表，任一张表都可以新建一个field，作为外键，来存储另一张表的主键值。当然，要根据具体情况进行合理选择。不过，这种方式缺少约束性，对于书中的例子，一个Team本来应该只有一个Captain，但在Member表中，可以输入这样的数据，即多个Member的"is\_captain_of"域可以相同，这样造成一个Team会有多个Captain.  
	For a 1–1 relationship, put the foreign key in the table where it is most likely to have a value or where the attribute is most important.  
* Representing Inheritance  
	如何来实现继承关系？对每一个类（基类、子类）都建立一张表，而基类与子类之间是1–1 relationship.  
	One way to capture the main aspects of inheritance in a relational database is to set up classes for each parent class and subclass and include a 1–1 relationship between each subclass and its parent.
	
##Chapter 08 Normalization  
Normalization is a formal way of checking the fields to ensure they are in the right table or to see if perhaps we might need restructured or additional tables to help keep our data accurate.  

###Update Anomalies  
更新时可能会出现数据不一致，比如，项目名称的叫法，不同的叫法可能对应同一个项目，如此，便出现了名称的不一致。  
  
* Insertion Problems  
	当field用作primary key时，filed的值是不能为空的。图8-1中的Assignment表，若将emp和project_num作为primary key，那么，在项目最初并未分配给职员时，此时录入（insertion）便会出现问题。  
* Deletion Problems  
	删除时可能导致未考虑到的信息丢失。  
* Dealing With Update Anomalies  
	通过增加新的表，并改进data model和use cases  
###Functional Dependencies   
	
* Definition of a Functional Dependency    
	所谓Functional Dependency：If I know the value for this attribute(s), I can uniquely tell you the value of some other attribute(s).  
* Functional Dependencies and Primary Keys  
	The key fields functionally determine all the other fields in the table. （注意，此处用词为key，而不是primary key）  
	A primary key has no subset of the fields that is also a key.（也就是说primary key所包含的filed的子集中，不可能再存在key）  
	A primary key is a (minimal) set of field(s) that functionally determines all the other fields in the table.
	上述中，key（当然，primary key是一种特殊的key） functionally determine其它的field。  
	然而，在用外键表述关系时（比如1–Many relationship），我们用的是primary key，若用非primary key作为外键，则会造成数据冗余，从而可能导致不一致问题（因为key包含primary key之外，还有其它冗余的field）。  
###Normal Forms  
* First Normal Form  
	A table is not in first normal form if it is keeping multiple values for a piece of information.   
	If a table is not in first normal form, remove the multivalued information from the table. Create a new table with that information and the primary key of the original table.  
* Second Normal Form  
	A table is in second normal form if it is in first normal form AND we need ALL the fields in the key to determine the values of the non–key fields.    
	If a table is not in second normal form, remove those non–key fields that are not dependent on the whole of the primary key. Create another table with these fields and the part of the primary key on which they do depend.  
	当有些non-key fields可以只通过primary key中的部分field就可以唯一确定时，那么就不满足第二范式，就需要调整。  
* Third Normal Form  
	A table is in third normal form if it is in second normal form AND no non–key fields depend on a field(s) that is not the primary key.  
	If a table is not in third normal form, remove the non–key fields that are dependent on a field(s) that is not the primary key. Create another table with this field(s) and the field on which it does depend.  
* Boyce–Codd Normal Form  
	A table is in Boyce–Codd normal form if every determinant could be a primary key.  
	对于determinant的定义：Say I know that the value of a particular field (e.g., proj\_num) determines the value of
	another field (e.g., proj\_name). We say that proj_num is a determinant (it determines the value of something else).  
	书中说，对于绝大部分表格，该范式等效于第三范式，但在有多个primary key的表中（more than one possible combination of fields that could be used as a primary key），该范式强条件于（slightly stronger statement）第三范式。我们知道，从第一范式到第二范式再到第三范式，所要求的条件是越来越强的。  
	
* Data Models or Functional Dependencies?  
	对于数据库表的具体实现方式，我们可以采用data models和以Functional Dependencies为基础的normal forms两种方法。  
	两种方法的思考视角不同。解决问题时，最开始采用data models，而最后则采用normal forms。The data model is great for the big picture, and normalization is great for the finer details. Use both these tools to ensure that you get the best structure for your database.   
	本章总结最后有数据库的设计步骤，步骤中包含了这两种方法。
###Additional Considerations  
前面介绍的四种范式（第一、第二、第三和Boyce–Codd），都是基于functional dependencies来进行定义的。那么，对于图8-13中的表，其primary key由所有的fields组成，也就是说不存在functional dependencies.  
本节主要讲了非前述四种范式所及的情况，Fourth and fifth normal forms deal with tables for which there are multi–valued dependencies. 这就要具体分析了，即使数据冗余，在有些情况下也是必要的。
##Chapter 09 More on Keys and Constraints
###Choosing a Primary Key  
* More About ID Numbers  
	We use the primary key in order to make connections to rows in different tables. 通常，用作primary key的field的值要求不要频繁改动，最好是常量，比如身份证号码。  
* Candidate Keys  
	The key fields functionally determine all the other fields in a table.  
	A candidate key is a key where no subset of the fields is also a key.  
* An ID Number or a Concatenated Key?  
	若有多个1–Many ownership relationships连接成一串时，且都以1-end的primary key作为foreign key，这样尾部的表，其foreign key所包含的field就非常的多。那么，就可以考虑在某些表（其前面的表或自身）中，再设计一个可以作为primary key的ID field，这样，就可以减少foreign key所包含的filed数量。  
###Unique Constraints  
当一个表中可以有多个key来作为primary key时，选用了某个，则会使其它的候选key在修改数据库时失去唯一性校验。书中举了一个例子，两个候选key：visitID 和 (date, farm, paddock). 那么，若选用了前者，则后者在输入数据时，可能会造成重复，但依旧能够输入到数据库中。  
解决方法：在创建表时使用UNIQUE (date, farm, paddock)  
唯一性约束的其它作用，Unique constraints are also a way to enforce a 1–1 relationship between tables. 也就是说A表的主键作为B表的外键时，B表中的外键值满足唯一性时，此时就是1-1关系。   
使用方法：在创建表时使用UNIQUE FOREIGN KEY REFERENCES Member.  
总结：Unique constraints are able to help us with a couple of design issues: enforcing a 1–1 relationship and
maintaining uniqueness for a candidate key that has not been chosen as a primary key.  
###Using Constraints Instead of Category Classes  
在前文中，有时候需要新建一个Category Class（一张表）来解决数据准确性问题。然而，有些情况是无需新建的。比如，书中关于等级例子，(‘Senior’, ‘Junior’, ‘Social’)，在Access中，创建表时使用语句type VARCHAR(20) CHECK type IN (‘Senior’, ‘Junior’, ‘Social’)便可解决。但是，因为创建表涉及到数据库管理员权限，当有新的等级需要添加时，这种方式就没有新建一个Category Class（一张表）方便。此外，若Category Class中需要添加额外的属性，则此时新建一个Category Class（一张表）便是唯一选择。 
###Deleting Referenced Records  
当field被用做了foreign key时，删除包含该field的表的记录，且记录中该field的数据已被别的表引用，会出现问题。在处理该问题时，有三种可能的方法：  
（1）Disallow delete，不允许删除。You cannot delete a row that is being referenced.  
（2）Nullify delete，将引用的地方置无效。
（3）Cascade delete，级联删除，即将引用的地方也删除。  

##Chapter 10 Query Basics  
###Simple Queries on One Table  
* The Project Operation  
	The project operation allows us to specify which columns of the table we would like to retrieve.  
	查询结果可能有重复数据，可以根据情况进行设定结果是否重复，关键词DISTINCT   
* The Select Operation  
	Retrieving a subset of the rows is known as a select operation.  
	在查询条件中，当条件中所涉及到的field的值有NULL时，需要注意：  
	One small but important point to bear in mind is that if a field (e.g., degree) has no value, then the truth of a statement such as degree = ‘Science’is unknown. SQL queries only return those rows for which the condition statement is known to be true.  
	上面这句话中，unknown时是肯定不满足条件的，这点要与C++中的undefined behavior对比，UB是未定义，结果未知；但是在SQL中，unknown就肯定是false.  
* Aggregates  
	COUNT(*) simply means count each record，返回符合条件的行数。  
	MAX(studentID) Finding the Maximum Value of a Field  
	all these aggregate queries can be combined with a WHERE clause to retrieve just a subset of the rows before we do the grouping and counting.  
* Ordering  
	The SQL phrase ORDER BY allows us to specify the order in which the rows are presented.  
###Queries with Two or More Tables  
RDBMS的特点之一：One really elegant feature of relational database operations is that when we carry out operations on one or more tables, we can think of the result as a new table.   
新表相当于一张虚表（a virtual table that exists for the time of the query），其也可以真实存在的表进行联合查询。  

* The Join Operation  
	* inner join  
		It is useful to think of an inner join operation as making a new virtual table that will have all the columns from both original tables.相对于后面介绍的外连接，内连接条件较强，只会返回条件为真的记录。  
	* outer join  
		LEFT OUTER JOIN, RIGHT OUTER JOIN, FULL OUTER JOIN三种。
* Set Operations  
Set （集合）operations are used on two tables(or virtual tables) that have the same number and type of columns.   
三种集合操作：INTERSECT, UNION和EXCEPT  
###How Indexes Can Help  
* Indexes and Simple Queries  
	索引信息会被存储到硬盘，利用索引检索时会加快速度。不用索引应该是N的复杂度，索引的复杂度是lgN.（N为表的行数）  
	对于外键通常也需要建立索引。
* Disadvantages of Indexes  
	索引会占用空间，数据表增加/删除记录时需要对索引进行维护。因此，创建索引前要对检索性能和记录增删性能两方面进行权衡。  
* Types of Indexes  
	索引分为两类：nonclustered和clustered.  
	A nonclustered index is where we keep the values from just one (or a couple) of the fields in order, along with a reference to the full row, which is kept elsewhere（也就是只包含表中数据的引用，而不包含具体的数据）. Nonclustered indexes具有唯一性。一张表至少有一个具有唯一性的索引。  
	A clustered index affects how the complete records or rows are physically stored on disk. A clustered index contains all the data in the table（包含表中数据）, while a nonclustered index contains only the indexing fields and a reference to the table（对比nonclustered index）.  
###Views  
Views are a way of storing the specification of a query so you can reuse it. Views are useful as a basis for forms and reports and can help with controlling access for different groups of users.

* Creating Views  
	对于view，It does not physically exist, however. If the data in the underlying tables changes, so will the resulting rows in the view. 

		CREATE VIEW Phone_view AS
		SELECT empID, last_name, first_name, phone from Employee; 
* Uses for Views  
	views are useful for retrieving data from the database.  
	Another use for views is providing some security for our data.  
##Chapter 11 User Interface  
###Input Forms  
* Data Entry Forms Based on a Single Table  
	由于student data数据较多，文中没有采用Table的方式，分行进行显示，而是采用了一种类似对话框的界面，在该界面中，每次只显示一行，而且可以在该界面中切换具体的显示的行编号，以显示对应的行。  
* Data Entry Forms Based on Several Tables  
	借助view，将多张表拼为一个view  
* Constraints on a Form  
	Check constraints are based on values in the table we are updating（限制和更新都针对的是同一张表）. 文中则是更新表A时，A中某些field的有效数据定义在了表B中，这样有时需要对表B提供的这些数据进行条件限制（比如，只提供field中的某些值），文中是使用view来实现的（Listing 11-2）。  
	有些限制可以在数据录入时进行，比如，选课时，只能选择当前提供的课程，对于过去已经取消的课程可以不加入选择列表。这样，在增加新记录和查询过往记录都不会出现问题。  
* Restricting Access to a Form  
	在前端表格中根据用户权限进行限制  
###Reports  
* Basing Reports on Views  
	对于多张表，依旧采用view，再基于该view来生成报告  
* Main Parts of a Report  
	根据view来生成报告。  
* Grouping and Summarizing  
	
###Summary  
  本章总结可以看看。  
##Chapter 12 Other Implementations  
###Object–Oriented Implementation  
OO databases provide a way of seamlessly creating objects, storing them to disk, and then finding them again.  

* Classes and Objects  
	Classes are just an abstract idea, but the objects themselves are independent entities.We construct a table to represent each class, and the objects are represented by rows.  
	We perform operations such as joins and unions on tables, not rows. Whereas operations within a relational database are firmly based on tables, in an OO environment the emphasis is on objects (hence “object oriented”).  
* Complex Types and Methods  
	在数据库中无法直接表示复杂的类型，比如一个类型由多个基本类型的属性组成，那么数据库中只能是对应多个列（fields）来进行存储。  
* Collections of Objects  
	Finding particular objects in a keyed collection is very similar to finding rows in a table that has an index on it.
* Representing Relationships  
	References to objects and collections of objects can be used to represent the relationships in a data model.
* OO Environments  
	one of the most powerful aspects of the relational model is the set of operations that we can perform on tables to retrieve complex subsets of data, as described in Chapter 10: joins, unions, intersections, and so on. The SQL commands to carry out these operations are relatively straightforward and are an integral part of any relational database system. However, the operations are all defined on tables and have no direct counterpart in an OO system. There is much work in progress to develop a set of standards for object–oriented databases and to develop OO database query languages.  
	One compromise between a full OO database and a relational database is an OO programming language connecting to a relational database.  
###Implementing a Data Model in a Spreadsheet  
从本节开始，后面章节暂时不看，没什么用处。	
	

	
	
 
	