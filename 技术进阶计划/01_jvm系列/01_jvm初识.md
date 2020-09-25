## 前言
该系列内容用来记录自己在JVM学习中的一些知识，梳理学习内容同时记录分享，进行交流。
### 课程目标
通过本课程可以了解如下内容，希望大家能够掌握和了解如下内容。
 - jvm处理的简单流程
 - jvm内存模型初识
## 目录
 - 概述
 - JVM内存模型
 - 大图总结
## 1.概述
> 一些概念
### 1.1.java简单的处理流程
我们大家应该都是经历过代码上线的流程，通过maven进行打包`mvn package`打出一个`java`包，然后通过`java -jar xx.jar`命令就启动了一个java程序，但是大家有没有思考过在这个过程中，我们究竟做了什么，或者说程序到底做了什么？
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gj1u71jll2j317407ydh1.jpg)
这是我们一个系统运行的一个简单流程图，从我们代码的编写到整个的运行。那么class文件是如何到JVM中并且运行起来的呢？这里就需要引入一个新的概念了`类加载器`，相信对于JVM有过了解的同学应该是很熟悉这个东西了。
### 1.2.JVM中的处理流程
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gj1up4d04kj31620qa76g.jpg)

这里`类加载器`会将class文件加载到jvm中去，然后供后续代码执行，这里还有一个执行引擎的概念。

## 2.JVM内存模型
> 类加载器加载到JVM的过程中肯定不是简单的把class类加载进内存，然后就去执行代码了，那么它的内部的内存模型究竟是怎样的呢？

JVM的内存模型在不同的虚拟机实现上略有不同，如`Hotspot`和`JRockit`是不一样的，而在JDK1.8中两者进行了合并，这也是为什么JDK1.7和1.8的内存区域有比较大的区别的原因。
### 2.1.内存模型规范
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gj2pifddfqj30yu0r2wgz.jpg)
#### 堆
<font color=red>所有线程共享</font>：该区域是需要我们重点关注的区域，所有的对象实例及数组都在此处进行分配。是垃圾回收器重点处理的区域
#### 方法区
<font color=red>所有线程共享</font>：该区域主要用于存储类的一些信息，常量、静态变量等信息。

#### 虚拟机栈
<font color=green>线程私有</font>：为虚拟机使用到的java方法提供服务。

#### 本地方法栈
<font color=green>线程私有</font>：为虚拟机使用到的本地`native`方法服务，如多线程CAS中的`Unsafe`类

#### 程序计数器
<font color=green>线程私有</font>：可以把它当做时程序的字节码执行的行号记录器。每个线程私有，它会记录该线程执行到代码中的哪一行

## 3.大图总结
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gj2pxs61u3j31ec0kqn1a.jpg)

## 关于
 - Github: [https://github.com/liangliang1259/common-notes](https://github.com/liangliang1259/common-notes)
 - 公众号
![](https://tva1.sinaimg.cn/large/007S8ZIlly1giznpxhgdvj3076076gm3.jpg)
