## 前言
> 75
### 入口类
> LoadBalancerFeignClient--->CachingSpringLoadBalancerFactory.create();
 - 1.LoadBalancerFeignClient.execute()
```java
	public Response execute(Request request, Request.Options options) throws IOException {
		try {
			URI asUri = URI.create(request.url());
			String clientName = asUri.getHost();
			URI uriWithoutHost = cleanUrl(request.url(), clientName);
			FeignLoadBalancer.RibbonRequest ribbonRequest = new FeignLoadBalancer.RibbonRequest(
					this.delegate, request, uriWithoutHost);

			IClientConfig requestConfig = getClientConfig(options, clientName);
            //此处需要重点关注lbClient方法
			return lbClient(clientName)
					.executeWithLoadBalancer(ribbonRequest, requestConfig).toResponse();
		}
		catch (ClientException e) {
			IOException io = findIOException(e);
			if (io != null) {
				throw io;
			}
			throw new RuntimeException(e);
		}
	}
```
 - lbClient(String clientName) : 处理功能
```java
	private FeignLoadBalancer lbClient(String clientName) {
        //CachingSpringLoadBalancerFactory，从该类中处理，主要用于创建出一个FeignLoadBalancer
		return this.lbClientFactory.create(clientName);
	}
```

## create()方法
此处用于构造`FeignLoadBalancer`,使用`SpringClientFactory`来构造
