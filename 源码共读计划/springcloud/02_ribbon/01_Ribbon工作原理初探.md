## 前言
 - 重要API

## 1.核心API
 - RestClient
 - ILoadBalancer
 - IRule
 - IPing


## 2.使用步骤
### 2.1.构造RestTemplate

```java
@Bean
@LoadBalanced
public RestTemplate getRestTemplate(){
    return new RestTemplate();
}
```
### 2.2.使用
```java
@GetMapping("/greeting/{name}")
public String greeting(@PathVariable("name")String name){
    return restTemplate.getForObject("http://order/sayHello/"+name,String.class);
}
```


## 3.工作流程

### 3.1.流程图
[流程图地址](https://www.processon.com/view/link/5f3b9996e401fd0be0357c98)

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ghv389kwlxj30t20jngnz.jpg)

### 3.2.流程图详解
> User服务通过RestTemplate调用Order服务

 - 1.RestTemplate通过Ribbon做负载均衡  
 - 2.Ribbon从EurekaClient中获取到Order服务的注册表信息
 - 3.Ribbon获取到服务表信息之后，通过ILoadBalancer将服务列表传递给IRule的实现接口，IRule的默认实现类会通过轮询的方式获取Order服务列表中的某个节点
 - 4.Ribbon通过Http组件和该节点进行服务通信