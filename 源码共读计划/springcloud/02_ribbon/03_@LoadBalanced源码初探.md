
## 1.看图说话
![](https://tva1.sinaimg.cn/large/007S8ZIlly1ghvuxl75hyj32q70u0n4k.jpg)

## 2.流程图详解
> User服务向Order服务发起接口调用

 - 1.调用者通过RestTemplate发起请求`http://order/sayHello/leo`
 - 2.根据@LoadBalanced知晓在LoadBalancerAutoConfiguration配置中是由拦截器来处理相关接口发送
 - 3.拦截器将请求转交给http调用的client `RibbonLoadBalancerClient `
 - 4.client获取server实例信息，调用Ribbon组件ILoadBalancer获取server list
 - 5.ILoadBalancer从Eureka Client中获取到server List,并通过IRule算法获取到一个实例`192.168.1.1:8080`
 - 6.RibbonLoadBalancerClient 替换url，将`http://order/sayHello/leon`替换为`http://192.168.1.1:8080/sayHello/leo`
 - 7.调用真正的httpclient组件执行接口调用
## 3.RestTemplate定制化组件
```java
@Bean
@ConditionalOnMissingBean
public RestTemplateCustomizer restTemplateCustomizer(
        final LoadBalancerInterceptor loadBalancerInterceptor) {
    return restTemplate -> {
        List<ClientHttpRequestInterceptor> list = new ArrayList<>(
                restTemplate.getInterceptors());
        list.add(loadBalancerInterceptor);
        restTemplate.setInterceptors(list);
    };
}
```
这里给`RestTemplate`添加了一个拦截器，由拦截器去处理相关的发送
