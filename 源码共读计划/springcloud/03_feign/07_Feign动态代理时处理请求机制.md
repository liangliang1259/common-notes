## 前言

 - 1.FeignClientFactoryBean.loadBalance()
```java
Targeter targeter = get(context, Targeter.class);
```
 - 2.DefaultTargeter.target()
```java
feign.target(target);
```
 - 3.Feign.target()
```java

```
 - 4.ReflectiveFeign.newInstance()
```java
//JDK动态代理实现
Proxy.newProxyInstance()
```