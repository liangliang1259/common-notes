## 前言
> 76
基于上篇文章分析了，Feign获取`LoadBalancerFeignClient`的过程，本篇来进行分析Feign基于LoadBalancer如何选择一台server  
 - 这里重点分析该方法中的`.executeWithLoadBalancer(ribbonRequest, requestConfig)`
```java
	@Override
	public Response execute(Request request, Request.Options options) throws IOException {
		try {
			URI asUri = URI.create(request.url());
			String clientName = asUri.getHost();
			URI uriWithoutHost = cleanUrl(request.url(), clientName);
			FeignLoadBalancer.RibbonRequest ribbonRequest = new FeignLoadBalancer.RibbonRequest(
					this.delegate, request, uriWithoutHost);

			IClientConfig requestConfig = getClientConfig(options, clientName);
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

## 2.代码示例
> AbstractLoadBalancerAwareClient.executeWithLoadBalancer()

### 2.1.代码实验
```java
    public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) throws ClientException {
        //整合http组件，具有发送请求的功能
        LoadBalancerCommand<T> command = buildLoadBalancerCommand(request, requestConfig);

        try {
            return command.submit(
                new ServerOperation<T>() {
                    //该方法即为发送http请求的方法，此处构造了一个url地址，基于底层的http组件进行通信
                    @Override
                    public Observable<T> call(Server server) {
                        URI finalUri = reconstructURIWithServer(server, request.getUri());
                        S requestForServer = (S) request.replaceUri(finalUri);
                        try {
                            return Observable.just(AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig));
                        } 
                        catch (Exception e) {
                            return Observable.error(e);
                        }
                    }
                })
                .toBlocking()
                .single();
        } catch (Exception e) {
            Throwable t = e.getCause();
            if (t instanceof ClientException) {
                throw (ClientException) t;
            } else {
                throw new ClientException(e);
            }
        }
        
    }
 ```   
### 2.2.LoadBalancerCommand
> 该类选取出一个server， 

 - `LoadBalancerCommand.submit()`：该方法中调用了`selectServer()`方法

```java
   private Observable<Server> selectServer() {
        return Observable.create(new OnSubscribe<Server>() {
            @Override
            public void call(Subscriber<? super Server> next) {
                try {
                    //此处完成了server的负载，从context中基于负载均衡策略完成了server的选举。
                    Server server = loadBalancerContext.getServerFromLoadBalancer(loadBalancerURI, loadBalancerKey);   
                    next.onNext(server);
                    next.onCompleted();
                } catch (Exception e) {
                    next.onError(e);
                }
            }
        });
    }
```  
## 3.执行流程





## 4.下集预告
下篇会讲解如何将请求发送出去