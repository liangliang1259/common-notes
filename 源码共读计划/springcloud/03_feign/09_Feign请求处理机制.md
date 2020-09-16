## 前言
> 74
SynchronousMethodHandler.executeAndDecode()

## 2.代码分析
> 此处未Feign处理的核心方法
```java
  Object executeAndDecode(RequestTemplate template, Options options) throws Throwable {
    //遍历拦截器，将每个拦截器应用到RequestTemplate上去，基于拦截器构建出一个新的Request。构造url
    //url:/users/1--->http://user/users/1：添加服务名
    Request request = targetRequest(template);

    if (logLevel != Logger.Level.NONE) {
      logger.logRequest(metadata.configKey(), logLevel, request);
    }

    Response response;
    long start = System.nanoTime();
    try {
        //LoadBalancerFeignClient处理请求，
      response = client.execute(request, options);
      // ensure the request is set. TODO: remove in Feign 12
      response = response.toBuilder()
          .request(request)
          .requestTemplate(template)
          .build();
    } catch (IOException e) {
      if (logLevel != Logger.Level.NONE) {
        logger.logIOException(metadata.configKey(), logLevel, e, elapsedTime(start));
      }
      throw errorExecuting(request, e);
    }
    long elapsedTime = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - start);


    if (decoder != null)
      return decoder.decode(response, metadata.returnType());

    CompletableFuture<Object> resultFuture = new CompletableFuture<>();
    asyncResponseHandler.handleResponse(resultFuture, metadata.configKey(), response,
        metadata.returnType(),
        elapsedTime);

    try {
      if (!resultFuture.isDone())
        throw new IllegalStateException("Response handling not done");

      return resultFuture.join();
    } catch (CompletionException e) {
      Throwable cause = e.getCause();
      if (cause != null)
        throw cause;
      throw e;
    }
  }
```


### 2.2.处理请求:LoadBalancerFeignClient.execute()
```java
	public Response execute(Request request, Request.Options options) throws IOException {
		try {
			URI asUri = URI.create(request.url());
            //获取hostname，即Order服务
			String clientName = asUri.getHost();
            //构造出url，去除了服务名称
        	URI uriWithoutHost = cleanUrl(request.url(), clientName);
            //创建RibbonRequest
			FeignLoadBalancer.RibbonRequest ribbonRequest = new FeignLoadBalancer.RibbonRequest(
					this.delegate, request, uriWithoutHost);
            //Ribbon相关的一些配置
			IClientConfig requestConfig = getClientConfig(options, clientName);
            //核心功能:此处是从CachingSpringLoadBalancerFactory中进行处理
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
#### 1. CachingSpringLoadBalancerFactory
> 此处功能是获取LoadBalancer的核心功能

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gilawicyjpj30hn0bqwf3.jpg)


下一节研究feign、ribbon、eureka是如何整合起来的？