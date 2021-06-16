## 例子分析

a.c

```c
int a;//弱符号
void fun()//强符号
{
	a = 20;//往a的内存上写20，写4个字节 编译阶段就完成
}
```

main.c

```c
#include<stdio.h>
short a = 10;//强符号 14 00 00 00最终将b覆盖为0
short b = 10;
int main()
{
	fun();//链接完成选择强符号
	printf("a = %d, b = %d\n", a, b);//输出20，0
	return 0;
}
```
- **为何b输出0？**

因为源文件是**独立进行编译**的，编译 a.c 的时候 gcc 根本看不到其它文件。a.c 里面声明 int 那就当成 int 写 4 字节。

根据汇编代码分析：

![2021011201](/2021011201.png)

首先汇编代码中：

```c
 64e:	c7 05 b8 09 20 00 14 	movl   $0x14,0x2009b8(%rip)        # 201010 <x>
 655:	00 00 00 
 658:	90                   	nop
```

其中第一条**因指令太长被折**到第二行显示（误区：认为第二行是nop指令），整理得：

```c
 64e:	c7 05 b8 09 20 00 14 00 00 00 	movl   $0x14,0x2009b8(%rip)        # 201010 <x>
 658:	90                            	nop
```

要知道，执行一条语句时， rip 指向下一条(**rip存的就是PC值**)，所以 0x2009b8(%rip) == 0x2009b8 + 0x658 == 0x201010，这就是第一条语句结尾 201010 的来由。

也就是说它往 0x201010 处存入了“14 00 00 00”四个字节。

接下来：

```c
 669:	0f b7 05 a2 09 20 00 	movzwl 0x2009a2(%rip),%eax        # 201012 <y>
 670:	0f bf d0             	movswl %ax,%edx
 673:	0f b7 05 96 09 20 00 	movzwl 0x200996(%rip),%eax        # 201010 <x>
 67a:	98                   	cwtl
```

前两句是从 0x201012 处取出两字节（取到“00 00”），先存ax再扩展符号到双字存入 edx 中，这是把short型变量b作printf的参数。

后两句是从 0x201010 处取出两字节（取到“14 00”），先存ax再扩展符号到双字存入 eax 中，这是把short型变量a作printf的参数。

回顾一下，会发现前面存入四字节时，覆盖了后面两个变量的原有值。

## 总结

链接器的输入是一组可重定位的目标模块，每个模块定义一组符号，有些是局部的（只对定义该符号的模块可见），有些是全局的（对其他模块也可见）。

在编译时，编译器向汇编器输出每个全局符号，或者是强或者是弱，而汇编器把这个信息隐含地编码在可重定位目标文件的符号表里。
**函数和已初始化的全局变量是强符号，未初始化的全局变量是弱符号。**

Linux链接器使用下面规则来处理多重定义的符号：

1. 不允许有多个同名的强符号。

2. 如果有一个强符号和多个弱符号同名，那么选择强符号。

3. 如果有多个弱符号同名，那么从这些弱符号中任意选择一个。（一般选占用空间大的）

## 参考文献
1. Randal E.Bryant, David O’Hallaron. [深入理解计算机系统(第2版). 机械工业出版社, 2010](https://book.douban.com/subject/5333562/)