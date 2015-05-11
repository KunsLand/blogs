title: Java字符编码与Unicode字符集
date: 2015-03-11 21:30:12
tags: [Java,Unicode]
---
昨天某人问我Java中如下代码的输出结果为什么不一样（"A"为1，"一"为2）？
```
System.out.println("A".getBytes().length+"\t"+"一".getBytes().length);
1	2
```
咋一看，好像是不对劲，Java中的字符不都是Unicode编码（16bit，2字节）的吗？怎么英文字母只占1字节呢？于是立马查看《Java编程思想（Thinking in Java）第四版》，关于常用类型的基本信息表（见“第二章-一切都是对象”，“2.2.2-特例：基本类型”，第23页）里赫然写着char的大小为16bit的Unicode。于是断定问题出在getBytes函数上。

<!-- more -->

### Java中String的getBytes方法
Java 7中关于String的[getBytes()](http://docs.oracle.com/javase/7/docs/api/java/lang/String.html#getBytes()方法，括号中不指定任何编码格式，该方法将自动使用平台（系统）默认的编码格式。倘若指定编码格式，例如常见的[GBK](http://en.wikipedia.org/wiki/GBK)，[GB2312](http://en.wikipedia.org/wiki/GB_2312)，[UTF-8](http://en.wikipedia.org/wiki/UTF-8)，将会出现下面的结果：
```
System.out.println("A".getBytes("GBK").length+"\t"+"一".getBytes("GBK").length);
System.out.println("A".getBytes("GB2312").length+"\t"+"一".getBytes("GB2312").length);
System.out.println("A".getBytes("UTF-8").length+"\t"+"一".getBytes("UTF-8").length);
1	2
1	2
1	3
```
很明显，系统默认的编码是GBK或GB2312，而不是UTF-8，更不是统统占2字节Unicode。那么Java中的Unicode是怎么样的呢？
Java中如果要让getBytes函数按照占两个字节的Unicode进行编码，可以按照如下方式强制执行：
```
System.out.println("A".getBytes("Unicode").length+"\t"+"一".getBytes("Unicode").length);
4	4
```
比较意外的是输出结果是4而非2，这又是为什么呢？难道Java中的Unicode是4个字节编码的？
先别急，我们再多测几个例子：
```
System.out.println("AB".getBytes("Unicode").length+"\t"+"一二".getBytes("Unicode").length);
System.out.println("ABC".getBytes("Unicode").length+"\t"+"一二三".getBytes("Unicode").length);
System.out.println("ABCD".getBytes("Unicode").length+"\t"+"一二三四".getBytes("Unicode").length);
System.out.println("".getBytes("Unicode").length);
6	6
8	8
10	10
0
```
4，6，8，10，……等差数列！f(n)=2*n+2?!(n>=1),f(0)=0
那么问题来了，多出来的两个字节是干什么的呢？
### 关于Unicode、UTF-8、UTF-16
网上搜罗一大筐后，综合[阮一峰](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)和[殇雲](http://blog.csdn.net/tianjf0514/article/details/7854624)的博客、[维基百科UTF-8](http://zh.wikipedia.org/wiki/UTF-8)、[维基百科Unicode](http://zh.wikipedia.org/wiki/Unicode)、[维基百科UTF-16](http://zh.wikipedia.org/wiki/UTF-16)，终于理清了Unicode与UTF-8、UTF-16之间的关系，也对编码有了更深的理解。
编码的目的是要实现将人类所使用的文字（字母、象形文字等）转换成计算机所能理解的二进制，__编码的过程则分两个步骤__：
* __文字->数字__：搜罗世界各国的语言文字，给每个文字进行编号，每个文字对应一个唯一确定的十进制数，统一或者规范每个文字占用的字节数；这里涉及到的是__国际标准和规范__的问题；
* __数字->二进制__：将数字编号转换成最终的二进制序列（由0和1组成的序列），并按照特定的方式存入计算机存储结构（内存、硬盘等）；这里涉及到的是对国际标准和规范的__具体算法实现__；

Unicode实际上只能算是国际标准，不能说是编码方案。UTF-8和UTF-16都是对Unicode标准的具体实现方案，区别在于UTF-8是不定长的（最多4字节），UTF-16是定长的2字节。两个字节就存在高低位之分（高位字节放前/大端/BigEndian和低位字节放前/小端/LittleEndian），针对不同的二进制存放方式，__UTF-16编码方案将在字符串前端的额外添加两个字节中指定字节的存放方式（FE FF对应大端，FF FE对应小端）__。上面的Java实验中，UTF-8对英文编码时就只有1个字节长，而对中文编码时就有3个字节长。而Java中实现Unicode标准时采用的是UTF-16BE（大端）方案，因此不管是英文还是中文，最终的字节数为__字符数的两倍+2__，这就是为什么上面最后一个实验多出两个字节的原因。