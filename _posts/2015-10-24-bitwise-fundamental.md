---

layout: post
title: 'Fundamental Bitwise Operation 位操作基础简介'
date: '2015-10-25'
header-img: "img/post-bg-web.jpg"
tags:
     - bit
     - 位操作
     - 移位
     - 基础
author: 'Bill Quan'

---

## 汇编中的bitwise shift

数据的逻辑操作，用于汇编语言中。

汇编语言中的**逻辑右移(SHR shift logical right)是将各位依次右移指定位数，然后在左侧补0，算术右移(SAR shift arithmetic right)是将各位依次右移指定位数，然后在左侧用原符号位补齐。**

比如，在汇编语言中，对于算术右移，如果最高位为1，则补1，否则补0， 如将10000000算术右移7位，应该变成11111111，而逻辑右移7位，则不考虑符号位，变为00000001，这点就是算术右移和逻辑右移的区别。

在汇编中，可以用算术右移来进行有符号数据的除法。把一个数右移n位，相当于该数除以2的n次方。




## 算术移位arithmetic shift 

算术移位相当于乘除2的移位数次方。算术左移n位相当于乘以2^n，算术右移n位相当于除以2^n。

	0b1 << 2 结果为0b100
	-0b100 >> 2 结果为-0b1
	
**移位后补齐：**左移位右侧用0补齐，**负数右移位左侧用原符号位补齐，-1算术右移结果不变**

	-0b100 >> 5 结果为-0b1	
	bin(-0b1) 为 0xffffffff, 右移结果不变

编程语言中的 >> 及 << 操作都是指算术移位。逻辑移位使用的不多，java中有逻辑移位（又称无符号移位）>>> 及 <<< 。

##简单位运算

- 1. and运算

and运算通常用于二进制**取位**操作，例如一个数 and 1的结果就是取二进制的最末位。这可以用来**判断一个整数的奇偶**，二进制的最末位为0表示该数为偶数，最末位为1表示该数为奇数.

- 2. or运算 

or运算通常用于二进制特定位上的**无条件赋值**，例如一个数or 1的结果就是把二进制最末位强行变成1。如果需要把二进制最末位变成0，对这个数or 1之后再减一就可以了，其实际意义就是把这个数强行变成最接近的偶数。

- 3. xor运算 

xor运算通常用于对二进制的特定一位进行**取反操作**，因为异或可以这样定义：**0和1异或0都不变，异或1则取反。**

xor运算的逆运算是它本身，也就是说两次异或同一个数最后结果不变，即(a xor b) xor b = a。

xor运算可以用于简单的加密，比如我想对我MM说1314520，但怕别人知道，于是双方约定拿我的生日19000101作为密钥。1314520 xor 19000101 = 20665500，我就把20665500告诉MM。MM再次计算20665500 xor 19000101的值，得到1314520，于是她就明白了我的企图。

下面我们看另外一个东西。定义两个符号#和@（我怎么找不到那个圈里有个叉的字符），这两个符号互为逆运算，也就是说(x # y) @ y = x。现在依次执行下面三条命令，结果是什么？


	x <- x # y
	y <- x @ y
	x <- x @ y


执行了第一句后x变成了x # y。那么第二句实质就是y <- x # y @ y，由于#和@互为逆运算，那么此时的y变成了原来的x。第三句中x实际上被赋值为(x # y) @ x，如果#运算具有交换律，那么赋值后x就变成最初的y了。这三句话的结果是，x和y的位置互换了。

加法和减法互为逆运算，并且加法满足交换律。把#换成+，把@换成-，我们可以写出一个不需要临时变量的swap过程(Pascal)。

	procedure swap(var a,b:longint);
	begin
	   a:=a + b;
	   b:=a - b;
	   a:=a - b;
	end;


好了，刚才不是说xor的逆运算是它本身吗？于是我们就有了一个看起来非常诡异的swap过程：

	procedure swap(var a,b:longint);
	begin
	   a:=a xor b;
	   b:=a xor b;
	   a:=a xor b;
	end;



- 4. not运算

not运算的定义是把内存中的0和1全部取反。使用not运算时要格外小心，你需要注意整数类型有没有符号。如果not的对象是无符号整数（不能表示负数），那么得到的值就是它与该类型上界的差，因为无符号类型的数是用$0000到$FFFF依次表示的。下面的两个程序（仅语言不同）均返回65435。

	var
	   a:word;
	begin
	   a:=100;
	   a:=not a;
	   writeln(a);
	end.

<space>

	#include <stdio.h>
	int main()
	{
	    unsigned short a=100;
	    a = ~a;
	    printf( "%dn", a );    
	    return 0;
	}

如果not的对象是有符号的整数，情况就不一样了，稍后我们会在“整数类型的储存”小节中提到。



- 5. shl运算

a shl b就表示把a转为二进制后左移b位（在后面添b个0）。例如100的二进制为1100100，而110010000转成十进制是400，那么100 shl 2 = 400。可以看出，a shl b的值实际上就是a乘以2的b次方，因为在二进制数后添一个0就相当于该数乘以2。

通常认为a shl 1比a * 2更快，因为前者是更底层一些的操作。因此程序中乘以2的操作请尽量用左移一位来代替。

定义一些常量可能会用到shl运算。你可以方便地用1 shl 16 – 1来表示65535。很多算法和数据结构要求数据规模必须是2的幂，此时可以用shl来定义Max_N等常量。



- 6. shr运算

和shl相似，a shr b表示二进制右移b位（去掉末b位），相当于a除以2的b次方（取整）。我们也经常用shr 1来代替div 2，比如二分查找、堆的插入操作等等。想办法用shr代替除法运算可以使程序效率大大提高。最大公约数的二进制算法用移位操作来代替慢得出奇的mod运算，效率可以提高60%。

##位运算的简单应用

有时我们的程序需要一个规模不大的Hash表来记录状态。比如，做数独时我们需要27个Hash表来统计每一行、每一列和每一个小九宫格里已经有哪些数了。此时，我们可以用27个小于2^9的整数进行记录。例如，一个只填了2和5的小九宫格就用数字18表示（二进制为000010010），而某一行的状态为511则表示这一行已经填满。需要改变状态时我们不需要把这个数转成二进制修改后再转回去，而是直接进行位操作。在搜索时，把状态表示成整数可以更好地进行判重等操作。这道题是在搜索中使用位运算加速的经典例子。以后我们会看到更多的例子。

下面列举了一些常见的二进制位的变换操作。
	
	    功能              |           示例            |    位运算
	—————————————————————-+——————————————————————————+————————————————————————–
	去掉最后一位          | (101101->10110)           | x shr 1
	在最后加一个0         | (101101->1011010)         | x shl 1
	在最后加一个1         | (101101->1011011)         | x shl 1+1
	把最后一位变成1       | (101100->101101)          | x or 1
	把最后一位变成0       | (101101->101100)          | x or 1-1
	最后一位取反          | (101101->101100)          | x xor 1
	把右数第k位变成1      | (101001->101101,k=3)      | x or (1 shl (k-1))
	把右数第k位变成0      | (101101->101001,k=3)      | x and not (1 shl (k-1))
	右数第k位取反         | (101001->101101,k=3)      | x xor (1 shl (k-1))
	取末三位             | (1101101->101)            | x and 7
	取末k位              | (1101101->1101,k=5)       | x and (1 shl k-1)
	取右数第k位          | (1101101->1,k=4)          | x shr (k-1) and 1
	把末k位变成1         | (101001->101111,k=4)      | x or (1 shl k-1)
	末k位取反            | (101001->100110,k=4)      | x xor (1 shl k-1)
	把右边连续的1变成0    | (100101111->100100000)    | x and (x+1)
	把右起第一个0变成1    | (100101111->100111111)    | x or (x+1)
	把右边连续的0变成1    | (11011000->11011111)      | x or (x-1)
	取右边连续的1        | (100101111->1111)         | (x xor (x+1)) shr 1
	去掉右起第一个1的左边 | (100101000->1000)         | x and (x xor (x-1))

最后这一个在树状数组中会用到。

##整数类型的储存

我们前面所说的位运算都没有涉及负数，都假设这些运算是在unsigned/word类型（只能表示正数的整型）上进行操作。但计算机如何处理有正负符号的整数类型呢？下面两个程序都是考察16位整数的储存方式（只是语言不同）。

	var
	   a,b:integer;
	begin
	   a:=$0000;
	   b:=$0001;
	   write(a,' ',b,' ');
	   a:=$FFFE;
	   b:=$FFFF;
	   write(a,' ',b,' ');
	   a:=$7FFF;
	   b:=$8000;
	   writeln(a,' ',b);
	end.

<sp>

	#include <stdio.h>
	int main()
	{
	    short int a, b;
	    a = 0x0000;
	    b = 0x0001;
	    printf( "%d %d ", a, b );
	    a = 0xFFFE;
	    b = 0xFFFF;
	    printf( "%d %d ", a, b );
	    a = 0x7FFF;
	    b = 0x8000;
	    printf( "%d %dn", a, b );
	    return 0;
	}

两个程序的输出均为0 1 -2 -1 32767 -32768。其中前两个数是内存值最小的时候，中间两个数则是内存值最大的时候，最后输出的两个数是正数与负数的分界处。由此你可以清楚地看到计算机是如何储存一个整数的：计算机用$0000到$7FFF依次表示0到32767的数，剩下的$8000到$FFFF依次表示-32768到-1的数。32位有符号整数的储存方式也是类似的。稍加注意你会发现，二进制的第一位是用来表示正负号的，0表示正，1表示负。这里有一个问题：0本来既不是正数，也不是负数，但它占用了$0000的位置，因此有符号的整数类型范围中正数个数比负数少一个。**对一个有符号的数进行not运算后，最高位的变化将导致正负颠倒，并且数的绝对值会差1。**也就是说，not a实际上等于-a-1。这种整数储存方式叫做“补码”。

<br>

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。