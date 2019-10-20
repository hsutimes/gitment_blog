---
title: Spring框架面试总结
tags:
  - 面试
  - Spring
categories:
  - 文章
toc: false
date: 2019-09-04 00:32:24
---

![image.png](/images/2019/09/12/ba1add50-d555-11e9-b519-9f31cc87a9f9.png)

# 介绍spring框架

它是一个一站式（full-stack全栈式）框架，提供了从表现层-springMVC到业务层-spring再到持久层-springdata的一套完整的解决方案。我们在项目中可以只使用spring一个框架，它就可以提供表现层的mvc框架，持久层的Dao框架。它的两大核心IoC和AOP更是为我们程序解耦和代码简洁易维护提供了支持。

# Spring的优点？

   1. 降低了组件之间的耦合性 ，实现了软件各层之间的解耦 
   2. 可以使用容易提供的众多服务，如事务管理，消息服务等 
   3. 容器提供单例模式支持 
   4. 容器提供了AOP技术，利用它很容易实现如权限拦截，运行期监控等功能 
   5. 容器提供了众多的辅助类，能加快应用的开发 
   6. spring对于主流的应用框架提供了集成支持，如hibernate，   JPA，Struts等 
   7. spring属于低侵入式设计，代码的污染极低 
   8. 独立于各种应用服务器 
   9. spring的DI机制降低了业务对象替换的复杂性 
   10. Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可以自由选择spring 的部分或全部 

# spring有两种代理方式：

答: 若目标对象实现了若干接口，spring使用JDK的java.lang.reflect.Proxy类代理。

  优点：因为有接口，所以使系统更加松耦合

  缺点：为每一个目标类创建接口

若目标对象没有实现任何接口，spring使用CGLIB库生成目标对象的子类。

  优点：因为代理类与目标类是继承关系，所以不需要有接口的存在。

  缺点：因为没有使用接口，所以系统的耦合性没有使用JDK的动态代理好



# 如何给Spring 容器提供配置元数据?

这里有三种重要的方法给Spring 容器提供配置元数据。

XML配置文件。

基于注解的配置。

基于java的配置。



# 构造方法注入和设值注入有什么区别？

请注意以下明显的区别：

在设值注入方法支持大部分的依赖注入，如果我们仅需要注入int、string和long型的变量，我们不要用设值的方法注入。

对于基本类型，如果我们没有注入的话，可以为基本类型设置默认值。

在构造方法注入不支持大部分的依赖注入，因为在调用构造方法中必须传入正确的构造参数，否则的话为报错。

设值注入不会重写构造方法的值。

在使用设值注入时有可能还不能保证某种依赖是否已经被注入，也就是说这时对象的依赖关系有可能是不完整的。而在另一种情况下，构造器注入则不允许生成依赖关系不完整的对象。

# 请介绍一下Spring框架中Bean的生命周期 

一、Bean的定义 

Spring通常通过配置文件定义Bean。如：  

这个配置文件就定义了一个标识为 HelloWorld 的Bean。在一个配置文档中可以定义多个Bean。 

二、Bean的初始化

有两种方式初始化Bean。 

1、在配置文档中通过指定init-method 属性来完成 

2、实现 org.springframwork.beans.factory.InitializingBean接口  

那么，当这个Bean的所有属性被Spring的BeanFactory设置完后，会自动调用afterPropertiesSet()方法对Bean进行初始化，于是，配置文件就不用指定 init-method属性了。 

三、Bean的调用 

有三种方式可以得到Bean并进行调用： 

1、使用BeanWrapper

2、使用BeanFactory

3、使用ApplicationConttext

四、Bean的销毁 

1、使用配置文件中的 destory-method 属性

2、实现 org.springframwork.bean.factory.DisposebleBean接口 



# Spring中AOP的应用场景、Aop原理、好处？

答：AOP--Aspect Oriented Programming面向切面编程；用来封装横切关注点，具体可以在下面的场景中使用:

Authentication 权限、Caching 缓存、Context passing 内容传递、Error handling 错误处理Lazy loading懒加载、Debugging调试、logging, tracing, profiling and monitoring 记录跟踪优化　校准、Performance optimization　性能优化、Persistence 持久化、Resource pooling　资源池、Synchronization　同步、Transactions 事务

原理：AOP是面向切面编程，是通过动态代理的方式为程序添加统一功能，集中解决一些公共问题。

优点：1.各个步骤之间的良好隔离性耦合性大大降低 

    2.源代码无关性，再扩展功能的同时不对源码进行修改操作 



# 有几种不同类型的自动代理？

BeanNameAutoProxyCreator

DefaultAdvisorAutoProxyCreator

Metadata autoproxying



# ApplicationContext通常的实现是什么?

FileSystemXmlApplicationContext ：此容器从一个XML文件中加载beans的定义，XML Bean 配置文件的全路径名必须提供给它的构造函数。

ClassPathXmlApplicationContext：此容器也从一个XML文件中加载beans的定义，这里，你需要正确设置classpath因为这个容器将在classpath里找bean配置。

WebXmlApplicationContext：此容器加载一个XML文件，此文件定义了一个WEB应用的所有bean。



# 有哪些不同类型的IOC（依赖注入）方式？

构造器依赖注入：构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参数代表一个对其他类的依赖。

Setter方法注入：Setter方法注入是容器通过调用无参构造器或无参static工厂 方法实例化bean之后，调用该bean的setter方法，即实现了基于setter的依赖注入。



# 什么是IOC，什么又是DI，他们有什么区别？

一、IOC介绍

IOC是控制反转。

创建对象实例的控制权从代码控制剥离到IOC容器控制(之前的写法，由程序代码直接操控使用new关键字)，实际就是你在xml文件控制，控制权的转移是所谓反转，侧重于原理。 

二、DI介绍

DI是依赖注入

创建对象实例时，为这个对象注入属性值或其它对象实例，侧重于实现。



# spring事务定义

事务的定义：事务是指多个操作单元组成的合集，多个单元操作是整体不可分割的，要么都操作不成功，要么都成功。其必须遵循四个原则（ACID）。

原子性（Atomicity）：即事务是不可分割的最小工作单元，事务内的操作要么全做，要么全不做；

一致性（Consistency）：在事务执行前数据库的数据处于正确的状态，而事务执行完成后数据库的数据还是应该处于正确的状态，即数据完整性约束没有被破坏；如银行转帐，A转帐给B，必须保证A的钱一定转给B，一定不会出现A的钱转了但B没收到，否则数据库的数据就处于不一致（不正确）的状态。

隔离性（Isolation）：并发事务执行之间互不影响，在一个事务内部的操作对其他事务是不产生影响，这需要事务隔离级别来指定隔离性；

持久性（Durability）：事务一旦执行成功，它对数据库的数据的改变必须是永久的，不会因比如遇到系统故障或断电造成数据不一致或丢失。



# Spring框架中的单例Beans是线程安全的么？

Spring框架并没有对单例bean进行任何多线程的封装处理。关于单例bean的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的Spring bean并没有可变的状态(比如Serview类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。