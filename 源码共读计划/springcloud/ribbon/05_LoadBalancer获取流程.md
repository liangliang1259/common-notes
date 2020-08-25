# 前言
 - RibbonLoadBalancerClient核心

## 2.核心处理流程
> 根据服务名`order`如何找到默认的`oadBalancer`
### 2.1.入口
```java
protected ILoadBalancer getLoadBalancer(String serviceId) {
    return this.clientFactory.getLoadBalancer(serviceId);
}
```
### 2.2.SpringClientFactory
> 基于Spring进行的一些封装扩展，从Spring中获取一些bean

一个服务对应一个ApplicationContext，获取`ILoadBalancer`

1.ILoadBalancer
serviceId = `order`
SpringClientFactory.getLoadBalancerContext(serviceId);

SpringClientFactory.getLoadBalancerContext(serviceTd,RibbonLoadBalancerContext.class)
