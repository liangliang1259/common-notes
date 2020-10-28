## 资源池容量管理+依赖服务划分
> 依赖服务->接口->线程池

```java
super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("GetCityNameGroup"))
    .andCommandKey(HystrixCommandKey.Factory.asKey("GetCityNameCommand"))
    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("GetCityNamePool"))
    .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
        .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)
        .withExecutionIsolationSemaphoreMaxConcurrentRequests(15)));
```
 - group:默认使用`command group`来定义线程池，可基于此聚合和报警信息
 - key:每个command都可以设置一个key
 - ThreadPoolKey：代表一个HystrixThreadPool，用来进行统一的监控，统计，缓存等，默认为`command group`名称

### 1.常用参数
 - coreSize: 线程池大学，默认10
```java
HystrixThreadPoolProperties.Setter()
   .withCoreSize(int value)
```
 - queueSizeRejectionThreshold

### 2.图说线程池及队列
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjvkbj8uoyj30iz05dglr.jpg)
 - 假如线程池最大容量为10个线程，而队列的最大容量为5个
 - 此时线程池中的10个线程都在工作中，新的请求就会进入到队列中去。
 - 若队列也满了，此时请求就会被拒绝。进行fallback降级逻辑，快速失败。

### 3.Hystrix执行流程
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjvmr251rgj30m70h2jsf.jpg)
#### 3.1.构建Command
```java
HystrixCommand command = new HystrixCommand(arg1, arg2);
HystrixObservableCommand command = new HystrixObservableCommand(arg1, arg2);
```
 - HystrixCommand：一条结果的调用
 - HystrixObservableCommand：多条结果的调用 

#### 3.2.调用Command执行方法
 - HystrixCommand: execute()，queue()
 - HystrixObservableCommand： observe()，toObservable()

```java
execute()：调用后block，同步调用，直到返回结果或者异常
queue()：异步调用：后续通过Future返回单条结果
observe()：定义一个Observable对象，
```

#### 3.3.检查是否开启缓存
若开启了缓存，则从缓存中直接返回结果，这里使用的是request context缓存

#### 3.4.检查是否开启了断路器
若该command对应依赖服务开启了断路器，则直接进行fallback，不再执行该command

#### 3.5.检查线程池/队列/Semaphore是否满了
如果command对应的线程池等满了，则直接拒绝，进行fallback降级，并不会执行该command

#### 3.6.执行command
执行command的run(),若出现`timeout`或者其他异常则进行fallback降级。

#### 3.7.短路健康检查
 - Hystrix会将每个依赖服务的调用事件发送给断路器进行统计，如成功，失败，超时等事件。 - 基于这些事件出现的次数进行统计，之后基于统计次数判定是否打开断路器。
 - 若打开断路器，则一段时间内会进行短路。在此时间之后若调用成功，则关闭断路器