###Common Gateway Interface  
**CGI定义**：Common Gateway Interface (CGI) offers a standard protocol for web servers to interface with executable programs running on a server that generate web pages dynamically.CGI一般使用脚本语言编写，当然也能用任意的非脚本语言。  
对于Web Server，其运行HTTP server software。通常，HTTP server有一个用于存放静态文件的文件夹。对于动态文件，则需要在请求时生成，一般是通过执行某个文件夹里的脚本或二进制可执行文件，而这个文件夹通常称为CGI文件夹（比如命名为cgi-bin），the HTTP server通过运行CGI里的文件，产生结果给客户端。  

**传统CGI的缺点**： 需要在server上创建新的process，若CGI是脚本实现，则可能还需要编译，这使得成本很高。  
**CGI的其它实现形式**：  
（1）直接在HTTP Server内部模块运行（通过第三方模块实现，例如插件）  
（2）SCGI，web app和http server之间通信的protocol。Web Server（例如Http Server）为SGCI客户端，web app则为SGCI服务器端，客户端通过传递字节（8-bit）流来将请求传递到服务器端，服务器的响应也是字节流的形式。（进程间通信的串行数据）   
（3）FastCGI，类似于一个守护进程（a single, long-running process），省却了不断创建进程的开销，同时可独立于Http Server。  
（4）改变产生动态网页的进制（Replacement of the architecture for dynamic websites can also be used.）。Java servlet container.   

* uWSGI  
	uWSGI用于连接Nginx和Django的一种部署方式：  
	the web client <-> the web server <-> the socket <-> uwsgi <-> Django  
	在上述的部署方式中，uWSGI与Server之间通过socket通信，采用WSGI协议。
