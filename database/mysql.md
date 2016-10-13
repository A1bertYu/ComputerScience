#Official Manual  

##Chapter 1 General Information

####MySQL Differences from Standard SQL  

* Foreign Key Differences  
	

##Tutorial  

* 连接数据库  
shell> mysql -h host -u user -p  
注：server在本机时，可以省略-h参数

* 查询  
	mysql下，语句需要以‘;’结尾，当然，有些不需要，比如quit。  
	在一行中若键入多条查询语句，可以以";"相隔。  
	mysql是在获取到";"才开始启动查询，因此，可以多行键入一条查询语句，直到分号，在此之前，若想取消，键入“\c”即可。  
	在mysql中，字符串可以使用单引号或者双引号表示。  
	在where语句中，字符串的比较是大小写不敏感的。

* 创建数据库  
	在unix下，数据库名称和表名称是大小写敏感的。  

* Loading Data into a Table  
	在INSERT时，String and date values are specified as quoted strings here.   
	LOAD DATA命令可以从文件中读取值并存储到数据库

##Globalization  
###Character Set Support
A character set is a set of symbols and encodings. A collation is a set of rules for comparing characters in a character set. 
关于字符集的示例，


##The InnoDB Storage Engine  
