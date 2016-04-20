---
layout: post
title:  "C语言学习手记"
date:   2013-06-08 15:00:03
categories: c
supertype: career
type: other
---

>_这个手记记录我学习C语言的一些心得、重要知识点、容易忘记、混淆的知识点等，这并不是记录C的所有知识点。_

**1.**字符串切割函数，相当java里面的split。  

{% highlight c %}
#include <string.h>
char *strtok(str,delimiter)  
{% endhighlight %}

注意它的用法，第一次str传入完整的串，以后就传入NULL，因为strtok使用static来存数据，相当于缓存了str。  
注意到这点，以后用起来小心一点。使用例子在UnderstandingUnix的第四章4.15题  
另外，还有一个函数类似的也可以用,strsep,去man一下  

**2.**字符串中的子串定位函数

{% highlight c %}
#include <string.h>
char *strstr(char *haystack, char* needle);
char *strchr(char *s, int c);
char *strrchar(char *s, int c);
{% endhighlight %}

对于strstr，返回needle在haystack首次出现的地址，并且'\\0'是不会比较的。  
对于strchr，是查找单个字符首次出现的地址，而strrchr是返回最会出现的地址。

**3.**内存分配函数malloc(size), 扩充内存使用realloc(ptr,newsize);去man了解更多。

{% highlight c %}
#include <stdlib.h>
void *malloc(size_t size);
void *realloc(void *ptr, size_t newsize);
{% endhighlight %}

对于这两个内存分配函数，一定要记得使用free释放，不然就内存泄漏了。  
对于realloc，这是重新分配内存函数，如果ptr为NULL，则效果和malloc一样的。如果ptr不为NULL，而newsize为0，那么就等于free函数。  
realloc会先判断ptr指向的内存是否还有足够的连续空间，如果有及扩大，没有则新分配newsize内存，在旧的复制过去，如果newsize小则丢失数据。

**4.**内存区域的划分：栈，堆，全局区(静态static区)，文字常量区，程序代码区。

这是对每个进程的内存地址空间的划分，由编译器和OS共同完成，编译器可以指定栈的大小和堆的大小，在堆扩充的时候需要系统调用完成，栈和堆的增长方向相反的。栈是线性的，堆是链式的。线程有私有的栈，共享进程的全局变量，享有私有的局部变量。栈存放函数的参数值和局部变量等，有编译器释放。堆由程序员分配和释放。编译初始化的时候，固定了栈底和栈顶的地址，也就是固定了栈的大小；栈是从高地址向低地址扩展的；堆是从低地址向高地址扩展的，堆是链式式的，遍历方式是由低地址向高地址，堆的大小受限于操作系统的虚拟内存。

全局变量和静态变量的存储是放在一起的，初始化的全局变量和静态变量在一块区域，为初始化的全局变量和静态变量在相邻的另一块区域。程序结束后由系统释放。

常量字符串放在文字常量区。程序结束后由系统释放。

需要注意的是，在函数内部是局部变量，存放在栈上面，就算是字符数组也是存放在栈里面的，但是如果像下面那样

{% highlight c %}
void test()
{
    char *p = "zhangge";
    char g[] = "gege";
    static int c = 0;
    static int b;
}    
{% endhighlight %}

g的地址和内容都是在栈。p这个指针地址是存放在栈的，但是内容“zhangge”却存放在文字常量区。c存放在全局(静态)初始化区，b存放在全局(静态)为初始化区。栈的速度比堆快，就是说g比p的速度快。

看下面这个C语言的内存模型图：

![cstack3]

**5.**unsigned转换成signed时，直接复制原码到signed的低位，高位为0，即signed是正数，如果signed类型位数不够，只直接装载unsigned低位。  
signed转换层unsigned时，直接复制补码到低位，高位的符号位也复制,如果unsigned位数不够，只直接装载signed低位。

**6.**在if这些判断条件中，就是逻辑比较时，要转换成signed int来比较

**7.**signed char转signed int，即有符号小类型转大类型，符号位相应复制过去，如果本来是负数，符号位到第一个数字位都是1，因为这是补码。

**8.**signed char转unsigned int,要先转换成signed int。

**9.**头文件和实现文件问题：详细看收藏的一个[网址][网址]。  
list.h定义了结构体，宏，声明变量，函数声明等，在list.c中实现这些函数；在list.c中不一定要include list.h，如果list.c中的函数实现调用了一些list.h中声明的函数并且这些函数的实现在后面，则需要include list.h。至于list.h和list.c是否需要同名才能关联，那是不定的。  
两者无须同名，在链接阶段（预编译，汇编，机器语言，链接）会寻找函数的实现，函数名字是符号，重定位，操作系统上讲的。

**10.**size_t类型是unsigned long类型，所以要用printf打印的时候要用%lu

**11.**sizeof(变量)会得到这个变量所占的字节，跟sizeof(类型)一样，但是要注意两个地方，如果是数组的时候，例如char test\[10\],那么sizeof(char[10])==sizeof(test)<>sizeof(char),但是如果是指针就不行了，char \*test, 依然是sizeof(char \*)==sizeof(test),却得不到test的大小长度，就是不等于strlen(test)。

[网址]: http://dabentu.com/1280.html 
[cstack3]: /image/c_stack_3.gif
