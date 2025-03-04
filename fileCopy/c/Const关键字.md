# const的使用

const变量能被其他文件extern引用吗？为什么？

先来看一段代码：

```
// 来源：公众号编程珠玑
// main.cc
#include<stdio.h>
// 引用外部定义的const_int变量
extern const int const_int;
int main()
{
    printf("const_int：%d\n",const_int);
    return 0;
}
// const.cc
// 定义const 变量
const int const_int  = 10;
```

编译链接：

```
 $ g++ -o main main.cc const.cc
 /tmp/ccWHAXxB.o: In function `main':
main.cc:(.text+0x6): undefined reference to `const_int'
collect2: error: ld returned 1 exit status
```

我们发现出现了链接问题，说const_int没有定义的引用，但我们确实在const.cc文件中定义了。

我们去掉const修饰符再编译一次，发现是可以的。从上面这个编译问题，就引出今天要讲的内容了。至于为什么会编译不过，最后再做解释。

当然你会发现，按照C代码去编译，是可以编译出来的。后面再解释。

## 链接属性

我们都知道，C/C++代码的编译通常经过预编译，汇编，编译，链接，通常会有变量会有三种链接属性：*外部链接，内部链接或无链接*。

**具有函数作用域，块作用域或者函数原型作用域的变量都是无链接变量**；具有文件作用域的变量可以是内部链接也可以是外部链接。**而外部链接变量可以在多个文件中使用，内部链接变量只能在一个编译单元中使用**（一个源代码文件和它包含的头文件）。

说了这么多，举个具体的例子：

```c
 // 来源：公众号【编程珠玑】
 // 作者：守望先生
 #include<stdio.h>
 int external_link = 10;  // 文件作用域，外部链接
 static internal_link = 20; // 文件作用域，内部链接
 int main()
 {
     int no_link = 30;   // 无链接
     printf("%d %d %d \n",external_link,internal_link,no_link);
     return 0;
 }
```

这里无链接变量还是比较好区分的，只要不是文件作用域的变量，基本是无链接属性。而文件作用域变量是内部链接还是外部链接呢？只要看前面是否有static修饰即可。当然对于C++，还要看是否有const修饰，后面我们再说。

## 如何知道某个变量是什么链接属性？

举个例子，在前面的代码中，我们按照C代码进行编译：

```
 $ gcc -c const.c 
 $ nm const.o |grep const_int
 0000000000000000 R const_int
```

从这里的结果可以看到const_int前面是R修饰的，
R：该符号位于只读数据区，READONLY的含义

而**该字母大写，其实也是表示它具有外部链接属性**。

再看看按照C++代码编译：

```
 $ g++ -c const.c
 $ nm const.o |grep const_int
 0000000000000000 r _ZL9const_int
```

可以看到，它的修饰符也是r，但是是小写的r，**小写字母表示该变量具有内部链接属性**。

nm命令非常实用，但本文不是重点。

## const关键字

说到const关键字，在《[const关键字到底该怎么用](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649284386&idx=1&sn=f2f6ad51cd026540f86759f3f93eb92c&chksm=f2f9ac45c58e25532915537153b0d5d0da74cc698058f9435a8ad02e111798e14f78ce6d188a&scene=21#wechat_redirect)》和《[C++中的const与C中的const有何差别？](http://mp.weixin.qq.com/s?__biz=MzI2OTA3NTk3Ng==&mid=2649285188&idx=1&sn=466f1615439b1c5be4321ad4ac6656ec&chksm=f2f99123c58e1835db31e73444e1457ebe9966112ef0b102426e7d4bc9f2a9fc598e861c5900&scene=21#wechat_redirect)》中已经分析过了，这里简单说一下，被const关键字修饰的变量，表明它是只读的，不希望被修改。

## extern关键字

extern关键字可以引用外部的定义，想必很多朋友已经很熟悉了，举个例子，如果把最开始的例子中的const关键字去掉，main.cc中的extern的意思，就是说有一个const_int变量，但是它在别的地方定义的，因此这里extern修饰一下，这样在链接阶段，它就会去其他的编译单元中找到它的定义，并链接。

当然，还有一个不太被关注的作用是，在C++中，它可以改变const变量的链接属性。

是的，在C++中，它改变了const_int的链接属性。我们可以修改const.c的内容如下：

```
 extern const int const_int  = 10;
```

然后再查看一下：

```
  $ nm const.o |grep const_int
 0000000000000000 R const_int
```

发现没有，它前面的修饰变成大写的R了，所以这个时候，你再编译，就能编译过，而不会报错了，对于C，它本来就是外部链接属性，所以根本不会报错。

## 解疑

所以，链接报错的通常问题就是找不到定义，原因无非就是：

- 未定义
- 在其他地方定义了，但是不具备外部链接属性
- 定义了，具备外部链接属性，但是链接顺序有问题

由于在C++中，被const修饰的变量默认为内部链接属性，因为链接会找不到定义。

