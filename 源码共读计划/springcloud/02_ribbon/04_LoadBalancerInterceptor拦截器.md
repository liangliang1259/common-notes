# 前言
上一节，我们学习了`LoadBalancerAutoConfiguration`,我们了解到`RestTemplate`通过`RestTemplateCustomizer`定制化，给`RestTemplate`增加了一个拦截器`LoadBalancerInterceptor`，这一节我们就来看下。
# 1.流程图
![](https://tva1.sinaimg.cn/large/007S8ZIlly1ghxd99knx6j30ry0xetam.jpg)
## 2.流程详解
## 2.1.拦截器
> LoadBalancerInterceptor
```java
@Override
public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
        final ClientHttpRequestExecution execution) throws IOException {
    final URI originalUri = request.getURI();
    String serviceName = originalUri.getHost();
    Assert.state(serviceName != null,
            "Request URI does not contain a valid hostname: " + originalUri);
    return this.loadBalancer.execute(serviceName,
            this.requestFactory.createRequest(request, body, execution));
}
```
这里从`request`中获取到`serviceName`,并将请求交给`LoadBalancerClient`去执行

## 2.2.LoadBalancerClient
> RibbonLoadBalancerClient
这里是真正的处理逻辑，下一讲进行剖析


## 3.整体流程
1.`LoadBalancerInterceptor`拦截`RestTemplate`,从url中获取到hostname
2.然后通过`LoadBalancerClient`去执行所有请求。