# 总体预览

spring可以说是现在java开发的标配了，它的IOC和AOP理念让轻量级开发企业应用成为可能。它的优点这里就不赘述了。但是自从2003年兴起到现在,spring已经发展成为一整套解决方案，下至数据库，上至云都有他的身影。这也造成了它的源码变得非常不轻量级，想看懂它的源码，那家伙绝对是烧脑的任务。

这里借用官方的架构图开启我的spring学习之旅。

![image](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/images/spring-overview.png)

架构图清晰的标明了spring的架构关系

# 模块简介

## 一、核心库

1、spring-beans：spring的IoC的基础实现，包含访问配置文件、创建和管理bean等；

2、spring-core：这个jar包中包含了spring的核心工具类，其他组件都需要使用的一个jar包；

3、spring-context、spring-context-support：在基础IOC功能上提供扩展服务，此外还提供许多企业级服务的支持，有缓存、多种视图层框架的支持、邮件服务、任务调度、JNDI定位，EJB集成、远程访问；

4、spring-expression  目前项目中没有涉猎这里不讨论，简单说来就是Spring表达式语言；

## 二、Aop and Instrumentation

1、spring-aop：提供了面向切面的编程能力，并且允许自定义；

2、spring-aspects：提供了和aspectj的整合能力；

3、spring-instrument，spring-messaging：目前项目中没有涉猎这里不讨论

## 三、数据库相关

1、spring-jdbc：提供了对jdbc的简单抽象，并提供了不同数据库错误的解析能力；

2、spring-orm：提供了整合不同orm框架的能力；

3、spring-tx：提供编程和声明式事务管理能力；

4、spring-oxm：目前项目中没有涉猎这里不讨论，简单说来提供了xml和object的mapping能力；

5、spring-jms：目前项目中没有涉猎这里不讨论，简单说来提供了生成消息和使用消息的特性；

## 四、web

1、spring-web：提供了面向web的基础特性，诸如：多文件上传，通过listener初始化IoC的容器，一个面向web的application context，当然它也包含了一些spring远程能力。

2、spring-webmvc：提供了MVC能力；

3、spring-websocket, and spring-webmvc-portlet：目前项目中没有涉猎这里不讨论

## 五、test

spring-test：支持spring组建junit或TestNG的集成测试和单元测试。它提供了一致spring ApplicationContext的加载和上下文的缓存。他还提供了可以用来测试代码隔离的模拟对象。
