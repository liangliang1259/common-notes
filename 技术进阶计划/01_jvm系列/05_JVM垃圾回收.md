## 前言 
> jvm垃圾回收机制

### 1.新生代垃圾回收(Minor GC)
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjk8n0hhu4j312w0fsgn0.jpg)
 - 1.新对象创建时，会先进入Eden区域，当Eden区域内存不足以容纳时，会触发垃圾回收。
 - 2.这个时候会采用`复制算法`(S0或者S1区域中会有一个是空的，假设S0为空),会将Eden区域中和S1区域中的存活对象标记出来，然后移动到空闲的S0区域中，并把Eden和S1清除

### 2.触发Minor GC之前会如何检查老年代大小
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjk8zthgvgj30v90u0gof.jpg)

### 3.Minor GC过后可能对应哪几种情况？ 
 - 1.Minor GC前判断：存活对象所占内存空间 < Survivor区域内存空间的大小，存活对象进入Survivor区域。否则直接进入老年代
 - 2.(存活对象所占空间 > Survivor区域内存空间) && (存活对象所占空间 > 老年代可用空间)触发Full GC。之后再进行Minor GC。若空间还不足则触发OOM。

## 4.哪些Minor GC后的对象直接进入老年代？
 - 经过15次`Minor GC`之后
 - 存活对象占用空间 > Survivor区 && 存活对象占用的空间 < 老年代可用区域



## 5.关于
 - Github: [https://github.com/liangliang1259/common-notes](https://github.com/liangliang1259/common-notes)
 - 公众号
![](https://tva1.sinaimg.cn/large/007S8ZIlly1giznpxhgdvj3076076gm3.jpg)
