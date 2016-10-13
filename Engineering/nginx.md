http://www.aosabook.org/en/nginx.html  
http://pl.atyp.us/content/tech/servers.html  
http://www.citi.umich.edu/projects/linux-scalability/reports/accept.html  
http://www.xmailserver.org/linux-patches/nio-improve.html   
关于同异步IO的问题   

##过滤模块  

* 过滤模块的调用顺序  
	所有HTTP过滤模块组成单链表，单链表的构成有点特殊，通过两个全局变量（类型分别是处理header和body的函数指针）来进行链接。注意，具体的处理函数是static的，只能在每一种过滤模块的.c文件中调用。在初始化的时候，通过这两个指针，将所有静态变量进行链接，在调用时，则只需调用这两个全局变量，这样，就依着链表，遍历的调用所有的过滤模块。【这里需要注意的是，static的函数只能在其所在的.c文件中被调用，但是，当在其.c文件中赋值给全局变量后，则突破了该限制】
