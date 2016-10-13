##基本概念  

* Chunks  
Each piece of code that Lua executes, such as a file or a single line in interac-
tive mode, is called a chunk.  
在交互模式，我们是运行的是一个独立的Lua解释器。在该模式下，Lua一般是将每一行当作一个完整的Chunk，当然，若一行没有输入完毕，比如do end这种语句，则等待其输入完成后（多行），才可算一个完整的Chunk

* Blocks  
A block is the body of a control structure, the body of a
function, or a chunk (the file or string where the variable is declared)  

* return  
	return必须要作为语句块（block）中的最后一条语句，也就是说return之后不能有其它语句，除了end, else和until。有时候在调试时需要在block中部加入return，这时候可以使用do return end的形式。  
* goto  
	label的定义，lable名称前后都有两个冒号。label的可见性与变量的可见性规则一致。labels are considered void statements