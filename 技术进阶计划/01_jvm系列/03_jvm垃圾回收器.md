## 前言
week03:本节主要学习jvm新生代和老年代的划分以及痛点
### 课程目标
通过本课程可以了解如下内容，希望大家能够掌握和了解如下内容。
 - xxxx
 - xxxx
## 目录
 - 概述
 - 一图百文
   - 架构图
   - 流程图 
 - 代码片段
 - 图文总结
 - 关于
## 问题集锦
### 1.新生代中的垃圾回收
#### 1.1.新生代中何时出发GC?
当新生代中的对象越来越多，快满的时候，就会出发新生代的垃圾回收。进而释放空间？
#### 1.2.哪些对象是不能被GC的？
> 可达性分析算法，GC ROOTs引用的对象无法被回收，
GC Roots：静态变量，局部变量

### 2.新生代的垃圾回收算法
>由于标记清除的算法会产生内存碎片，因此这里采用了复制算法。
> 将新生代分为Eden和survivor区域，并且将survivor分为s0和s1这样，彼此的内存比例为
>  Eden:s0:s1 = 8:1:1,因此新生代的可用内存可达到90%

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjqxlqdstkj30ag07ldg3.jpg)
 - 系统每次创建的对象都会放在Eden区域中
 - 当eden区域中的内存快满时候，会将存活对象一次性转移到s0或者s1中(空的区域，假设为s0)，然后一次性清空eden区域。
 - 当s0的区域也满时，这时候会把eden和s0中存活的对象都转移到s1中去，同时一次性清空edne和s0
 - 对象经历15次GC之后进入老年代。`“-XX:MaxTenuringThreshold”`设置进入老年代的年龄，默认15

### 3.新生代的对象在什么情况下会进入老年代？




## 5.关于
 - Github: [https://github.com/liangliang1259/common-notes](https://github.com/liangliang1259/common-notes)
 - 公众号
![](https://tva1.sinaimg.cn/large/007S8ZIlly1giznpxhgdvj3076076gm3.jpg)
