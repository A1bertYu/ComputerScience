#Markdown学习笔记

##区块元素

通过使用无序列表，来介绍一些基本元素的使用。'*','+'和'-'，作为列表标记符号。

* 段落

    段落前后需要有一行或者多行的空行。空行的定义是显示上看起来像是空的，便会被视为空行。比方说，若某一行只包含空格和制表符，则该行也会被视为空行。普通段落不该用空格或制表符来缩进。

* 有序列表

    以数字接一个英文句点，作为标记符号。
    >1. hello world
    >2. DreamFruit Inc.

* 区块引用

    >1. 基本语法  
    在每行的最前面加上'>'符号,便构成了区块引用。Markdown 也允许你偷懒只在整个段落的第一行最前面加上 >。
	
	>2. 区块引用的嵌套  
    引用的区块内也可以使用其他的 Markdown 语法，包括标题、列表、代码区块等
		>>引用内的引用，根据层次使用对应数量的>>

* 代码区块

	要在 Markdown 中建立代码区块很简单，只要简单地缩进 4 个空格或是 1 个制表符就可以。一个代码区块会一直持续到没有缩进的那一行（或是文件结尾）  
	Here is an example of C:

    	if a > 0
        	return;
 
* 分割线

	你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。
	***
##区段元素

1. 链接
	Markdown 支持两种形式的链接语法： 行内式和参考式两种形式。不管是哪一种，链接文字都是用 [方括号] 来标记。  
   	This is [an example](http://dreamfruit.cn "DreamFruit") inline link.  
	This is [an example][url1] reference-style link.  
	隐式链接标记功能让你可以省略指定链接标记，如：  
	Visit [Dream Fruit][] for more information.  
	自动链接：<http://example.com/>   
	<address@example.com>

2. 强调

	Markdown 使用星号（*）和底线（_）作为标记强调字词的符号，被 * 或 _ 包围的字词会被转成用 < em> 标签包围，用两个 * 或 _ 包起来的话，则会被转成 < strong>注意，html标签有意写错，类似转义  
	*single asterisks*  
	**double asterisks**  
3. 代码

	Use the `printf()` function.  
	``There is a literal backtick (`) here.``  
	 single backtick in a code span: `` ` ``  
	A backtick-delimited string in a code span: `` `foo` ``

4. 图片

	Markdown 使用一种和链接很相似的语法来标记图片，同样也允许两种样式： 行内式和参考式。  
 	![Alt text](/jennifer.jpg)
	![Alt text][tencent]
##参考文献

[url1]:"http://www.dreamfruit.com.cn"
[Dream Fruit]: http://www.dreamfruit.com.cn
[tencent]: /tencent.jpg
<br/>2015&copy;DreamFruit Inc.