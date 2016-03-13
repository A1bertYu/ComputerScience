* 构造函数  
	对于某类，用其构造函数A调用构造函数B，即使B中对某数据成员（非静态）进行了初始化，但最终还是无效的。  
* QT中的数据库驱动  
	在Windows 7环境下，使用了QT的MySQL驱动，那么必须在程序所在目录创建plugins/sqldrivers目录，里面放置驱动才可以正常使用MySQL。当然，通过QCoreApplication::addLibraryPath ( const QString & path )可以添加更多路径，但是，要在路径下创建文件夹sqldrivers并放置相应驱动才行（必须要有这个sqldrivers名称的文件夹）  