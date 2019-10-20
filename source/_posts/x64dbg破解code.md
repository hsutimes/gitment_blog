---
title: x64dbg破解code
tags:
  - x64dbg
categories:
  - 文章
toc: false
date: 2019-10-05 15:35:05
---

# x64dbg破解code

```
1075248.PAR  pp.las  ppcheck.scr  ppcheck_A.exe*  新建文本文档.txt
1075248.SET  pp.pwd  ppcheck.txt  PPM1828.DAT
```


> 将后缀是.par .set的文件名称修改为想要查询的序列号，后缀要保留。 点击PPCHUCK_A.EXE的执行文件执行，选择适当的机器型号，选择需要查询的选项，回车执行，选择三种润滑方式任意一种。执行。

## x64dbg加载ppcheck_A.exe

![image.png](/images/2019/10/05/96dd00a0-e740-11e9-8a93-9b78d2af1e3b.png)

设置断点，运行程序，进行测试。

![image.png](/images/2019/10/05/c9255120-e740-11e9-8a93-9b78d2af1e3b.png)

CALL 过程调用相同

```bash
call 0x0040ACE6
```

![image.png](/images/2019/10/05/34fb0810-e744-11e9-8a93-9b78d2af1e3b.png)

分析核心代码。。。


## 模拟程序执行过程

1. Please enter a serial number:

输入 num 

![image.png](/images/2019/10/06/e17c47d0-e810-11e9-9212-5951f1d89d17.png)

输入的数据存入内存 `0x42A708` 

![image.png](/images/2019/10/06/8870daa0-e817-11e9-9212-5951f1d89d17.png)

![image.png](/images/2019/10/06/9c557990-e817-11e9-9212-5951f1d89d17.png)

`0x00429374`

2. 选择后Enter

```
Code to turn on TSC：   742
```

![image.png](/images/2019/10/06/777a09a0-e813-11e9-9212-5951f1d89d17.png)

汇编代码执行到

![image.png](/images/2019/10/06/d67d2860-e813-11e9-9212-5951f1d89d17.png)

推测，计算code的程序在打印头部信息 HAAS 与 打印 Code 之间

存储 TSC code 的地址为 `0x0042F26C`

![image.png](/images/2019/10/06/3d9ba030-e814-11e9-9212-5951f1d89d17.png)

16进制表示： `000002E6`

![image.png](/images/2019/10/06/4d72a620-e814-11e9-9212-5951f1d89d17.png)

总结执行步骤

- 输入num
- 选择型号（三次选择）
- 打印对应参数的code

在汇编程序中的输入num的内存地址与code的值得内存地址都可以找到，而且程序执行中均已常数形式存在，推测在（三次选择）结束后，所有参数对应的code值均计算后，以常数形式保存在，之后的程序判断将部分的参数对应的code值打印出。


![image.png](/images/2019/10/07/3a6e4da0-e8db-11e9-b1c6-59aa3ad54a59.png)