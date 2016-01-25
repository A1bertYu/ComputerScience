#Computer Systems --a programmer's perspertive
second edition 英文版 机械工业出版社

##Chapter 2 Representing and Manipulating Information
###2.4 Floating Point
IEEE-754标准
####2.4.1 Fractional Binary Numbers  
书中用10进制的小数表示法，类比导出2进制的小数表示法。小数点之后的所有位，肯定是小于1的。
非十进制整数用二进制表示，有局限，只能表示十进制值为x * 2^y的那些数（十进制值为x1 * 2^y1 + x2 * 2^y2这些组合也可以表示），其它数只能近似表示，比如0.2（十进制）都无法精确表示。  

####2.4.2 IEEE Floating-Point Representation
IEEE-754中表示浮点数的形式为  
	
	V = (-1)^s * M * 2^E
s表示符号位，0为正1为负，占用1-bit；  
M表示小数部分（当然此处的小数部分不是说一定小于1），范围为[0,1)或[1,2)，占用n-bit；  
E表示指数部分，可正可负，占用k-bit。

单精度浮点数，s占用1-bit，E占用8-bit (k=8)， M占用23-bit (n=23)  
双精度浮点数，s占用1-bit，E占用11-bit (k=8)， M占用52-bit (n=23)

根据指数部分k-bit的取值，分3种情况：  

* case 1: Normalized Values  
	这是最常见的情况，指数部分的k-bit，既不是全0也不是全1的时候，对于E的求值，使用下面公式：  

		E = e - Bias = (unsigned value of k-bit) - (2^(k-1)-1)
这样，对于单精度浮点型，Bias=127， E的范围为(-126, 127);对于双精度浮点型，Bias=1023，E的范围为(-1022, 1023)；**注意, 此种情况下k-bit不能全0或全1，所以最后得到上述范围**  
对于M的求值，公式如下：

		M = 1 + f = 1 + 0.(n-bit) = 1.(n-bit)
也就是将n-bit全部在小数点右边，该结果最后就是整数位为1，小数位为n-bit的值。这种表示法书中称之为**an implied leading 1 representation**. 另外，书中说这种技巧能够不花费任何代价而得到额外的1-bit精度（getting an additional bit of precision），这是因为可以调整E的值，使得M取值为[1,2)。其实，这也是二进制相对于十进制特殊的地方，对于十进制，若采用这种科学计数法形式，那么整数位取值会是从1到9，而二进制只有可能为1，那么只有一种情况的话我就可以不表示，便节省了空间。

* case 2: Denormalized Values  
	此种情况对应指数部分的k-bit为全为0。求取E和M的公式如下：

		E = 1 - Bias
		M = f
	此种情况下，可以表示值为0的浮点数（符号位之外的bits全为0，可表示+0.0和-0.0），此外，还能表示接近0的值。
	相对于case 1，E没有采用公式E=-Bias，在当前情况下E的取值为-126（单精度）和-1022（双精度），而M没有采用公式M=1+f。之所以如此，是为了从case 1 到case 2的平滑过渡。case 1中的最小值是在E最小时取得的，而E最小时的值（case 1）为1 - Bias，而M最小时为1（case 1），在case 2中，最大值时M接近于1，这样，case 1的最小值接近case 2的最大值，如此，方可平滑过渡。

* case 3: Special Values  
	此种情况对应指数部分的k-bit全为1。当M的n-bit全为0时，对应无穷大，根据符号位，来确定是正无穷还是负无穷。而当M-bit不是全0时，则对应一个NaN（Not a Number）