## 前言
上一篇文章已经找到了`LoadBalancer`的处理是由`ZoneAwareLoadBalancer`实现。这一节我们来看下`ZoneAwareLoadBalancer`的相关流程。
## 看图说话
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gi2uewtgwuj30qw0mlmys.jpg)
1.`RibbonLoadBalancerClient`根据`ZoneAwareLoadBalancer`获取server List
2.`ZoneAwareLoadBalancer`在构造函数中调用父类构造函数`DynamicServerListLoadBalancer`,核心逻辑为`restOfInit()`内部的`serverListImpl.getUpdatedListOfServers() `实现。
3.`serverListImpl.getUpdatedListOfServers()`的真实实现类是`DiscoveryEnabledNIWSServerList`,在该类中找到根据eureka CLient获取到server list的代码
4.将上一步获取到的server list填充到`BaseLoadBalancer`中去，并返回给上一步


