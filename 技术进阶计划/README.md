## 目录

## 1.基础知识
### 1.1.JMM内存模型
> JMM内存模型并非真的存在，是逻辑上划分的方便理解。即每个线程有自己独立的工作内存，所有线程共享的主内存。


![](https://tva1.sinaimg.cn/large/008eGmZEly1gnsrn5exchj314k0oq407.jpg)
 - 1.线程使用数据需要经历如下6步骤
   - read：从主内存中将数据读出来
   - load：将读到的数据加载到自己的工作内存
   - use： 线程使用工作内存中的数据，进行修改
   - assign(赋值)： 将数据写回自己的工作内存
   - store(存储)：将数据发送到主内存
   - write：写入主内存


### 1.2.volatile
> volatile经常使用的场景是变量的标志位等，用于同步的变量等(CopyOnWriteArrayList)读

#### 原子性：
> volatile无法保障原子性


#### 可见性：
> 本质原因是写缓冲器和无效队列之间不是实时执行的，会存在延迟
  - 读之前会添加`loadload`屏障，即执行`refresh`指令,强制从主内存加载数据
  - 写之前会添加`storestore`屏障，执行`flush`指令，强制将数据从工作内存刷回主内存
#### 有序性
> 有序性产生的本质原因是，假如一个共享变量此时是M状态的，可以不用写入写缓冲器直接写入`cacheEntry`,而之前的数据可能还没有经过写缓冲器同步到其他线程
 - volatile会添加`Lock前缀指令`，禁止指令重排序。
 - happens-before原则
   - 不允许变量赋值前使用等

### 1.3.synchronized
> synchronized的核心是通过`monitorenter`和`monitorexit`指令来实现加锁和释放锁的过程
#### 核心加锁流程
![](https://tva1.sinaimg.cn/large/008eGmZEly1gnsuuff9huj313n0u0whk.jpg)
 - 每个加锁的对象都会有一个`monitor`计数器和一个`ObjectMonitor`的锁，如果获取锁判定`monitor`是否为0，若为0，加锁并自增。同一个线程的可重入通过monitor的值自增来实现
#### 原子性
`ObjectMonitor`来实现，通过CAS操作`monitor`
#### 可见性
 - monitorenter指令之后：强制执行`load屏障`,refresh最新数据到高速缓存，确保获取到最新数据
 - monitorexit之后：强制执行`store屏障`，flush数据到高速缓存或者主内存
#### 有序性
> 通过`acquire屏障`和`release屏障`，保障`synchronized`修饰的方法内部可进行指令重排，外部不可
 - Acquire：保障`Acquire`后的读写操作不会发生在`Acquire动作`之前
 - Release：保障`Release`前的读写操作不会发生在`Release动作`之前 

### 1.3.可能指令重排的几个地方
![](https://tva1.sinaimg.cn/large/008eGmZEly1gnsw2gv0hdj31es0ietcg.jpg)

 - 高速缓存：无效队列和谐缓冲器可能存在执行顺序不一致的问题

### 1.4.ThreadLocal
> 核心原理在于 `Thread`中维护了一个`ThreadLocalMap`的类，用来维护每个线程的局部变量，下次这个线程还能获取到该变量

## 2.无锁化
### 2.1.CAS相关问题
> Atomic*相关的一些操作都是通过CAS来实现的，底层通过`Usafe`类来实现，通过偏移量，当前值和期望值来进行比对处理
### 2.2.ABA问题
> 通过实践戳来解决

### 2.3.自旋问题
> CAS内部通过while循环来实现

## 3.AQS与Lock
### 3.1.AQS
> 抽象队列同步器，多种`juc`核心组件的底层实现。

 - state：通过`state`状态维护判定是否加锁以及锁的重入(类似Synchronized)，通过CAS操作`state`来实现获取锁
 - node队列：先进先出的双向链表，维护数据的队列
### 3.2.ReentrantLock
> 底层通过AQS实现，提供公平和非公平两种实现

![](https://tva1.sinaimg.cn/large/008eGmZEly1gmxo0zhvndj30nw0850t6.jpg)
 - Fair: 所有请求加锁的的线程依次进入队列
 - Nonfair：新申请加锁的线程会先尝试调用下`accqure()`方法，若能够获取到锁则直接占用，即和队头元素竞争锁，若失败，入队
#### 加锁流程(公平锁)
> 入队的线程会调用`LockSupport.park()`将当前线程挂起

![](https://tva1.sinaimg.cn/large/008eGmZEly1gmyvl6n8c4j31c40k6gni.jpg)
 - 线程基于state判定，若state=0，则进行排队。若state!=0，但是占用线程是当前线程，则对state进行
   - state=0，直接入队
   - state!=0，并且占用线程是当前线程，state自增，可重入锁
   - stete!=0,占用线程非当前线程，入队

#### 释放锁流程
![](https://tva1.sinaimg.cn/large/008eGmZEly1gmywjxhbizj31ok0pen05.jpg)

 - 假设线程1释放锁，会对state进行递减操作，直到state=0时，设置加锁线程为null，下一个线程才可以去抢占锁
 - 此时线程1释放锁之后会唤醒线程2，线程2会继续去申请加锁

### 3.3.ReentrantReadWriteLock
> 和`ReentrantLock`的区别在于使用了使用了两把锁来区分读写

#### 锁的互斥
> 通过将`state`的高低16位来区分读锁和写锁
其中`高16位代表读锁`,低16位`写锁`

 - 读读锁：共享
 - 读写锁：互斥
 - 写写锁：互斥

#### 写锁的流程
![](https://tva1.sinaimg.cn/large/008eGmZEly1gmzzy6g21gj310b0c70tt.jpg)
 - 先判断`state`是否为0，若为0，说明没有加任何锁。直接加写锁
 - 若state不为0，说明加过锁
   - 判断w即低16位，写锁，若w=0，说明未加锁，直接加锁并设置加锁线程为当前线程。
   - 若w不为0，但是加锁线程为当前线程，则低16位递增，做可重入
   - 若写入失败，则入队，在tail后拼接一个新的node

#### 读写锁互斥
> 基于`state`的高低16位判断，即w和r的值进行判断

#### 锁的释放

 - 释放锁：将state进行递减操作
 - 若低16位为0时，解锁成功，设置`ownerThread`为空，唤醒继任者

## 4.缓存一致性协议(MESI)
> 主要涉及到`写缓冲器`和`无效队列`

![](https://tva1.sinaimg.cn/large/008eGmZEly1gng6ozv7g7j314u0h7dhc.jpg)

### MESI协议
 - M：修改
 - E：独占
 - I：无效
 - S：共享

### 共享变量的写入
> cpu0需要对共享变量进行修改

 - cpu0：先将数据写入写缓冲器，同事发送`invalidate`消息，非阻塞
 - cpu1：嗅探到`invalidate`消息，写入无效队列，返回`ack`消息
 - cpu0：嗅探到`invalidate ack`消息，将数据设置为`E`独占锁，并从写缓冲器中拿到之前的数据，修改数据，然后将状态修改为`M`状态
 - cpu1：使用该数据时，发现该数据状态是`I`,发送消息到总线去读取消息，cpu0同步完数据之后，将`cpu0`和`cpu1`数据的状态修改为`S`

### 可见性与有序性的问题
 - 可见性：主要是写缓冲器和无效队列异步写数据导致的。写数据不是写入高速缓存，读也不是立即从高速缓存获取
 - 有序性：因为如果是`M`状态的数据，不是写入写缓冲器，而是直接写入`cache line`


## 5.并发数据结构
### 5.1.ConcurrentHashMap
#### hashmap存在的问题
> jdk1.7中，多线程同时扩容的情况下，倒序插入可能会导致死循环，如下图

![](https://tva1.sinaimg.cn/large/008eGmZEly1gng9hrljocj30qq0cj3ys.jpg)
 - 假如线程1在处理的时候，线程2也在处理,而线程1处理完了之后，线程2才继续处理
 - 此时线程1节点顺序变为 k3->k2->k1
 - 而线程2继续执行，此前的顺序为k1->k2,此时线程1已将k2的next变为k1，此时变成了死循环。
#### 分段加锁机制
> 在concurrentHashmap中使用了分段加锁机制，并且jdk1.7也1.8也是不一样的
 
  - 1.7：16个`Segment`，每个对应一个Node数组，放到同一个Node数组的元素竞争同一把锁，锁`Segment`
  - 1.8：定位到数组元素为空时，通过cas加锁，若不为空，则通过`synchronized`锁头结点，锁`头结点`
#### put操作
 - 数组为空：执行数组初始化操作，此处使用的是`sizeCtl`状态位来进行CAS并发控制
 - 定位：基于hash算法(此处采用的是位运算，将高低16位都考虑进行)，定位到该元素在数组中的位置
   - 若该位置还没有元素，则通过CAS将元素设置进去，CAS成功直接break
   - 若CAS失败则说明有其他线程同时设置元素进去，则等下一轮循环，此时数组中该位置不为null，则会使用`synchronized`对链表节点加锁，将该元素追加到链表尾部去
#### get()/size()
> 不涉及锁，使用`volatile`，通过load屏障读到最新数据

#### 扩容
> 基于`sizeCtl`，cas进行扩容。

### 5.2.CopyOnWriteArrayList
> 写时复制，每次写都会复制一个新的数组快照

 - 读：volatile，实现共享读
 - 写：加`ReentrantLock`独占锁，新数组长度增1，放入新数据，此时读的数据不是最新的
 - 写：删除，和修改等使用的是一把锁，因此彼此互斥

### 5.3.CountDownLatch

### 5.4.ConcurrentLinkedQueue
> 无界队列，基于CAS操作


### 5.5.LinkedBlocingQueue
> 有界队列，独占锁+Condition队列。

![](https://tva1.sinaimg.cn/large/008eGmZEly1gnn2cfzgtvj31t40im0uo.jpg)
## 6.线程池
> 线程池`corePoolSize`或者`maxPoolSize`中指的是`Worker(AQS)`
基于AQS创建，将一个个`Runnable`任务封装成Worker,放到haset中，

### 6.1.核心参数
 - corePoolSize：核心线程数，线程存放到hashset中
 - maxPoolSize：最大线程数，若线程数已达到核心线程数，并且工作队列已满，则开始创建新线程，最终到maxPoolSize，线程存放到hashset中
 - keepAliveTime+TimeUnit：超出核心线程数的部分线程，若空闲，且空闲时间超过`keepAliveTime`则销毁
 - workQueue：线程中的任务队列，队列分为很多种`LinkedBlockingQueue`,`SynchronousQueue`等
 - rejectHandler：拒绝策略，workQueue已满并且线程数已达到maxPoolSize，则只需拒绝策略
 - threadFactory：创建线程的工厂
### 6.2.执行过程
> 基于一个AtomicInteger的数据`ctl`(32)位
   - 线程状态：前3位
   - 线程数量：后29位
#### 执行过程
![](https://tva1.sinaimg.cn/large/008eGmZEly1gnrjc6d909j30nk0jwabn.jpg)
   - 线程数是否小于corePoolSize,若小于，添加到`workQueue`中
   - 若线程数大于`corePoolSize`,`workQueue`是否已满,若未满则添加任务到队列
   - 若`workQueue`已满，则判断线程数是否小于`maxPoolSize`
     - 若小于，则创建新的工作线程执行任务
     - 若大于，则执行拒绝策略
#### 执行细节
![](https://tva1.sinaimg.cn/large/008eGmZEly1gnrqh4p2inj30u50r741c.jpg)

### 6.3.核心数据结构
#### LinkedBlockingQueue

#### SynchronousQueue

### 6.4.编程实战选型


### 6.5.参数配置动态化
> 线程池的参数没有通用的解决方案，io密集型,cpu*2,技术密集型cpu+1这种说法并不准确

一般采用动态配置的方式
  - 基于配置中心动态调整，corePoolSize和maxPoolSize
  - 基于QPS动态调整
    - 使用redis滑动窗口计算qps

## 7.锁优化策略

 - 无锁化：标志位，以及状态判定等优先使用`volatile`
 - CAS：数值类型递增，非强一致状态使用等，并发编程的数据结构中大量使用了CAS
 - 读写锁：读多写少的场景:`ReentrantReadWriteLock`
 - 分段加锁：减小锁力度，ConcurrentHashMap等
 - 减少并发场景下锁的争抢：注册中心中的多级缓存机制
 - 死锁问题：至少两把锁，彼此争抢
   - jstack pid -->dump.txt
 - 线程饥饿，活锁：非公平锁状态下，某些线程一直获取不到锁 








