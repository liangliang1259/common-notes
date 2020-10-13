## 前言
本篇内容作为Hystrix的入门先导课，主要介绍Hystrix的一些作用，以及核心功能。
### 课程目标
 - hystrix是什么？
 - 高可用，熔断，限流
 - Hystrix的隔离技术
## 目录
 - 概述
 - Hystrix设计原则
 - 故障背景及解决方案
 - 隔离技术
## 1.概述
Hystrix是微服务中提供资源隔离，熔断，降级的工具。由Netflix推出的这款组件是断路器的一种实现，用于解决微服务中服务雪崩，并提高服务的可用性。
## 2.hystrix的设计原则
1.阻止服务因为并发等问题耗尽所有资源。提供线程隔离等功能
2.避免排队和积压，基于fast-fail机制
3.资源隔离技术，比如bulkhead（舱壁隔离技术），swimlane（泳道技术），circuit breaker（短路技术），来限制任何一个依赖服务的故障的影响

## 3.故障
### 3.1.故障现场
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjmqzpadbyj30ve0p23zh.jpg)
如上图，假设A服务需要调用B,而B服务又需要依赖C,D，E三个服务。
1. 此时C服务出现问题，有可能会导致B服务也挂掉，进而导致A服务也会挂掉，这即称作服务的雪崩。
   1. 在系统内部去发送网络请求都是并发的去访问，需要依赖于自己的线程资源。每个线程发起一次请求。而系统的总资源是有限的，这里假设系统只有100个线程资源。最多并发数为100.
2. 假设B服务调用C，D，E服务时，都是30个线程，每个服务调用平均耗时都是20ms
3. 因为C服务异常，导致超时，超时时间是2s，这样就导致会有更多的线程资源来访问C。30个线程来访问C时候不够，会调度更多的线程资源来访问，然后可能会出现线程资源耗尽的情况。进而导致B服务无法提供服务。导致宕机。
### 3.2.故障解决
而采用Hystrix就可以解决该问题，Hystrix通过资源隔离的方式。可以给请求设置最大线程资源，达到资源隔离的效果。

- 服务B给服务C的调用设置40个线程，D和E分别为30个，这样即使服务C出现异常，也只有40个线程资源耗尽，其他服务可以正常访问

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjmrtf0u2vj317k0qe41t.jpg)

## 4.隔离技术
### 4.1.线程池
 - 单个接口:HystrixCommand
```java
public class GetProductInfoCommand extends HystrixCommand<ProductInfo> {
  private Long productId;
  public GetProductInfoCommand(Long productId) {
    super(HystrixCommandGroupKey.Factory.asKey("GetProductInfoGroup"));
    this.productId = productId;
  }
  @Override
  protected ProductInfo run() throws Exception {
    System.out.println("调用接口，查询商品数据，productId=" + productId);
    String url = "http://127.0.0.1:8082/getProductInfo?productId=" + productId;
    String response = HttpClientUtils.sendGetRequest(url);
    return JSONObject.parseObject(response, ProductInfo.class);
  }
}
```

 - 批量功能：HystrixObservableCommand
```java
public class GetProductInfosCommand extends HystrixObservableCommand<ProductInfo> {
  private String[] productIds;

  public GetProductInfosCommand(String[] productIds) {
    super(HystrixCommandGroupKey.Factory.asKey("GetProductInfoGroup"));
    this.productIds = productIds;
  }

  @Override
  protected Observable<ProductInfo> construct() {
    return Observable.create(new Observable.OnSubscribe<ProductInfo>() {

      public void call(Subscriber<? super ProductInfo> observer) {
        try {
          for(String productId : productIds) {
            String url = "http://127.0.0.1:8082/getProductInfo?productId=" + productId;
            String response = HttpClientUtils.sendGetRequest(url);
            ProductInfo productInfo = JSONObject.parseObject(response, ProductInfo.class);
            observer.onNext(productInfo);
          }
          observer.onCompleted();
        } catch (Exception e) {
          observer.onError(e);
        }
      }

    }).subscribeOn(Schedulers.io());
  }
}
```
### 4.2.信号量
 - 信号量(Semaphore)
```
public class GetCityNameCommand extends HystrixCommand<String> {
  private Long cityId;
  public GetCityNameCommand(Long cityId) {
    super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("GetCityNameGroup"))
        .andCommandKey(HystrixCommandKey.Factory.asKey("GetCityNameCommand"))
        .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("GetCityNamePool"))
        .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
            .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)
            .withExecutionIsolationSemaphoreMaxConcurrentRequests(15)));
    this.cityId = cityId;
  }

  @Override
  protected String run() throws Exception {
    return LocalCache.getCityName(cityId);
  }
}
```
### 4.3.区别
线程池主要用于解决服务间通信故障问题即(http/rpc)等请求处理。而信号量主要用于解决自身服务之间功能的处理。
 - 功能调用

```java
	@RequestMapping("/getProductInfo")
	public ProductInfo getProductInfo(Long productId) {
		// 拿到一个商品id
		// 调用商品服务的接口，获取商品id对应的商品的最新数据
		// 用HttpClient去调用商品服务的http接口
		HystrixCommand<ProductInfo> getProductInfoCommand = new GetProductInfoCommand(productId);
		ProductInfo productInfo = getProductInfoCommand.execute();
		Long cityId = productInfo.getCityId();
		GetCityNameCommand getCityNameCommand = new GetCityNameCommand(cityId);
		String cityName = getCityNameCommand.execute();
		productInfo.setCityName(cityName);
		System.out.println(productInfo);
		return productInfo;
	}
```
 - model

```java
public class ProductInfo {
  private Long id;
  private String name;
  private Double price;
  private String pictureList;
  private String specification;
  private String service;
  private String color;
  private String size;
  private Long shopId;
  private String modifiedTime;
  private Long cityId;
  private String cityName;
  private Long brandId;
  private String brandName;
}
```

## 5.关于
 - Github: [https://github.com/liangliang1259/common-notes](https://github.com/liangliang1259/common-notes)
 - 公众号
![](https://tva1.sinaimg.cn/large/007S8ZIlly1giznpxhgdvj3076076gm3.jpg)