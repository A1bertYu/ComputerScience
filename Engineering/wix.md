WiX is a set of tools that allows you to create Windows Installer-based deployment packages for your application. 
首先编写wxs文件，例如Sample.wxs，之后分两步：  
（1）使用candle.exe Sample.wxs，创建Sample.wixobj  
（2）使用light.exe Sample.wixobj，创建Sample.msi  
msi文件时一个安装数据库，以使Windows Installer进行正确的程序安装。因此，wxs文件中的内容并不一定按顺序执行，

it is a comfortable, XML-style way to describe your installation requirements that gets translated into Windows Installer .msi databases by its compiler and linker. WiX is a relatively thin wrapper around Windows Installer technology.  

对于每一个application，Windows通过GUID来对其进行识别，很多工具都能够产生GUID，可以使用uuidgen.exe来生成。VS也集成了生成工具。Wix需要使用十六进制且大写格式的GUID。  

首先，需要三个GUID， APP、安装包和升级时用到的GUID。对于安装包的GUID，可以要求Wix自动产生，每一个安装包其GUID都不一样。

软件版本格式major.minor.build，Windows Installer会忽略第四个域。xml文件可以使用UTF-8和ANSI两种格式，注意Product 和Package 标签的Language和Codepage/SummaryCodepage的属性。

