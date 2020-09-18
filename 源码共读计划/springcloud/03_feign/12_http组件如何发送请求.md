## 前言
> 78，本篇基于上篇的内容进行扩展，研究下真正发送请求的处理


## 代码示例
>AbstractLoadBalancerAwareClient.executeWithLoadBalancer()
```java
    new ServerOperation<T>() {
        @Override
        public Observable<T> call(Server server) {
            //生成最终的url
            URI finalUri = reconstructURIWithServer(server, request.getUri());
            S requestForServer = (S) request.replaceUri(finalUri);
            try {
                //调用`FeignLoadBalancer`execute()方法
                return Observable.just(AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig));
            } 
            catch (Exception e) {
                return Observable.error(e);
            }
        }
    })
    .toBlocking()
    .single();
```

### 2.2.真实发送
```java
	public RibbonResponse execute(RibbonRequest request, IClientConfig configOverride)
			throws IOException {
		Request.Options options;
		if (configOverride != null) {
			RibbonProperties override = RibbonProperties.from(configOverride);
			options = new Request.Options(override.connectTimeout(this.connectTimeout),
					override.readTimeout(this.readTimeout));
		}
		else {
			options = new Request.Options(this.connectTimeout, this.readTimeout);
		}
        //此处完成真正的url调用，包含了connect以及readTimeout的时间
		Response response = request.client().execute(request.toRequest(), options);
		return new RibbonResponse(request.getUri(), response);
	}
```