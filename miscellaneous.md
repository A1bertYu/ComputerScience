printk和dmesg调试 valgrind  
编译器，构建，调试  
多线程  
数据库  
网络  
算法  
操作系统  
qt中的private slot的实现  
gameprogrammingpatterns.com  
https://en.wikipedia.org/wiki/Prolog  
https://en.wikipedia.org/wiki/Nomad_software  
https://en.wikipedia.org/wiki/OPS5  
http://ezlippi.com/blog/2014/12/c-open-project.html  
使用std::lower\_bound来查找map里面的数据会出现编译问题（当然，map有自己对应的lower\_bound函数）  

	PrsKey minUnitKey (key.getVaule(0), key.getVaule(1), MIN_ID);//a small enough unit id
    std::map<PrsKey, PrsRTUnitInfo*>::const_iterator itmin = 
        units_.lower_bound(minUnitKey);

DOM SAX XML JSON  
reinterpret_cast<char*> 与 (char\*)区别   
关于模板头文件的包含，在.h中声明了QVecot<int> & arg的类型，貌似可以不用在头文件中先行声明class QVector或者包含QVector，而只需要在cpp文件中包含QVector头文件即可。待验证  

关于一种代码的重复，例如OPC rtdb的初始化全站信号时，以及PrsModelInfo在getYc, getYx等的重复，是否有相关的解决方法？初步想到的是使用template  

connect(socket_, SIGNAL(readyRead ()), this, SLOT(readData()));若socket_=0时，connect是否可行？  
关于signal和slot参数的兼容性，比如void	error ( QAbstractSocket::SocketError socketError )，能否用整型来接收？  
若发送和接收分别在两个线程，而参数使用const QVector<QString> & 可以吗？也就是说跨线程传送栈上的参数。  

TimeIsHere构思  
需求：  
（1）若要查看某个项目的工作时间，可以实现，例如，OPC工具开发用了多久，分别哪些天。  

http://qwt.sourceforge.net/index.html  
https://www.linux-apps.com/browse/ord/top/
