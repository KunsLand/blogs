title: Spring Struts Hibernate
date: 2015-09-05 14:26:38
tags: [Java, Spring, Struts, Hibernate, IoC, AOP, MVC, ORM]
---
最近遇到一个比较棘手的有关Web服务器的性能问题，概括起来就是大规模并发数据库异步查询请求的高效性问题。由于项目时间有限，所以采取缓存查询结果的方式提高响应速度，但是仍然不能解决首次查询所带来的时间消耗。倘若某个耗时查询正在执行中，还存在新的相同请求该如何处理的问题。重复造轮子并不是很好的解决办法，看看Java中是如何解决这些问题的，这里以流行的Spring Struts Hibernate(SSH)为例。

<!-- more -->

### Overview
SSH是一套非常流行的Java Web集成开发框架，注意是一套集成框架，而非一个框架，包含三个框架，搭配使用以开发高并发、高性能的企业级Web应用。

### Spring
* 定义：Java平台的应用框架和反转控制容器（参见[维基](https://en.wikipedia.org/wiki/Spring_Framework)）。
* 范围：任何Java应用（不限于Web应用）。
* 核心一：反转控制（Inverse of Controll, IoC）/依赖注入（Dependency Injection, DI），（参见[Martin Fowler](http://martinfowler.com/articles/injection.html)）。
* 核心二：面向切面编程（Aspect Oriented Programming, AOP）。
* 文档：参见[Spring Framework Reference Documentation](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/)。

### Struts
* 定义：Apache旗下的、免费的、开源的Java Web应用开发MVC框架。
* 范围：Java Web应用。
* 核心：HTTP, Java Application Frameworks, JavaScript, AJAX, SOAP, Servlets, JSP, XML, JAAS, ...
* 文档：参见[Apache Struts](https://struts.apache.org)

### Hibernate
这才是我要找的。
* 定义：针对Java语言的对象关系映射（Object-Relational Mapping, ORM）框架。
* 范围：Java与数据库。
* 文档：参见[Hibernate. Everything data.](http://hibernate.org/)。

### Java Reflection
这是Java非常重要的特性，它在一定程度上能突破Java的安全机制，使得程序能够动态加载组件（或类），检查、审查并且操控程序的内部属性，广泛应用于JavaBean相关（JSP, DI, AOP）的设计开发中。