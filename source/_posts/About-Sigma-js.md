title: 使用Sigma.js库在Web页面画网络图
date: 2014-11-15 20:28:34
tags: [Sigma.js, JavaScript, Network, Graph, Node.js]
---
#Sigma
__[Sigma]__是用来画_[图]_(_[Graph]_)的开源JavaScript库（[sigma-github源码]，[sigma历史版本]），用它可以很方便的在_Web页面_中画__网络图__。

<!-- more -->

### 自己Build最新版本
由于是开源的，不时有新的代码更新，这些更新一般是bug修复，但作者的发布周期较长，所以需要自己build导出sigma库，以保证不会出现各种奇奇怪怪的bug。[sigma-github源码]首页已经说了怎么build，步骤大致如下：
* 下载sigma源码（不会用git的可以直接下载[zip文件](https://github.com/jacomyal/sigma.js/archive/master.zip)并解压出sigma.js文件夹）:
```
$ git clone git@github.com:jacomyal/sigma.js.git
```
* 安装[node.js];
* 安装[gjslint](https://developers.google.com/closure/utilities/docs/linter_howto?hl=en)，步骤大致如下：
    1. 安装[python](https://www.python.org/downloads/)，最好下2.7版本;
    2. 安装[easy_install](https://pypi.python.org/pypi/setuptools#installation-instructions)(windows下直接下载并运行[ez_setup.py](https://bootstrap.pypa.io/ez_setup.py)，最后添加__python安装目录下的scripts路径__到系统Path环境变量);
    3. 运行如下命令:  
```
$ easy_install http://closure-linter.googlecode.com/files/closure_linter-latest.tar.gz  
```  
* 在sigma.js路径下执行如下命令进行build，最终的js文件会保存在`build`文件夹下：
```
$ npm install
$ npm run build
```
`build`文件夹最终包含一个`plugins`文件夹和两个js文件(`sigma.min.js`和`sigma.require.js`文件)，有用的只是`plugins`文件夹和`sigma.min.js`文件。

### 运行样例
源码包下的`examples`文件夹包含了很多例子，大部分例子直接点开就可以在浏览器中查看。少数涉及__使用脚本读取文件__的例子需要__运行本地服务器__(使用的是node.js平台框架，需要安装[node.js])，在`sigma.js`路径下执行如下命令（第一条命令如果运行过了就不必再运行），最后点开`http://localhost:8000/examples/` 即可：
```
$ npm install
$ npm start
```
### API文档
源码总体分4部分，分别为`graph`, `renderers`, `settings`和`controller`。`graph`是图的数据结构，定义了图的基本接口；`renderers`是渲染器，目前支持`webgl`, `canvas`和`svg`三种；`settings`是对图的显示设定，如节点大小、颜色、边的属性；`controller`将前面三者关联起来提供接口。更多详细内容请看官方文档：
* [官方简介](https://github.com/jacomyal/sigma.js/wiki);
* [graph接口](https://github.com/jacomyal/sigma.js/wiki/Graph-API)；
* [Renderers文档](https://github.com/jacomyal/sigma.js/wiki/Renderers)；
* [Settings文档](https://github.com/jacomyal/sigma.js/wiki/Settings);
* [Controller接口](https://github.com/jacomyal/sigma.js/wiki/Settings);
* 问答（[Q&A](https://github.com/jacomyal/sigma.js/issues)）
* [sigma历史版本]

### 可处理的数据格式
* `gexf`格式：实际为xml文件。该格式源于[Gephi]，它是开源的网络可视化与分析软件，以插件形式提供各式各样的基于图的算法；
* `json`格式：该格式用于`Web客户端`与`服务器端`的数据交互。

### Graph, Node和Edge数据结构
官方文档中没有包含对数据结构的介绍，以下源码中有关数据结构的代码（从[`plugins/sigma.parsers.gexf/gexf.parser.js`](https://github.com/jacomyal/sigma.js/blob/master/plugins/sigma.parsers.gexf/gexf-parser.js)中找的，`json`对应部分也有相似的内容）：
* Graph数据结构：
    * version: string
    * mode: string
    * defaultEdgeType: string
    * meta: object {lastmodifieddate , ...}
    * model: object {node, edge}
    * nodes: arry
    * edges: arry
* Node数据结构：  
	* id: string
    * label: string
    * viz: object {color, position, size, shape}
    * attributes: map
* Edge数据结构：  
    * id: string
    * type: string
    * label: string
    * source: string
    * target: string
    * weight: number
    * viz: object {color, shape, thickness}
    * atrributes: map

[Sigma]:http://sigma.org/
[图]:http://zh.wikipedia.org/wiki/%E5%9B%BE_(%E6%95%B0%E5%AD%A6)
[Graph]:http://en.wikipedia.org/wiki/Graph_(mathematics)
[sigma-github源码]:https://github.com/jacomyal/sigma.js
[sigma历史版本]:https://github.com/jacomyal/sigma.js/releases/
[node.js]:http://nodejs.org/download/
[Gephi]:http://gephi.github.io