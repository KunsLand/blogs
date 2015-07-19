title: Scalable Event Multiplexing
date: 2015-07-19 19:08:49
tags: [Event Multiplexing, select, poll, epoll, kqueue]
---
# 事件多路复用
去年因为搞MindNetwork项目接触了Sigma.js，进而又接触了Node.js，当时对Node.js印象比较深刻的一点是它的事件驱动/非阻塞IO的特性，由于是首次接触到这么底层的描述，以为事件驱动是Node.js的一大发明，还以为所谓的事件循环就是简单的用一个端口连接代替多个端口连接。后来在实习面试过程中，得知所谓的事件驱动并不是框架层面的事，而是操作系统层面的事，Node.js只不过利用了这一特点。后来崔老师又在微信圈里分享了一篇技术分享文档【[服务器并发，“事件驱动”的本质](http://mp.weixin.qq.com/s?__biz=MzAxNzQ3MDAyNg==&mid=208324485&idx=1&sn=01104b6fa69069b435e9d1fa1e87c3d8&scene=5#rd)】，勾起了我对服务器端并发深层次技术的好奇。那篇微信分享有些地方翻译不是很通顺，所以我又在Google上搜了一些文档，基本上对如何提高服务器端并发有了深层次的理解。

<!-- more -->

# 参考
* curl的作者的理解：[poll vs select vs event-based](http://daniel.haxx.se/docs/poll-vs-select.html)
* 伯克利大学的相关文档：[Scalable Event Multiplexing: epoll vs. kqueue](https://www.eecs.berkeley.edu/~sangjin/2012/12/21/epoll-vs-kqueue.html)
* 一篇比较详实的技术比较附代码样例：[select / poll / epoll: practical difference for system architects](http://www.ulduzsoft.com/2014/01/select-poll-epoll-practical-difference-for-system-architects/)