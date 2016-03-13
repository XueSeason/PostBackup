title: 拥抱函数式编程-RxSwift-0
date: 2016-01-17 13:51:27
tags: Swift
---

最近开始研究FRP，即函数响应式编程，发现里面的思想有种颠覆过去传统的编程思维。入门 RxSwift 并不平滑，而且除了官方文档，缺乏优秀的资料，所以我只将我学习 RxSwift 的过程记录下来，希望可以帮助大家。  

学习响应式，最难的应该是它的思想，我们不能一贯地按照以往命令式和状态式来思考问题。我们要时刻告诉自己的大脑，如何用响应式的思考问题。  

# What the fuck is FRP?

对，什么是操蛋的 FRP 呢？我们看下维基百科有这么一段话：

** Functional reactive programming (FRP) is a programming paradigm for reactive programming (asynchronous dataflow programming) using the building blocks of functional programming (e.g. map, reduce, filter). FRP has been used for programming graphical user interfaces (GUIs), robotics, and music, aiming to simplify these problems by explicitly modeling time. **

WTF！这都是个什么鬼。异步数据流、GUI、各种乱七八糟的名词堆一起。我选择死亡。。。  

好，吐槽完，我们该静下来思考下平时遇到的一些问题。  

一个想起来很简单实现起来不那么简单的需求，有一个按钮，我们需要知道被点击的次数（一定的时间间隔内）。按照以往命令式编程的习惯，代码量会比较繁琐。换一种思考方式，我们来看下面这张图是如何展示响应式编程的思想：  
![](http://ww2.sinaimg.cn/large/aa0fbcc4gw1f02h0pln50j20e80jzdho.jpg)

关键代码只有四行。[地址](http://jsfiddle.net/staltz/4gGgs/27/)  

图中，首先根据250毫秒的间隔时间 buffer 函数把点击事件流隔开，再通过 map 函数计算每一段事件的次数总和，最后通过 filter 函数过滤出次数大于 2 的事件流。接下来我们就可以订阅这个事件来执行我们想要的操作。  

感受下，有没有体会到 FRP 的那么一丝丝快感？

（未完待续）