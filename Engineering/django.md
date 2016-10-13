##Models

A model is the **single, definitive** source of information about your data. 通常，每一个model对应数据库中的一张表。  

* 基本概念  
	（1）每一个model都是继承自django.db.models.Model的类  
	（2）model的每一个属性代表数据表中的一个域  
	（3）满足（1）（2）的规则后，便可以使用django的数据库查询api  

* model与具体的数据库的关系  
	在定义了model之后，需要将包含该model的app加入到INSTALLED\_APPS中，再运行如下两个命令   
		
		python manage.py makemigrations myapp
		python manage.py migrate 
	（1）命令makemigrations  
		命令makemigrations是告诉django，我们对models做了更改，并且希望django将更改存为一个migration。当我们需要对migration进行版本管理时，该命令也是非常有用的。    
	（2）migration  
		migration是一些文件，用于存储models的更改（进而关系到database scheme），其路径通常为myapp/migrations/0001\_initial.py。虽然这是自动产生，在必要时也可以手动调整。  
	（3）命令sqlmigrate  
		
		python manage.py sqlmigrate myapp 0001  
		
	上述命令用于查看migration对应的SQL语句，也就是django会进行的数据库操作。  
	（4）命令migrate  
		migrate首先查看INSTALLED\_APPS中有哪些app，并且根据各app对应migration以及数据库设置来创建必要的table 。对于各app的migration，如何知道这些migration已经migrated？django通过在数据库中创建一张django\_migrations表来管理这些信息。

* Fields  
	model必须要定义class attributes，这对应到database的fields。  
	Field继承自django.db.models.fields，django通过Filed可以获知如下信息：  
	（1）database中field的类型，如INTEGER, VARCHAR, TEXT  
	（2）当render一个form field时  
	（3）在Django的admin和自动生成的forms中，基本的校验规则。   
	filed类型的变量名要求：名字中不能包含连续且数量超过两个的下划线；

* Field options  
	field可以指定一些特定参数（通过构造函数进行传递），不同类型的field参数不同，但也有一些通用的参数，下面列出一些最常用的：
	（1）null，若该值为True，django将空值存储为NULL，该参数默认值为False  
	（2）blank，若该值为True，则在form validation时将允许空值；blank仅用于校验。默认为False  
	（3）choices，参数值必须是tuple，且tuple中的各元素也是tuple（包含两个元素的tuple），第一个元素用于存储到数据库，而第一个元素用于显示；类似与QCombobox  
	（4）default，该field的默认值，可以是value，也可以是一个callable object（在每次创建该field object时调用）  
	（5）help\_text，在作为 form widget显示时，可以作为提示信息。  
	（6）primary\_key，若为True，则该Field为Primary Key。若在model中，所有的attribute都没有指定primary\_key为True，则Django会自动增加一个IntegerField并指定为Primary Key。Primary key是只读的，若修改了该属性，则django会自动增加一条新的记录，来存储修改后的值。每一个model仅且只能有一个属性为Primary Key. <font color='red'>此处与常理不太一致，primary key若涉及多个fields，如何处理?</font>   
	（7）unique  
		该field必须唯一    

* Verbose field names  
	除了 ForeignKey, ManyToManyField and OneToOneField这三种类型的field，其它field类型，第一个位置参数（positional argument ）可用于指定其详细名称（verbose name），当然，若该参数未指定，django会自动使用field的变量名称（会去掉下划线）。  
	ForeignKey, ManyToManyField and OneToOneField这三种类型，需要使用keyword argument来指定（keyword为verbose\_name）  

* Relationships  
	django可以设定最常用的三种关系：多对一、多对多和一对一  
	（1）多对一  
		在model中定义ForeignKey类型的属性，一般将该属性变量名称设置为关联的model的类名的小写。定义了ForeignKey类型变量的model是多，而被ForeignKey关联的model则是一  
	（2）多对多  
		在这种关系的两个model中的任何一个定义都可以，但一般在更可能被编辑的那个model（称为source model，另一个则称为target model）中定义ManyToManyField，且名称习惯用复数形式。对于多对多的两个model，两者之间的其它信息可以由第三个model（中介model， intermediary model ）来管理，中介model与这两个model通过。  
		中介model的要求：必须定义关于source model和target model的ForeignKey属性，两个model各对应一个。其它要求请参考文档。  
		在当前关系下，中介model的object只能通过自身实例化创建，而不能通过另外的两类model来进行创建，不能使用的方法包括 add(), create(), set()和remove()  
	（3）一对一  
		类似于继承关系。  

* Meta options  
	给model添加meta信息的方式，是通过在model中定义一个inner class Meta，Model metadata is “anything that’s not a field”.

* Model attributes and methods  
	（1）objects（最重要的一个attribute）   
		其类型为django.db.models.Manager，其给models提供了查询数据库的接口（操纵的层面是table）。Managers are only accessible via model classes, not the model instances.  
	（2）methods  
		操纵的层面是row-level，method关联到instance，而不是class；通过methods，可将业务逻辑放至model。  

*  Model inheritance  
	（1）Abstract base classes  
		通过在inner class Meta中，属性abstract=True来指定为抽象基类。该基类中定义的属性（Field）会自动加到其子类对应的表中。注意，这种继承关系，抽象基类不会创建table，而子类才会创建。  
	（2）Meta inheritance  
		子类也可以继承父类的inner class Meta，注意，在抽象基类的Meta中，abstract=true，那么子类在继承前，django做了一些处理，即继承前使abstract=false；若子类依旧想为抽象基类，那么可以显式的在子类的Meta中定义abstract=true.   
		在使用keyword参数related\_name and related\_query\_name时，因为继承原因，该用哪一个类名，这里需要注意，具体查看帮助文档。  
	（3）Multi-table inheritance   
		每一个model都对应一张表，父类和子类之间是一对一关系（uses an implicit OneToOneField to link the child and the parent）.  
		In the multi-table inheritance situation, it doesn’t make sense for a child class to inherit from its parent’s Meta class.   
		但是，也有一些情况，the child inherits behavior from the parent，如ordering,若子类不想继承这些行为，则可以在其inner Meta中显式声明。  
对于继承行为，因为隐式的使用了OneToOneField，这样，若在子类中又定义了父类类型的ManyToManyField，且没有显式的指定related\_name，由于子类到父类有了两种关系，而这两种都使用了默认的related\_name，那么就造成了冲突，因此，这种情况需要对related\_name调整。【注意，A->B关系中，B往后回溯到A时会使用到related\_name，为了区分所有到B的关系，不同的关系需要使用不同的related\_name。从而，若A->B有多种关系，则每种关系的related\_name都不能重名，django在自动生成related\_name时只用到了A的类名信息，这样多种关系自动生成时，它们的related\_name是相同的】
		
* Proxy models  
	在使用Multi-table inheritance时，父类和子类都会建立table，但有时候，希望子类不要自己建立table（比如，只是添加了一个新的方法），这时候，在子类的inner class Meta中，指定proxy=True即可。  
	对Proxy model进行create, delete and update instances 时，相当于对被代理的model进行同样的操作，当然，通过Proxy model进行查询时，返回的是Proxy model实例。然而，你可以在Proxy model进行一些其它更改，例如the default model ordering or the default manager，这些不会影响到被代理类。  

	* Proxy models的基类要求：  
		必须满足继承自一个非抽象的model，当然，对于抽象的model（若这些抽象的model中没有定义任何fields），可以继承自多个。从1.10版本开始（含），可以继承自多个proxy model（若这些proxy model继承自同一个非抽象的model）  

	* Proxy model managers  
		（1）若没有指定Manager，则会使用父类的Manager  
		（2）若在proxy model中定义了manager，则会成为proxy的默认manager，当然，父类的Manager依旧可用。  
		（3）若想指定一个manager，但又不让该manager成为默认的，可以定义一个抽象基类，在该基类中定义一个manager，之后继承该基类。  

* unmanaged models  
	在model的inner class Meta中，若指定了managed=false，则该model不会涉及到数据库表的创建和删除操作（若对应的表已经存在，则这种方式的model较为有用）。unmanaged models详细情况可参考django.db.models.Options.managed文档  
* Proxy models和unmanaged models的区别  
	（1）Proxy models会与其代理类自动同步（django实现）。  
	（2）unmanaged models则需要人为的手动同步，比如，两者的某个method需要修改，则需要人为的修改两个地方，That option is normally useful for modeling database views and tables not under the control of Django.  
* 多重继承  
	在多重继承时，只有第一个基类的Meta起作用，而其它基类的Meta不起作用。（遵循Python name resolution rules）。多重继承尽量使用简单的情况，不要太过复杂，若多个基类都有名为id的primary key，则会报错。  
* Field name “hiding” is not permitted  
	在Python中，子类可以覆盖基类定义的属性，而django不允许fields属性的覆盖。【从版本1.10（含）开始，允许对抽象基类的field的覆写】  

###Making queries  

* Creating objects  
	在Django中，一个model class代表一张表，而该class的一个实例则代表一条记录。  
	先通过构造函数返回一个实例化对象，在调用save()，则django会insert一条记录到数据库。这两步操作可以通过create()合二为一。  
* Saving changes to objects  
	对已经存在的记录，调用save()，则django会执行update操作。  
* Saving ForeignKey and ManyToManyField fields  
	对于外键，需要指定外键关联的那个model类的实例，之后再保存。  

		entry.blog = cheese_blog
		entry.save()
    对于ManyToManyField，需要通过add()来添加关联的model的实例（因为这是多对多，没法通过赋值进行）  
* Retrieving objects  
	通过model class的Manager（每个Model至少有一个Manager，且名称默认为objects，注意，需要通过class来访问Manager）来创建一个QuerySet，来进行查询。  
	QuerySet相当于SELECT，而它的filter类似于WHERE或者LIMIT。
	通过对Manager调用all()可以返回一个QuerySet，其包含所有records。调用filter()和exclude()也会返回一个QuerySet  

	* Chaining filters  
		The result of refining a QuerySet is itself a QuerySet, so it’s possible to chain refinements together.
	* Filtered QuerySets are unique    	
		Each time you refine a QuerySet, you get a brand-new QuerySet that is in no way bound to the previous QuerySet.   
	* QuerySets are lazy  
		QuerySets are lazy – the act of creating a QuerySet doesn’t involve any database activity. In general, the results of a QuerySet aren’t fetched from the database until you “ask” for them. When you do, the QuerySet is evaluated by accessing the database.  
	* Retrieving a single object with get()  
		我们知道，通过上面介绍的操作，返回的是QuerySet，但是当我们确定返回的记录只有一条时，可以通过get()来直接获取记录对应的model实例。  

* Field lookups  
    where语句，通过fieldname\_\_lookuptype=value的keyword arguments来进行，若只有fieldname，而没有后面的\_\_lookuptype，则相当于fieldname\_\_exact=value。对于ForeignKey的field，需要在fieldname后面加上后缀"_id"  
    
    * Lookups that span relationships   
    	To span a relationship, just use the field name of related fields across models, separated by double underscores, until you get to the field you want.  
	    （1）对于ForeignKey的field  
        格式fieldname\_\_subfieldname\_\_lookuptype;这里，subfieldname是指ForeignKey对应的那个model的field，若有多层关系，依旧可以按照这个格式连接下去（This spanning can be as deep as you’d like.）。  

		对于多层关系，若subfieldname有不存在的情况，Django在查询时，不会raise错误，而是将值当成NULL。  
		
			#return Blog objects that have an empty name on the author and also those which have an empty author on the entry.  
			Blog.objects.filter(entry__authors__name__isnull=True)

			#return Blog objects that have an empty name on the author
			Blog.objects.filter(entry__authors__isnull=False, entry__authors__name__isnull=True)  
 
		（2）对于ManyToManyField和reverse ForeignKey  
		 	使用filter()实现。对filter()来说，需要满足该filter()中的所有条件才行；对于多个filter()连接构成的查询语义，每个filter()的输入都是最开头的model（the primary model）的objects，而不是前面filter()产生的结果作为输出。其实，相当于与关系和或关系，具体看例子  

				#在filter()中的条件都要满足，与关系
				Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)  

				#两个filter的输入都是Blog.objects，或关系
				Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)
			
		关于exclude()，在帮助文档中说，the conditions in a single exclude() call will not necessarily refer to the same item，意思是exclude()括号中的条件，是或关系，也就是满足其中一个就exclude了，这对比filter()中的与关系，是不一样的。有点类似于集合中的德摩根定律。  

	* Filters can reference fields on the model  
		在前面的介绍中，field都是与常量进行比较，django还提供了与变量的比较，其提供的F表达式用于同一个model instance的两个field的比较（当然，若其中一个field有关联的model，则也支持上述的多级查找）  
	* The pk lookup shortcut  
		主键field的快捷查找，也就是将name就替换为pk，其它都一样。  
	* Escaping percent signs and underscores in LIKE statements  
		在SQL的LIKE语句中，有两个特殊字符，百分号和下划线，前者用于匹配多个字符，而后者则用于匹配单个字符；因此，在进行特殊字符查找时，直接使用SQL语句是需要转义的，但通过Field lookups，则无需转义，同理，对于下划线也是如此。   
	* Complex lookups with Q objects  
		Q对象用于封装前述的查询格式，Q对象可以用&, |, ~来进行操作，结果仍然是一个Q对象。在进行查询时，可以将Q对象和前述查询格式混合使用，混用时需要Q对象在前面。
* Caching and QuerySets  
	每一个QuerySet实例都有一个cache. 对于刚创建的QuerySet实例，其cache是空的，当QuerySet被evaluated时，Django会将数据库的查询结果放入QuerySet的cache，随后的evaluations便能利用cache  
	
	* When QuerySets are not cached  
		When evaluating only part of the queryset, the cache is checked, but if it is not populated then the items returned by the subsequent query are not cached.  
		不会populated的情况：（1） printing the queryset （2）using an array slice or an index   
* Comparing objects  
	比较pk  
* Deleting objects  
	通过调用model instance的delete()方法，进行单条记录删除；  
	通过嗲用QuerySet的delete()方法，进行批量删除，在进行批量删除时，并不会调用被删除的各model instance的delete()方法，而是直接采用SQL的语句。  
	Django在删除一个实例时，默认是ON DELETE CASCADE，也就是说所有以该object为外键的objects也会被删除。（当然，这是默认情况，This cascade behavior is customizable via the on_delete argument to the ForeignKey.）   
	
* Copying model instances  
	对于继承情况，以及related objects，需要注意。  
* Updating multiple objects at once  
	对于non-relation fields，可以直接使用update()，其中，参数用keyword arguments，赋值为常量；  
	但是对于ForeignKey fields，set the new value to be the new model instance you want to point to.  
	
##Views  
###URL dispatcher  
 URL patterns到Python Function的映射。  

* Django处理请求的过程  
	（1）确定root URLconf module  
		若 HttpRequest object中有urlconf属性，则使用其作为root URLconf；否则，使用ROOT_URLCONF setting.  
	（2）django查找名称为urlpatterns的变量，该变量为一个list，元素类型为django.conf.urls.url()【注意，即该函数的返回值】  
	（3）遍历list，当第一次匹配发生，django imports and calls the given view（一个Python 函数，），同时会传递参数给该view，包括：  HttpRequest；匹配若返回no named groups，则当做位置参数传递给view；若返回named groups ，则会当做keyword参数传递；**传递的参数值都为字符串，即使是如year='2016'，也是字符串**   
	（4）若没有匹配，或者匹配过程中发生了错误，则Django invokes an appropriate error-handling view. 	
	
* The matching/grouping algorithm  
	关于named groups和non-named groups的格式，在正则表达式中， (?P<name>pattern)用于named groups参数的捕获，而(pattern)则用于non-named groups参数的捕获。  
	关于named groups和non-named groups的解析规则：  
	（1）若正则表达式中有named groups参数，则会忽略掉任何non-named groups参数；  
	（2）non-named groups只有在没有named groups时，才会被捕获，并作为位置参数传递。  

* Error handling  
	已提供4个变量，可以满足大多数情况，分别是handler400, handler403, handler404, handler500。当然，也可以自定义；注意，值必须是callables, or strings representing the full Python import path to the view that should be called to handle the error condition at hand.  
* Including other URLconfs  
	类似于头文件的包含，django先将匹配后的剩余部分字符，发送至包含的url继续进行匹配处理。包含的变量只要与urlpatterns同类型即可。  
	An included URLconf receives any captured parameters from parent URLconfs.  
* Nested arguments  
	我们知道，通过在正则表达式中添加()，便是要捕捉参数，但是，若正则表达式本身的()怎么办，使用格式 (?:...)，称为a non-capturing argument.关于reversing,需要参考revese resolution of URLs。  
* Passing extra options to view functions  
	django.conf.urls.url()可以接收第三个参数，类型需要为dictionary，这样，将该dictionary的元素作为keyword arguments传递给view。当dictionary的参数名称与捕获的named group相同时，则dictionary优先。  
	当然，在include时也能将dictionary的参数传递过去。【注意，因为include中的每一个url对应的view都会接收到dictionary的参数】  
* Reverse resolution of URLs  
	存在这样的需求，在view中，会继续用到已经被解析（匹配）的url，也就是需要根据一定规则还原出传递进来的url（django称之为URL reversing），django根据不同模块，提供了三种方式：  
	（1）template中，使用url标签；  
	（2）view中，使用reverse()  
	（3）model中，使用get\_absolute\_url()  
	
	* Naming URL patterns  
	为了进行URL reversing，需要将url进行命名，也就是传递参数name='myapp.myurl'，在具体命名使，请使用后文介绍的URL namespaces。  
	* URL namespaces  
	URL namespace由两部分组成：application namespace和instance namespace。  
	Application namespace describes the name of the application that is being deployed.  
	Instance namespace identifies a specific instance of an application. Instance namespaces should be unique across your entire project. Instance namespace默认情况下与Application namespace名称相同。
	一个app的每一个instance都有相同的application namespace，但有不同的instance namespace。  
	* Reversing namespaced URLs  
	对于
##RESTful API  
Representational State Transfer


	