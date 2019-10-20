---
title: 深入理解Node.js 中的进程与线程
tags:
  - Nodejs
originContent: ''
categories:
  - 文章
toc: false
date: 2019-09-05 21:26:58
---

![image.png](/images/2019/09/05/0d781bf0-cfe0-11e9-a152-33ef044f5d50.png)

# 前言

进程与线程是一个程序员的必知概念，面试经常被问及，但是一些文章内容只是讲讲理论知识，可能一些小伙伴并没有真的理解，在实际开发中应用也比较少。本篇文章除了介绍概念，通过Node.js 的角度讲解进程与线程，并且讲解一些在项目中的实战的应用，让你不仅能迎战面试官还可以在实战中完美应用。

# 面试会问

> Node.js是单线程吗？
    
> Node.js 做耗时的计算时候，如何避免阻塞？

> Node.js如何实现多进程的开启和关闭？

> Node.js可以创建线程吗？

> 你们开发过程中如何实现进程守护的？

> 除了使用第三方模块，你们自己是否封装过一个多进程架构?

# 进程

进程Process是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础，进程是线程的容器（来自百科）。进程是资源分配的最小单位。我们启动一个服务、运行一个实例，就是开一个服务进程，例如 Java 里的 JVM 本身就是一个进程，Node.js 里通过 node app.js 开启一个服务进程，多进程就是进程的复制（fork），fork 出来的每个进程都拥有自己的独立空间地址、数据栈，一个进程无法访问另外一个进程里定义的变量、数据结构，只有建立了 IPC 通信，进程之间才可数据共享。

```javascript
const http = require('http');

const server = http.createServer();
server.listen(3000,()=>{
    process.title='程序员成长指北测试进程';
    console.log('进程id',process.pid)
})
```
运行上面代码后，以下为 Mac 系统自带的监控工具 “活动监视器” 所展示的效果，可以看到我们刚开启的 Nodejs 进程 7663
![image.png](/images/2019/09/05/827bb4c0-cfe0-11e9-a152-33ef044f5d50.png)

# 线程

线程是操作系统能够进行运算调度的最小单位，首先我们要清楚线程是隶属于进程的，被包含于进程之中。一个线程只能隶属于一个进程，但是一个进程是可以拥有多个线程的。

## 单线程

**单线程就是一个进程只开一个线程**

Javascript 就是属于单线程，程序顺序执行(这里暂且不提JS异步)，可以想象一下队列，前面一个执行完之后，后面才可以执行，当你在使用单线程语言编码时切勿有过多耗时的同步操作，否则线程会造成阻塞，导致后续响应无法处理。你如果采用 Javascript 进行编码时候，请尽可能的利用Javascript异步操作的特性。

```javascript
const http = require('http');
const longComputation = () => {
  let sum = 0;
  for (let i = 0; i < 1e10; i++) {
    sum += i;
  };
  return sum;
};
const server = http.createServer();
server.on('request', (req, res) => {
  if (req.url === '/compute') {
    console.info('计算开始',new Date());
    const sum = longComputation();
    console.info('计算结束',new Date());
    return res.end(`Sum is ${sum}`);
  } else {
    res.end('Ok')
  }
});

server.listen(3000);
//打印结果
//计算开始 2019-07-28T07:08:49.849Z
//计算结束 2019-07-28T07:09:04.522Z

```

查看打印结果，当我们调用127.0.0.1:3000/compute
的时候，如果想要调用其他的路由地址比如127.0.0.1/大约需要15秒时间，也可以说一个用户请求完第一个compute接口后需要等待15秒，这对于用户来说是极其不友好的。下文我会通过创建多进程的方式child_process.fork 和cluster 来解决解决这个问题。

单线程的一些说明

- Node.js 虽然是单线程模型，但是其基于事件驱动、异步非阻塞模式，可以应用于高并发场景，避免了线程创建、线程之间上下文切换所产生的资源开销。
- 当你的项目中需要有大量计算，CPU 耗时的操作时候，要注意考虑开启多进程来完成了。
- Node.js 开发过程中，错误会引起整个应用退出，应用的健壮性值得考验，尤其是错误的异常抛出，以及进程守护是必须要做的。
- 单线程无法利用多核CPU，但是后来Node.js 提供的API以及一些第三方工具相应都得到了解决，文章后面都会讲到。


