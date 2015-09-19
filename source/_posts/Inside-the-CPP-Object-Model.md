title: Inside the C++ Object Model
date: 2015-09-05 19:38:39
tags: [CPP]
---
经常遇到有关C++的奇怪问题，而且常常不是什么难题，但是就是觉得一团糟。偶然接触到C++对象模型的概念，挖掘出一本经典的书籍[Inside the C++ Object Model](http://read.pudn.com/downloads120/ebook/511418/inside.the.c%2B%2B.object.model.pdf)，虽然年代久远，书中也有多处明显错误，但瑕不掩瑜，仔细品读，还是有茅塞顿开的感觉。参考[shifan](http://lifegoo.pluskid.org/upload/doc/object_models/C++%20Object%20Model.pdf)的总结，我将部分关键的图片罗列出来，以便以后查阅。

<!-- more -->

### Data Layout
一般视图：
![Plain](/blogs/img/plain.PNG)

带字节对齐的视图：
![Alignment](/blogs/img/alignment.PNG)

继承视图：
![Inheritance](/blogs/img/inheritance.PNG)

组合视图：
![Object in object](/blogs/img/inheritance-2.PNG)

带虚函数的一般视图：
![Common implementation I](/blogs/img/impl-common-data.PNG)

另一种带虚函数的一般试图：
![Common implementation II](/blogs/img/impl-common-data-2.PNG)

g++视图：
![Linux g++ implementation](/blogs/img/impl-linux-gplusplus-data.PNG)

多虚继承：
![Chaotic](/blogs/img/chaotic-evil.PNG)

### Virtual/Static Binding
* __虚绑定(Virtual Binding)__：指向某一对象的指针或引用调用虚函数就是虚绑定。
* __静态绑定(Static Binding)__：指向某一对象的指针或引用，或是对象本身，调用任何非虚函数就是静态绑定。

### Type info
MSVC视图：
![MSVC implementation](/blogs/img/impl-msvc.PNG)
Linux g++ 视图：
![Linux g++ implementation](/blogs/img/impl-linux-gplusplus.PNG)

### 构造顺序
虚基类->基类->虚指针(vptr)->不再初始化列表中的对象->初始化列表中的对象->构造器。

* 虚指针(vptr)沿着继承树不断被替换。
* 不论是静态绑定还是动态绑定，虚函数都会在构造完成前失去虚特性（virtuousness）。
