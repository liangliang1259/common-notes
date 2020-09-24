## 前言
上一篇已经完成了Eureka的初步简介，这篇文章我们来研究Eureka源码环境的搭建
### 课程目标

## 目录
 - 概述
 - 源码环境搭建
 - 入口类

## 1.概述
eureka源码不是直接由springcloud提供的，我们平时通过注解`@EnableDiscoveryClient`和`@EnableEurekaServer`使用的，都是`springcloud`在原有代码上进行了封装的。而原生的底层实现是有`netflix`实现的。因此我们阅读源码的时候直接阅读`netflix`下的eureka源码。
### 1.1.源码的阅读技巧
 - 抓住核心流程，其他流程去猜测它的实现，后续去验证
 - 通过画图去梳理，可以先绘制一个大图，然后逐步去完成其细节。
## 2.源码环境搭建
> 本博客以Eureka的,此处i建议使用`idea.2018`版本，Eureka选择`v1.7.x`分支
### 2.1.环境配置
 - gradle配置：eureka需要依赖gradle的环境配置。关于gradle的配置这里就不多说明了，自己网上百度下就好
 - 源码下载
``` shell
git clone https://github.com/Netflix/eureka.git
```
 - 执行eureka目录下的`gradlew.bat`文件，mac的执行`gradlew命令`
### 2.2.导入idea
> 使用`idea.2018`版本导入，其他版本可能存在问题，导入idea后如下图，分为如下几个module

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gj0db8opirj31400pjjx4.jpg)
>结构图概述
 - eureka-client：eureka的客户端，注册到Eureka上的一个服务，即我们通过`@EnableDiscoveryClient`完成标记的这些服务
 - eureka-client-jersey2: jersey2是一个mvc框架，类似我们经常使用的Spring MVC，这里主要是通过jersey2来完成`Eureka Server`和`Eureka Client`之间服务的数据传输的
 - eureka-core: eureka server包中引入的一个子module，该模块主要就是eureka server的一些处理，也即eureka的注册中心
 - eureka-core-jersey2: 同eureka-client-jersey2，这里是`eureka-server`中的一些mvc通信的组件
 - eureka-examples: 一些测试案例，可以debug来完成服务的处理
 - eureka-resources：一些资源管理，这里主要是eureka中的web页面资源，也即我们上一篇看到的eureka的页面，这里主要是一些jsp的资源
 - eureka-server：该模块的gradle中引入了`eureka-core`,`eureka-client`以及`eureka-resources`，会将这几个组件打包成一个war。从这里也就可以看出，我们的eureka-server会同时作为一个client注册到其他服务器上。

## 3.入门分析
基于我们之前的知识，我们知道Eureka的启动顺序是 `Eureka Server`--> `Eureka Client`启动并注册到 Eureka Server上 --> 服务之间完成调用。如图
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gizjhgyqryj316e0ry76u.jpg)
 - 入口类
那么我们先来了解下`Eureka Server`启动时，都做了哪些事情。先梳理Eureka Server的启动流程。Eureka server是一个jsp组成的项目，所以我们先研究下`Eureka server`下的`web.xml`文件。
```xml
  <listener>
    <listener-class>com.netflix.eureka.EurekaBootStrap</listener-class>
  </listener>
```
回忆下我们之前学习jsp的时候，项目的入口类就是`listener`，因此我们可以找到`EurekaBootStrap`:该类即为`Eureka server`的入口类。
 - 主页
```xml
  <welcome-file-list>
    <welcome-file>jsp/status.jsp</welcome-file>
  </welcome-file-list>
```
![](https://tva1.sinaimg.cn/large/007S8ZIlly1giznihsn34j31xx0u00yn.jpg)
该页面即为我们上一篇看到的页面。
 - filter
此处的一些filter，暂时先不管，后续再进行讲解。
```xml
  <filter>
    <filter-name>requestAuthFilter</filter-name>
    <filter-class>com.netflix.eureka.ServerRequestAuthFilter</filter-class>
  </filter>
  <filter>
    <filter-name>rateLimitingFilter</filter-name>
    <filter-class>com.netflix.eureka.RateLimitingFilter</filter-class>
  </filter>
  <filter>
    <filter-name>gzipEncodingEnforcingFilter</filter-name>
    <filter-class>com.netflix.eureka.GzipEncodingEnforcingFilter</filter-class>
  </filter>
```
本篇内容就到此为止，先对Eureka的结构有个大致的了解，然后下一节我们开始Eureka Server的启动流程。
## 关于

 - Github: [https://github.com/liangliang1259/common-notes](https://github.com/liangliang1259/common-notes)
 - 公众号
![](https://tva1.sinaimg.cn/large/007S8ZIlly1giznpxhgdvj3076076gm3.jpg)
