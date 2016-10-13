#学习  
本节笔记参考内容来自[官网][Developer_guide_url]  
##Introduction to Android   

* Apps provide multiple entry points  
	一个activity代表一个用户界面，service用于后台执行。组件（component）的概念，组件之间可以通过intent进行通信。  
* Apps adapt to different devices  
	Android provides an adaptive app framework that allows you to provide unique resources for different device configurations.  
###Application Fundamentals  
Android程序最后以.apk（Android package）文件呈现，里面包括代码和资源文件。 
 
* the principle of least privilege（最少特权原则）：  
	（1）每一个app运行在自己的安全沙箱内（security sandbox）。  
	（2）Android系统是一个多用户的Linux系统，每一个app对应一个用户。  
	（3）默认情况下，Android会为每个app分配一个只有操作系统知道的用户ID（app无法知道自己的user ID）。只有app对应的用户ID才有权限访问app的文件。  
	（4）系统给每一个进程都分配一个虚拟机（Virtual Machine）。      
	（5）默认情况下，每一个app会运行在其专有的Linux进程中。当app的组件（components）需要被执行时，Android会开启该进程，当app不需要再被执行或者必须要进行内存回收时，Android会关闭该进程。   
* 例外情况  
	当然，也有少许例外，多个app之间共享数据，以及app需要访问系统服务（ system services）时，就可能有如下情况存在：  
	（1）两个app共用一个相同的Linux user ID，以便互访各自的文件。共用同一个user ID的app，会运行在相同的Linux进程中，且共用相同的虚拟机，这些apps必须要有相同的签名。  
	（2）app需要获取系统权限（一般情况下，app通过所分配的User ID只具有访问自身文件的权限）时，比如通讯录、短信等    
####App Components  
Components是一个APP最基本的组成（ the essential building blocks），每一个component是app的一个进入点。each one is a unique building block that helps define your app's overall behavior.  
Android一共有四种组件，用途各异，且生存周期各异（何时创建和销毁）。下面依次予以介绍：  

* Activities  
	一个activity就对应屏幕中的展示的一屏UI，每个activity是独立的，多个activity便组成了一个APP。Activity由继承Activity的子类实现。  
* Services  
	Service没有UI，是长时间运行于后台执行较为耗时的工作，或者为remote processes执行任务。  
* Content providers  
	用于管理共享数据，当然，对于也能用于私有数据的读写操作。  
* Broadcast receivers  
	用于响应系统级的广播信息。其也没有UI，但可以创建状态栏。当然，APP除了响应以外，也可以发送广播事件。  

Android系统独有的一个特点是，app可以启动别的app的component。当系统启动一个component时，也就为该app开启了一个进程（若app当前没有运行），同时实例化该组件对应的classes.这也就是说Android程序有多个进入点（不像Windows中，只能通过main()进入）。我们知道，每一个app在独自的沙箱中运行，那么，其是不具备开启另一个app的component的权限的，在具体实现中，是通过Android系统来启动另一个app的component，而app只需要传递intent信息给系统即可。  
####Activating Components   
组件activities, services和broadcast receivers，是通过异步消息intent启动（Activated）。Intent用于启动一个特定的组件或者一种特定类型的组件。  
组件content provider，则不是通过Intent启动，而是 通过ContentResolver的请求进行激活。ContentResolver相当于对content provider又进行了一层包装，这样，其它组件就不会直接跟content provider交互，保证了安全性。  
####Intents and Intent Filters  

* 用于组件之间通信，有如下三大基本用途：  

	* To start an activity   
		这种情况下，The Intent describes the activity to start and carries any necessary data.  
	* To start a service   
		The Intent describes the service to start and carries any necessary data.  
	* To deliver a broadcast:  
		You can deliver a broadcast to other apps by passing an Intent to sendBroadcast(), sendOrderedBroadcast(), or sendStickyBroadcast().   
* 两种类型的Intent  
	* Explicit intents   
		直接通过组件的class名称（the fully-qualified class name）来启动组件。一般只有APP自身组件才通过这种方式，当然，若其它APP开发者知道某APP的class名称，也可以通过这种方式启动。若不想让外部APP以这种方式启动，可以在AndroidManifest.xml文件中的<Activity\>标签中的属性"exported"，设置为false即可。    
	* Implicit intents   
		没有指定具体的组件class名称，而是声明了action，以启动能够处理这种action的组件。若没有组件能够处理这种action，则会使程序挂掉，因此，之前需要用Intent的resolveActivity()方法进行检查是否有组件响应    
	* 注意事项：为保证系统安全，启动service应尽量用explicit intents.   
* 创建Intent  
	Intent包含有如下信息，其中，前面四种(component name, action, data, and category) ，可以帮助Android OS来确定哪些组件用于响应该Intent：  
	* Component name  
		若是Explicit intents，则必须指定该信息。对于Service组件，应该使用Explicit intents，否则，你无法确定Service是否响应了该Intent.（因为Service对用户来说不可见）  
	* Action  
		若要以Implicit intents方式启动组件，则必须指定Action. 系统会根据Intent携带的Action信息来选择能够处理该Action的组件进行处理。那么，若是自定义的组件，打算响应某些Action，则需要在AndroidManifest.xml中进行声明。详见后续的相关章节笔记。  
	* Data  
		用于指定data的URI，若data是MIME类型，还需要指定其类型。  
	* Category  
		A string containing additional information about the kind of component that should handle the intent.  
	* Extras  
		Key-value pairs that carry additional information required to accomplish the requested action.   
	* Flags  
		Flags defined in the Intent class that function as metadata for the intent. The flags may instruct the Android system how to launch an activity (for example, which task the activity should belong to) and how to treat it after it's launched (for example, whether it belongs in the list of recent activities).   
* Receiving an Implicit Intent  
	若组件需要响应Implicit Intent，则需要在AndroidManifest.xml中，定义<intent-filter> ，对于不同的工作，需要定义不同的<intent-filter>  
* Intent Resolution  
	* Action test  
		在<intent-filter>下的action，逐一进行比较，若匹配成功，则该组件通过Action test；若请求的Intent没有包含Action，而组件的<intent-filter>至少有一个Action，则会通过Action test  
	* Category test  
		For an intent to pass the category test, every category in the Intent must match a category in the filter.需要注意的是， Android automatically applies the the CATEGORY_DEFAULT category to all implicit intents passed to startActivity() and startActivityForResult()，所以在这种情况下，要响应这种Intent，则需要包含"android.intent.category.DEFAULT"的category.  
	* Data test  
		When the URI in an intent is compared to a URI specification in a filter, it's compared only to the parts of the URI included in the filter.   
		The data test compares both the URI and the MIME type in the intent to a URI and MIME type specified in the filter.   
		[具体规则见文档][intent_resolution_url]  
	

####The Manifest File  
Android系统如何知道一个app中有哪些组件呢？通过对AndroidManifest.xml文件(在main目录下，也就是与java文件夹和res文件夹所在位置相同，且名字必须为AndroidManifest.xml)的解析得知的。因此，我们必须将所有组件信息存储于该文件中。除了组件信息之外，该文件还包括如下信息：  
（1）声明该APP所要求的权限  
（2）声明最小API level  
（3）声明所要求的硬件和软件特征  
（4）声明该APP所要求Android framework APIs之外的其它API libraries  
（5）用于命名app的java包，包名称是app的唯一性标示。

* Declaring components  
	通过标签进行声明。对于Activities, services和content providers这三种类型的组件，若不在AndroidManifest.xml中声明，则系统是不知道有这些组件的，进而，涉及到的代码也不会运行。对于broadcast receivers组件，可以在文件中声明，也可以动态创建。  
* Declaring component capabilities  
	The real power of intents lies in the concept of implicit intents. The way the system identifies the components that can respond to an intent is by comparing the intent received to the intent filters provided in the manifest file of other apps on the device.  
	因此，在The Manifest File中，可以为组件添加<intent-filter>，以便被相关的Intent启动。  
* Declaring app requirements  
	API版本要求，某些硬件要求。   
* <intent-filter>  
	用于activity, service, or broadcast receiver，相当于指定某些类型的Intent，以便组件（Activity/Service/Broadcast Receiver）能够响应这些Intent. 若不指定<intent-filter\>，则组件只能显式的启动。   
	* <action\>   
		 <intent-filter>标签下面必须要包含至少一个<action\>，否则，no Intent objects will get through the filter. action只有一个属性，即android:name,值为string.Android系统已经定义了一些标准的Action值，并且以android.intent.action.打头。对于自定义的action名称，最好用项目的包名打头。  
		当我们在<intent-filter> 标签下定义了action之后，便可在代码中通过action来建立Intent，进而启动相应的组件。  
	* <category\>  
		该标签只有android:name，值为string.  
	* <data\>
* <meta-data>  
	主要提供一些共用的key-value参数  
* <permission\>  
	用于自定义本APP的权限，每一个<permission\>代表一种权限，可以被组件使用，那么别的APP需要使用本APP指定了权限的组件时，那么别的APP就需要在其AndroidManifest.xml的<user-permission>中声明需要该权限。  
* <user-permission>  
	用于声明APP所需要的权限  
	
	 
	
####App Resources  
图片、声音、动画、菜单、样式和布局等。对于每一个资源（比如一个菜单名称，一张图片等），the SDK build tools 会为其定义一个唯一的整型ID，以便在代码中引用。   
###Device Compatibility  
####What Does "Compatibility" Mean?  
两种兼容性：device compatibility和app compatibility.开发人员只用考虑后者，因为android设备非常的多样。  
####Controlling Your App's Availability to Devices  
Android supports a variety of features your app can leverage through platform APIs.   
   
###User Interface  
所有UI元素都是通过View和ViewGroup对象构建。对于布局，可以在代码中实现，也可以使用xml文档定义，推荐使用后者。xml文件中，标签的名称即对应Android中的一个类，比如TextView。当在app中载入layout文件时，Android会对这些标签对应的类型进行实例化。  
####Layouts  
推荐使用xml，因此笔记中只对xml方式布局进行了总结。  
下面这段话讲得很好。  
In general, the XML vocabulary for declaring UI elements closely follows the structure and naming of the classes and methods, where element names correspond to class names and attribute names correspond to methods. In fact, the correspondence is often so direct that you can guess what XML attribute corresponds to a class method, or guess what class corresponds to a given XML element.   

* Write the XML  
	布局文件必须要有且只能有一个root element，可以是View或者ViewGroup. 文件放入/res/layout目录。  
* Load the XML Resource  
	在编译项目时，布局文件会被编译为一个View资源，你可以在Activity的onCreate()方法中引用，引用方式为R.layout.layout\_file\_name.  
* Attributes  
	View的有些属性是共有的（如View或ViewGroup的属性就被继承者享有），而有些属性则对某种类型的View才有。  
	（1）ID  
	都具有。在xml文件中，使用string值，但编译结果是整型值。   

		<!-- ‘+’表示新建，@表明XML parser需要对后续的ID 
			String进行解析并将其识别为ID资源-->
		android:id="@+id/my_button"
		<!-- 表示从android.R中引用ID资源 -->
		android:id="@android:id/empty"
	ID是可以重复的，但是，最好依据如下原则：  
	An ID need not be unique throughout the entire tree, but it should be unique within the part of the tree you are searching (which may often be the entire tree, so it's best to be completely unique when possible).   
	（2）Layout Parameters   
	这些属性是以"layout\_"为前缀的，比如layout\_width和layout\_height. 用于指定layout参数的值，一般用相对值，而不是绝对值，比如density-independent pixel units (dp), wrap\_content, or match_parent.  
####Layout Position  
可以用矩形表述View位置的参数，通过左上顶点的坐标，以及长宽确定，单位是pixel.  
####Size, Padding and Margins  
一个View其实有两组长宽参数。第一组称为measured width和measured height，用于定义View期望的尺寸；第二组就是width and height（有时候称为 drawing width and drawing height），用于定义实际的尺寸。	
在计算View大小时，View需要考虑Padding所占用的空间。对于Margin，则只有ViewGroup支持。  
####Common Layouts  
LinearLayout, RelativeLayout和WebView  
####Building Layouts with an Adapter  
A subclass of the AdapterView class uses an Adapter to bind data to its layout. The Adapter behaves as a middleman between the data source and the AdapterView layout—the Adapter retrieves the data (from a source such as an array or a database query) and converts each entry into a view that can be added into the AdapterView layout.  
例如，ListView和GridView  
Android提供了两种常用的Adapter，ArrayAdapter和SimpleCursorAdapter.   
* Adapter  
	Adapter在数据与视图之间起着桥梁作用，The Adapter provides access to the data items. The Adapter is also responsible for making a View for each item in the data set.  
* ListView  
	ListView是ViewGroup的子类（非直接继承）。其显示的item（View）由Adapter提供，如前所述，Adapter将数据源转换为View，再由ListView进行显示。   
###Supporting Different Devices  
####Supporting Different Screens  
屏幕的两大参数：size和density。  
size分为四大类：small, normal, large, xlarge；  
density分为六大类： low (ldpi), medium (mdpi), high (hdpi), extra high (xhdpi), extra-extra-high(xxhdpi), and extra-extra-extra-high(xxxhdpi).   
手机放置方向，水平和垂直，分别对应不同的size。  

Density-independent pixel (dp):抽象的像素单位，其与实际像素之间的换算关系为：px = dp (dpi/160)

http://developer.android.com/guide/practices/screens_support.html#qualifiers

http://developer.android.com/guide/topics/resources/providing-resources.html#BestMatch

#####Create Different Layouts  
为适应不同屏幕，增强用户体验，可以根据size种类，建立不同的layout文件，来进行适应。比如，适应large size的布局，可以放置到layout-large文件夹下。  
layout-land文件夹下用于水平放置时的布局。默认情况下，不带-land后缀的layout用于垂直布局。  
综上，我们可能用到的布局文件夹名称有layout, layout-land, layout-large, layout-large-land等。  
#####Supporting Different Screen Sizes  
* Use "wrap\_content" and "match\_parent"  
* Use RelativeLayout  
* Use Size Qualifiers  
	为不同的屏幕尺寸提供不同的layout文件。  
* Use the Smallest-width Qualifier  
	layout-sw600dp表示适配这样的机器，其最小宽度大于等于600dp。不过这对于3.2版本以前的机器不适用。  
* Use Layout Aliases  
	对于3.2以前的系统，依旧需要使用larger之类的后缀，而不能使用sw600dp，那么，对于一个app，若其可在3.2前后的版本都能使用，那么，有可能会出现这种情况，在layout-large和layout-sw600dp文件夹下，分别都会有同一个layout文件，只是前者被3.2以前的系统使用，后者则被3.2及以后的系统使用。那么，如何避免重复呢？有一种解决方案如下，将那个重复文件放在layout文件夹下，若其与layout中的其它文件重名，改之。之后，在values文件中，加入下述语句即可。  

		res/values-large/layout.xml:
		<resources>
    		<item name="main" type="layout">@layout/main_twopanes</item>
		</resources>
* Use Orientation Qualifiers  
	结合Layout Aliases和the configuration qualifiers，可以复用layout文件。  
#####Supporting Multiple Screens  

#####Create Different Bitmaps  
 


#工程实践  
##入门篇  
###APP开发的过程  
* 开发环境搭建  
	本文只对使用Android Studio进行开发的方式，进行环境搭建介绍。其实环境搭建就是Android Studio的安装，当然，还有JDK。  
* 开发  
	创建工程，写代码  
* 生成、调试和测试  
	不用多说。介绍一下工具：  
	（1）Gradle     
	To compile and build your Android project into an installable .apk file(s).  
	（2）adb （Android Debug Bridge）  
	it lets you communicate with an emulator instance or connected Android-powered device.   
	（3）Android Device Monitor   
	Android Device Monitor is a stand-alone tool that provides a graphical user interface for several Android application debugging and analysis tools.   
	其又包括一下几个工具：  
	* DDMS（Dalvik Debug Monitor Server）   
		it provides port-forwarding services. 提供的功能很强大，具体看文档。  
	* Tracer for OpenGL ES   
		用于图像处理分析  
	* Hierarchy Viewer  
		用于对UI布局的优化
	* Systrace  
		The Systrace tool helps analyze the performance of your application by capturing and displaying execution times of your applications processes and other Android system processes.  
	* Traceview  
		Traceview is a graphical viewer for execution logs saved by your application. Traceview can help you debug your application and profile its performance.
	* Pixel Perfect magnification viewer  
	* 反编译工具[apktool][apktool_url]  

* 发布   

###使用Gradle构建  


###SQLite
查询速度问题：
http://stackoverflow.com/questions/4298542/android-sqlite-performance-with-indexes
##API文档阅读笔记  
以下分为若干部分来对Android文档进行阅读总结。当然，所读到的部分不限于API文档，也可能与前述部分有重叠。  
###Content Providers   
Content Providers用于管理结构化数据，其对数据进行了封装，也提供了定义数据安全的机制，并对外提供标准接口，使用该接口，即使访问数据的代码不在数据所在线程，也能进行访问。  
使用ContentResolver作为一个客户端（client），来通过Content Providers访问数据。后者接收请求，执行操作，最后返回结果。  

* 特点  
	 Content Providers对外展现数据形式类似于RDBMS中的数据表。因此，接口的参数与SQL语句中的相关部分可以对应，比如URI对应table name.    
* 使用方式  
	如上述，利用ContentResolver作为client访问。Content Providers对象在app所在进程，而ContentProvider对象则在拥有该Provider的app进程，后者自动处理IPC。  
* 抽象层  
	ContentProvider处于实际数据和对外展现之间，即its repository of data and the external appearance of data as tables  
* Content URIs  
	用于识别provider中的数据（identifies data in a provider），其包含两部分，authority和path，前者是provider的标记名称（symbolic name），后者是类似于表名称。举个例子  

		/* content://用于表示这是一个content URI，称为scheme；
		   user_dictionary是provider's authority
		   words是the table's path */
		content://user_dictionary/words  
* Content Provider Permissions  
	A provider's application can specify permissions that other applications must have in order to access the provider's data.   
	若app要使用其它app的provider，则需要声明权限。例如，

		 <uses-permission android:name="android.permission.READ_USER_DICTIONARY">
	对于自定义的ContentProvider，可以在AndroidManifest.xml中，对其<provider\>的属性android:writePermission和android:readPermission进行赋值，以次声明读和写该provider的权限。赋的值也是在AndroidManifest.xml中通过标签<permission\>进行定义。
* 增删改  
	insert：使用ContentValues，将一行数据的所有列put到该类型的对象中，再进行insert操作。  
	update：使用ContentValues，将要更新的值put到该类型的对象中，再进行update操作。  
	delete：只需要指明条件即可。  
* Provider Data Types  
	支持的数据类型有：文本，整型，长整型，单精度浮点型，双精度浮点型，BLOB。  
	* MIME Type Reference  
		ContentProvider.getType()返回的是MIME Type，这种类型由两部分组成，type/subType，返回的type可以是常用的已知类型，如text/html，表示type是text，而subType是html；对于自定义的类型（也称"vendor-specific" MIME types），type的返回值只能是两种，vnd.android.cursor.dir（对应查询结果是多行时）和vnd.android.cursor.item（对应查询结果为1行），而子类型是 provider-specific了（也就是在自定义的ContentProvider中定义，可以用已有的类型，也可以根据自己的APP来定义合适的）。
		
* 总结  
	官方文档称一般无需自定义ContentProvider，只有在与其它APP共享数据（包括 copy and paste complex data or files from your application to other applications），以及在自己的APP中需要对自定义的数据库进行查询（文档原话，中文是我的理解：to provide custom search suggestions in your own application）。我们知道，ContentProvider通过Authority进行区分，一个ContentProvider可以对应多个authorities（只需要在AndroidManifest.xml中进行赋值，以;隔开），当我们以下面语句来进行数据操作时，若Uri（注意前面笔记，关于URI和authority的关系）是对应自定义的ContentProvider，则是调用该provider的对应操作。    

		context.getContentResolver().update(
									ScheduleContract.SearchIndex.CONTENT_URI,
                					new ContentValues(), null, null);
	我们可以这么思考，若URI是自定义的，那么，对其的操作理应也是自定义的，因为系统还没有智能到通过一个未知的URI名称来操作数据（若我们没有定义URI对应的ContentProvider，比如这个provider是操作数据库的，那么没有我们的定义，系统怎么知道这个URI操作的数据库名称叫什么呢？）。对于系统已经定义过的URI，比如UserDictionary.Words.CONTENT_URI，那么对其操作就无需自定义ContentProvider了。还有一点，这也是Android普遍的特点，就是我们无需自己实例化自定义的ContentProvider，通过上述语句，只要URI指向了我们自定义的provider，那么，对应的方法就会调用。
	
###Service  
Service对象本身是在主线程中，若其要做CPU密集型的工作，则其要spawn its own thread in which to do that work. Service的启动方式有两种，Context.startService()和Context.bindService().  

* Service不是什么   
	（1）其不是一个单独的进程，一般情况下（<font color='red'>特殊情况是什么</font>），其在app所在的进程运行。  
	（2）其不是一个线程。
* 两大特点  
	（1）Service被APP用于告知Android OS，该APP想做background的工作。通过调用Context.startService()来进行,直到服务停止。  
	（2）Service用于为其它APP提供服务。通过Context.bindService()进行。  
* Service Lifecycle  
	根据前述的两大特点，APP（Client）使用Service有两种方式：  
	(1)当APP调用Context.startService()后，OS将搜寻服务，若服务没有创建，则调用Service对象的onCreate()函数，接着调用onStartCommand(Intent, int, int)，启动服务，Service将在Context.stopService() or stopSelf()停止服务。  
	(2)APP也能通过Context.bindService()来与Service建立连接，同样，若服务没有创建，也会调用onCreate()函数，但与方式（1）不同的是，不会调用onStartCommand(Intent, int, int)函数。APP可以调用onBind(Intent) 获得一个IBinder对象。<font color='red'>待添加</font>。  
	Service可以同时以两种方式被使用。  
* Process Lifecycle  
	Service所在进程的声明周期。  
###Loaders  
* 特点  
	可以在Activity和Fragment中使用；
	异步方式载入数据；  
	监视数据源，当数据改变时，会更新结果；  
	当配置参数发生变化，自动重连到最后一个loader的cursor处。  
* Loader API Summary  
	（1）LoaderManager  
	用于管理Loader实例，每个Activity或者Fragment仅有一个LoaderManager，但可以有多个Loaders。  
	（2）LoaderManager.LoaderCallbacks  
	A callback interface for a client to interact with the LoaderManager.  
	（3）Loader  
	用于异步载入数据，比较常用的有AsyncTaskLoader和CursorLoader，CursorLoader继承自AsyncTaskLoader.    
	（4）AsyncTaskLoader  
	Abstract loader that provides an AsyncTask to do the work.  
	（5）CursorLoader  
	A subclass of AsyncTaskLoader that queries the ContentResolver and returns a Cursor.  
####Using Loaders in an Application  
使用Loaders的场景如下：

- 在Activity和Fragment中  
- 实例化LoaderManager  
- 使用CursorLoader，或者具体化Loader或AsyncTaskLoader.  
- 实现LoaderManager.LoaderCallbacks接口，这也是创建新Loaders或者引用已创建的Loaders的途径。  
- 使用SimpleCursorAdapter    
#####Starting a Loader  
在Activity的onCreate()或者Fragment的onActivityCreated()方法中，初始化Loader，调用LoaderManager的initLoader()方法进行。  
初始化结果：  
（1）若初始化的Loader ID已经存在，则之前的Loader对象会被重用。  
（2）若不存在，则会回调LoaderManager.LoaderCallbacks接口中的方法onCreateLoader()，在该方法中，你需要实例化并返回一个新的Loader。  
#####Restarting a Loader  
To discard your old data, you use restartLoader().  
#####Using the LoaderManager Callbacks  
顾名思义，用于用户与LoaderManger进行交互。  

* onCreateLoader  
	如前述，在调用initLoader()后，首先LoaderManager会根据ID进行查找，若无此Loader,则会回调onCreateLoader()方法。  
* onLoadFinished  
	在Loader完成新数据载入之后，该方法保证在供应给Loader的旧数据被释放前被回调。因此，在该方法内，需要对所有使用到旧数据地方进行处理（因为旧数据将被release，但不应该由用户来进行release操作，Loader在知道APP不再使用这些旧数据后，会release）  
* onLoaderReset  
	This method is called when a previously created loader is being reset, thus making its data unavailable. This callback lets you find out when the data is about to be released so you can remove your reference to it.  
###其它  
#### AbstractThreadedSyncAdapter   

- 同步操作   
	An abstract implementation of a SyncAdapter that spawns a thread to invoke a sync operation. 当有同步请求时，会在spawn的线程中调用onPerformSync(Account, Bundle, String, ContentProviderClient, SyncResult)进行同步操作。（注意，当同步操作进行时，会对新的同步请求返回错误）

- 关于取消同步操作的注意事项   
	同步操作可以在任意时间被系统取消。比如，默认情况下超过30分钟（Timeout）会被取消，还有在一分钟内，网络流量接近0也会被取消。此外，也可以通过调用ContentResolver对象的方法cancelSync(Account, String) or cancelSync(SyncRequest)进行取消。  
	在同步操作的线程，通过interrupt()来取消同步，因此，要么在onPerformSync()函数中，检查interrupted()来判断是否被取消，要么覆写onSyncCanceled()函数来处理取消事件。若两者都不采用，则有可能整个APP的进程被kill。  
- 与Service组件的配合  
	继承该类。创建Service，在Service的onBind()中调用AbstractThreadedSyncAdapter的方法getSyncAdapterBinder()以返回。同时在AndroidManifest.xml中的Service模块做相应配置。 
#### AbstractAccountAuthenticator  
待添加 
####Creating a Sync Adapter  
用于客户端和服务器端的数据同步。要使用Android提供的sync adapter Framework，需要做以下事情：  

* 创建一个Sync adapter class  
	 也就是实现AbstractThreadedSyncAdapter的派生类。其中，要实现ctor（即构造函数）和onPerformSync()。  
	在ctor中，做一些初始化工作（setup tasks），类似Activity.onCreate()为产生一个Activity所做的工作一样。ctor有两个版本，有一个是Android3.0才加入的，可以支持并行。这两个ctor都是需要实现的。  
	在onPerformSync()中，做数据同步的工作。当sync adapter framework需要进行数据同步时，会调用onPerformSync(). 在这里需要对该函数接收的参数做一个说明：  
	* Account  
		触发同步操作的事件所关联的账户信息。若服务器端不需要使用账户，则数据同步代码中可以不使用该Account对象中的信息。  
	* Extras  
		触发同步操作的事件所关联的flags  
	* Authority  
		The authority of a content provider in the system.一般是对应到APP中的一个content provider   
	* Content provider client  
		前面Authority参数所对应的content provider的ContentProviderClient，其是一个关于ContentProvider对象的轻量级公共接口，功能有点类似于ContentResolver  
	* Sync result  
		发送至sync adapter Framework的结果信息。  
	
* 绑定Service：Bind the Sync Adapter to the Framework  
	我们在第一步中，实现了sync adapter component（也就是AbstractThreadedSyncAdapter派生类），但是，若要sync adapter Framework来调用AbstractThreadedSyncAdapter派生类中的代码（即onPerformSync()），那就需要创建一个bound Service，以从sync adapter component中返回一个binder到framework。
 	一般的做法是，在Service的onCreate()中，使用单例模式，来创建sync adapter component实例。在sync adapter Framework进行数据同步时，会开启Service，从而在Service的onCreate中创建sync adapter component实例. 在实例化的过程中，必须要保证线程安全。  

* Creating a Stub Authenticator  
	authenticator提供了一个标准接口，用于处理用户的登录等认证信息。即使APP不需要账户信息，依旧需要提供authenticator component。同样，需要绑定Service，以便sync adapter framework调用authenticator的方法。  
	* Add a Stub Authenticator Component  
		实现AbstractAccountAuthenticator的派生类。对于无需实现的抽象函数，可以使其返回null或者throw an exception.  
	* Bind the Authenticator to the Framework  
		与sync adapter component不同的是，无需考虑线程安全。  
	* Add the Authenticator Metadata File  
		为了能够将Authenticator Component加入到sync adapter and account frameworks，需要为这些框架提供描述Authenticator Component的meta-data。这些meta-data用于描述账户类型和展示用户账户的UI信息。在res/xml目录下，定义一个xml文件存放该meta-data信息，名称通常为authenticator.xml。  
		该xml文件中，只有一个元素<account-authenticator>，有下列属性：  
		* android:accountType  
			The sync adapter framework 要求每一个sync adapter有一个account type，以域名的形式表示。The framework将其作为sysc adapter的内部标识的组成部分。同时，在发送给服务器的数据中，也会将account type进行发送。  
		* android:icon
			若sync adapter visible，必须要提供该信息。  
		* android:smallIcon  
			在小屏幕上，可能用该信息代替android:icon  
		* android:label  
			用于向用户表示账户类型。   
	* Declare the Authenticator in the Manifest  
		我们前面声明了meta-data，那么，如何将meta-data所在的文件与Authenticator组件关联？我们可以通过在Authenticator所绑定的服务中，其<meta-data>属性下进行（也就是AndroidManifest.xml中，对应的Service所在的<service\>标签下）。   
* Add the Account Required by the Framework  
	因为每个sync adapter要有一个accout type，将上一步创建的accout type添加到AccountManager中。  
* Add the Sync Adapter Metadata File  
	为了能够将sync adapter component加入到sync adapter framework，还需要为框架提供描述the component and provides additional flags的metadata。该metadata用于指定sysc adapter的account type，声明content provider authority等。在res/xml目录下，定义一个xml文件存放该meta-data信息，名称通常为syncadapter.xml。  
	该xml文件中，只有一个元素<sync-adapter>，有下列属性：  
	* android:contentAuthority  
		The URI authority for your content provider.   
	* android:accountType  
		The value must be the same as the account type value you provided when you created the authenticator metadata file.  
	* android:userVisible  
		
	* android:supportsUploading  
		app是否需要upload数据   
	* android:allowParallelSyncs  
		注意这句话：Allows multiple instances of your sync adapter component to run at the same time.   
	* android:isAlwaysSyncable  
		 Indicates to the sync adapter framework that it can run your sync adapter at any time you've specified.  
* Declare the Sync Adapter in the Manifest  
	Once you've added the sync adapter component to your app, you have to request permissions related to using the component, and you have to declare the bound Service you've added.  
	除了网络访问的权限外，还需要如下两个权限：  
	android.permission.READ\_SYNC\_SETTINGS：Allows your app to read the current sync adapter settings.  
	android.permission.WRITE\_SYNC_SETTINGS：Allows your app to control sync adapter settings.    
	在对应的<service\> 节点，将meta-data标签添加进来。且添加<intent-filter>，为其加入属性<action android:name="android.content.SyncAdapter"/> 。 When the filter is triggered, the system starts the bound service you've created.  

经过以上步骤，便创建完成了Sync Adapter的创建。  
####Creating a Stub Content Provider  
The sync adapter framework is designed to work with device data managed by the flexible and highly secure content provider framework.因此，我们必须要定义自己的content provider，这样才能使用同步框架来进行数据同步。但是若你管理本地数据并不是用的content provider，那么还能不能利用同步框架呢？其实，还是可以的，这就需要通过创建Stub Content Provider来完成。   

A stub provider implements the content provider class, but all of its required methods return null or 0. If you add a stub provider, you can then use a sync adapter to transfer data from any storage mechanism you choose.   

* Add a Stub Content Provider     
* Declare the Provider in the Manifest  

经过以上两步操作，便可以使用sync adapter framework，主要是因为同步要借助Content Provider的requestSync()函数。  


####Running a Sync Adapter  
四种情况，不建议通过提供一个刷新按钮的形式，来让用户手动更新。  

* Run the Sync Adapter When Server Data Changes  
	首先，服务器端发送信息到BroadcastReceiver，为了回应广播，调用ContentResolver.requestSync()来进行数据同步。现在有很多类似Google Cloud Messaging (GCM)的产品，若使用GCM，只有在消息到达时，才会激活BroadcastReceiver，从而可以响应同步。不建议使用轮询的方式查询服务器，因为这需要一直开启一个Service进行周期性的查询。  
	注意：若是服务器同时推送到所有设备，这些设备基本上会同时进行同步操作，这会对服务器访问造成压力。因此，最好是分开推送。  
* Run the Sync Adapter When Content Provider Data Changes  
	当更新Content Provider时，也想更新服务器端的数据，这就需要为content provider注册一个观察者。观察者是通过继承ContentObserver 实现，覆写其onChange()函数，在该函数中调用content provider的requestSync()进行同步。在Content Provider的数据被修改后，the content provider framework (注意framework类型)会告知其观察者，也就是调用观察者的的onChange()函数，因而，我们若在该函数中调用requestSync() 便进行了自动同步。在registerContentObserver()中，可以传递想观测数据的content URI.  
	注意，若APP中使用sync adapter framework是借助了Stub Content Provider（见前面笔记），因为此时数据不是通过Content Provider进行操作管理，故是否有数据变化，ContentProvider是不知道的，也就是说即使注册了观察者，在数据变化时，观察者的onChange()也不会被调用。这时的同步机制（从数据变化到调用requestSync() ）需要自己实现。  
* Run the Sync Adapter Periodically  
	通过addPeriodicSync()、AlarmManager和setSyncAutomatically()等方式触发。   
* Run the Sync Adapter On Demand  
	这种方式不推荐，The framework is specifically designed to conserve battery power when it runs sync adapters according to a schedule.   
	使用方法： 首先set the sync adapter flags for a manual sync adapter run, then call ContentResolver.requestSync().  
	两个flag：SYNC\_EXTRAS\_MANUAL，强制为手动同步；SYNC_EXTRAS\_EXPEDITED，立即开始同步。    
####关于Sync Adapter的总结  
必须要有用户自定义的ContentProvider，同步框架才能运行。因为同步框架需要指定android:contentAuthority属性，而该属性值必须从AndroidManifest.xml中provider标签下的android:authorities属性值中进行选择，那么，为有provider标签，必须要自定义ContentProvider了。关于需不需要自定义ContentProvider，文档中对此也进行了说明。若不需要自定义ContentProvider，而同步框架又需要，那么就定义Stub Content Provider吧。同步操作通过ContentResolver.requestSync()启动，也就是说，在判断需要同步后，你只需要调用ContentResolver.requestSync()，其它就交给同步框架处理了。
####Transmitting Network Data Using Volley  
Volley is not suitable for large download or streaming operations, since Volley holds all responses in memory during parsing. 大的数据下载请用DownloadManager.  
* Sending a Simple Request  
	
	   	  
  
	   
	  
	
 


#参考资料  
[Developer_guide_url]:[https://developer.android.com/guide/index.html]
[intent_resolution_url]:[http://developer.android.com/guide/components/intents-filters.html#Resolution]  
[apktool_url]:[http://ibotpeaches.github.io/Apktool/]

	