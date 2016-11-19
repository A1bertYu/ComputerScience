##2.6 Core Command-line Options

###2.6.1 Core Command-line Options
--tool=<toolname> [default: memcheck]  

###2.6.2. Basic Options
* --trace-children=<yes|no> [default: no]  
	该选项适用于多进程程序，当通过系统调用exec()来产生子进程时，若需要valgrind对子进程进行调试，则将该选项打开。  
	注意，对于多线程程序（系统调用fork()），valgrind是直接进行跟踪的，也就是子线程没有开关来启停valgrind的跟踪。  

* --trace-children-skip=patt1,patt2,...  
	对于部分子进程，可以跳过（不trace into）；值为进程名称，可以使用匹配符，可以指定多个进程名称，用','分隔。当某个进程被跳过后，在该进程中，其子进程也不会被跟踪。  

* --trace-children-skip-by-arg=patt1,patt2,...  
	也是跳过子进程，不过值为子进程的参数，也就是使用这些参数的进程，跳过  

* --child-silent-after-fork=<yes|no> [default: no]  
	当该选项使能后，Valgrind将不会输出通过fork()产生的线程的debug信息。  

* --vgdb=<no|yes|full> [default: yes]  
	当该选项的值为yes或者full时，可以使用外部的GDB来调试程序（运行于Valgrind），full可以提供更精确的断点设置。Valgrind提供了gdbserver的功能。

* --vgdb-error=<number> [default: 999999999]  
	在暂停程序运行和等待GDB连接之前，tools需要先行等待--vgdb-error数量的错误发生。也就是说在该选项指定的错误数量发生之后，Valgrind gdbserver会在之后的每一次错误发生时被调用。

* --vgdb-stop-at=<set> [default: none]  
	除了--vgdb-error可以控制Valgrind gdbserver外，有其它三种事件也能控制，startup,exit和valgrindabexit.这三个事件效果分别如下：  
	（1）startup 程序运行之前，gdbserver会被调用；  
	（2）exit 在程序的最后一条语句运行之后，gdbserver会被调用  
	（3）valgrindabexit Valgrind异常退出时会被调用  

* --track-fds=<yes|no> [default: no]  
	用于输出已打开的文件信息。对每一个fd，输出其open过程的信息（a stack backtrace），以及其它的详细信息（如名称）  

* --time-stamp=<yes|no> [default: no]  
	对于输出的消息，会带有从启动开始计时的用时信息。

* --log-fd=<number> [default: 2, stderr]  
	用于指定输出的目的fd，注意，若Valgrind在运行时（若使用stderr），用户使用的其它程序也向stderr输出，则二者可能会交叉。  

* --log-file=<filename>  
	用于指定输出的目的文件名。若名称为空，则Valgrind会abort；文件名的指定形式有如下三种：  
	（1）'%p'，名称中的该部分将会被替换为当前进程的ID，这对多进程的调试输出是很有用的。  
	（2）'%q{FOO}'，将使用环境变量FOO的值替换该部分，  
	（3）'%%'用于表示'%'。注意'%'之后的字符，除了前述3种，其它都不合法，会导致abort  

* --log-socket=<ip-address:port-number>  
	用于指定输出的目的socket，port默认值为1500，若无法连接，则Valgrind输出到stderr，该选项主要用于配合valgrind-listener使用。  
###2.6.3. Error-related Options
对于需要输出错误信息的工具，可能有如下的选项。  

* --xml=<yes|no> [default: no]  
	重要的输出信息将以xml格式输出，非重要信息以plain text格式输出  
* --xml-fd=<number\> [default: -1, disabled]  
	指定输出的目的fd，xml格式  
* --xml-file=<filename\>  
	指定输出的目的文件名，命名规则可以参考--log-file选项  
* --xml-socket=<ip-address:port-number\>  
	参考--log-socket选项  
* \-\-xml\-user\-comment=<string\>  
	在xml文件的开头增加信息  
* --demangle=<yes|no> [default: yes]  
	Enable/disable automatic demangling (decoding) of C++ names.The demangler handles symbols
	mangled by g++ versions 2.X, 3.X and 4.X.   
	在Suppressing errors的文件中，并不会使用demangle，因为会影响效率。  
* --num-callers=<number\> [default: 12]  
	跟踪的调用深度，最大值500，值越大，消耗内存越多，用时越长。  
* --unw-stack-scan-thresh=<number\> [default: 0] , --unw-stack-scan-frames=<number>[default: 5]  
	只支持ARM平台。在常规的栈展开机制（使用Dwarf CFI records和frame-pointer following）失效时，stack scanning可能能恢复栈的跟踪信息。注意的是，这种信息不一定精确，且当前实现方式有诸多限制，具体参考Manual。后一个参数值不能超过--num-callers选项值。  
* --error-limit=<yes|no> [default: yes]  
	当为yes时，Valgrind在遭遇1000个不同错误或者10,000,000个错误时，会停止。  
* --error-exitcode=<number\> [default: 0]  
	当为0时，返回值为target program的返回值；当为非0值时，在Valgrind检测到错误时，返回的就是该非0值。  
* --error-markers=<begin\>,<end\> [default: none]  
	当输出为plain text时，对于每一个错误，前面加上<begin\>指定的字符串，后面加上<end\>指定的字符串，这样方便对于错误查找。因为除了错误信息，还会有程序输出的其它信息。  
* --sigill-diagnostics=<yes|no> [default: yes]  
	Enable/disable printing of illegal instruction diagnostics. Whenever an instruction is encountered that Valgrind cannot decode or translate, before the program is given a SIGILL signal.对于有些程序，有意trap the SIGILL signal，可以通过该选项避免这种信息输出。  
* --show-below-main=<yes|no> [default: no]  
	对于main-like函数的调用，因为更多时候涉及的非用户代码，所以可以无需跟踪。<font color='red'>关于main-like的函数，例如glibc’s \_\_libc\_start\_main，还需要了解，什么是main\-like？</font>
* --max-threads=<number\> [default: 500]  
	顾名思义   
* --fullpath-after=<string\> [default: don’t show source paths]  
	对于工程较大时，文件较多时调试较为有用；可以指定多次  
* --extra-debuginfo-path=<path\> [default: undefined and unused]  
	只能指定一次，绝对路径，用于搜寻调试信息。  
* --debuginfo-server=ipaddr:port [default: undefined and unused]  
	用于获取非本机的调试信息文件  
* --allow-mismatched-debuginfo=no|yes [no]  
	用于对调试文件的校验  
* --suppressions=<filename> [default: $PREFIX/lib/valgrind/default.supp]  
	最多可以指定100个supp文件  
* --gen-suppressions=<yes|no|all> [default: no]  
	相当于显式设置supp的条件。对C++调试来说，很为有用，因为其会显示mangled names，而我们在编写supp文件时，是会用到的。  
* --input-fd=<number\> [default: 0, stdin]  
	Valgrind在需要输入时，从该fd读取输入，默认为stdin  
* --dsymutil=no|yes [yes]  
	适用于OS X，该系统中，debug information并没有被链接到lib或者可执行文件中，必须要手动通过dsymutil进行链接， 
* --max-stackframe=<number\> [default: 2000000]  
	一般不会用到，In general, allocating large structures on the stack is a bad idea, 编码时注意就行。  
* --main-stacksize=<number\> [default: use current ’ulimit’ value]  
	Specifies the size of the main thread’s stack.  

###2.6.4. malloc-related Options
当工具需要使用自定义的malloc()时，需要设置如下选项  

* --alignment=<number\>  
	2的幂，不超过4096，不小于默认值（8或者16）  
* --redzone-size=<number\> [default: depends on the tool]  
	用于设置padding  
###2.6.5. Uncommon Options  

* --smc-check=<none|stack|all|all-non-file> [default: all-non-file for x86/amd64/s390x, stack for other archs]  
	该选项用于Valgrind对于自修改代码的检测（<font color='red'>what is self-modifying code</font>）  

* --read-inline-info=<yes|no> [default: see below]   
	读取DWARF3信息中被调用的内联函数。	
* --read-var-info=<yes|no> [default: no]  
	读取DWARF3信息中的变量类型和位置（源码中的位置）。
* --vgdb-poll=<number> [default: 5000]  
	<font color='red'>具体用途待核实</font>
* --vgdb-shadow-registers=no|yes [default: no]  
	与GDB配合使用，<font color='red'>what is shadow registers</font>  
* --vgdb-prefix=<prefix> [default: /tmp/vgdb-pipe]  
	为了与gdb/vgdb进行通信，Valgrind gdbserver创建了3个文件（2 named FIFOs and a mmap shared memory file），该选项则用于指定路径和文件名前缀。  
* --run-libc-freeres=<yes|no> [default: yes]  
	仅用于linux，