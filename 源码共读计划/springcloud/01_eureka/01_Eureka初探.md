## 前言
学习本课程之前，希望需要具备如下知识，Eureka应用。能够将client和server通过eureka完成注册，并能够完成应用的简单调用
### 课程目标
本篇课程作为Eureka系列的入门热身，希望大家能够掌握和了解如下内容。
 - 了解Eureka整体的结构
 - 了解Eureka中的一些概念及名词
 - 了解Eureka各个组件之间的交互关系以及流程
## 目录
 - 概述
 - Eureka架构图
 - 请求流程图
 - 代码说明
 - 图说流程
## 1.概述
> Eureka的一些概念
 - 服务提供者(Provider)：我们这里可以把它称作服务提供者或者叫做server。当然这里只是一个相对的概念，如服务A可以作为server给服务B提供服务，同时服务B也可以作为server给A提供服务。`一个springboot应用，通过注解标明`
 - 服务调用者(Consumer): 作为服务使用者，调用Provider端的接口，返回数据进行逻辑处理，`一个springboot应用，通过注解标明`,这里我们吧`Provider`和`Consumer`都称作Eureka的client。因为它们续约将本身作为client注册到Eureka上。而`Eureka`服务在这里称之为Eureka server.
 - 服务注册(Rigister)：服务(包括Server和Client)启动时，会将本身注册到Eureka Server上，同时会将一些数据信息发送到Eureka Server上(包含ip，端口，服务名等)
 - 服务续约(Renew)：这个概念是在注册中心中比较常见的，不仅仅是在Eureka中。该功能通过心跳机制实现，每间隔30s会向Eureka发送一个信息通知Eureka，自己还在运行。若长时间未续约，则该服务会被剔除。具体源码后续会进行分析
 - 服务下线(Eviction):若Eureka client长时间未向Eureka Server发送心跳，则会将该节点从Eureka 中剔除。
## 2.Eureka架构图
> Eureka是满足`CAP`理论中的`AP`，无法保障一致性，因为对于Eureka来说每个节点都是等价的。Eureka集群之间，每个节点会作为client向其他节点发送信息。这块内容后续会详细解释，本篇先作为入门热身。

[图地址](https://processon.com/diagraming/5f2d5dd55653bb1b61174f08)
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gizjhgyqryj316e0ry76u.jpg)
 - Eureka
## 3.代码实践
### 3.1.Eureka Server
#### pom配置
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
```
#### 引入注解
```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```
#### 配置文件
```properties
# 服务名
spring.application.name= eureka
#端口
server.port=7777
# 是否注册到eureka（eureka本身是不需要再注册到自己的）
eureka.client.register-with-eureka=false
# 是否从eureka获取注册信息
eureka.client.fetch-registry=false
# eureka服务器的地址（注意：地址最后面的 /eureka/ 这个是固定值）
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
#服务失效时间，Eureka多长时间没收到服务的renew操作，就剔除该服务，默认90秒
eureka.instance.leaseExpirationDurationInSeconds=15
#eureka server清理无效节点的时间间隔，默认60000毫秒，即60秒
eureka.server.evictionIntervalTimerInMs=20000
# 自我保护模式（缺省为打开）
eureka.server.enable-self-preservation: true
# 续期时间，即扫描失效服务的间隔时间（缺省为60*1000ms）
eureka.server.eviction-interval-timer-in-ms: 5000 
```
### 3.2.服务提供者(Provider)
> 这里我们模拟一个用户服务调用订单服务的功能。将`user`作为访问调用者，`order`作为服务提供者
#### pom依赖
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>
    </dependencies>

```
#### 注解
```java
@EnableDiscoveryClient
@SpringBootApplication
public class OrderApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(OrderApplication.class, args);
    }
}
```
#### 配置文件

```properties
server.port=8088
spring.application.name=order
#================================eureka配置==============================
#注册到eureka中心，获取到配置服务
eureka.client.service-url.defaultZone=http://localhost:7777/eureka/
#设置实例的ID为ip:port
#================================续约配置============================
# 心跳时间，即服务续约间隔时间（缺省为30s）
eureka.instance.lease-renewal-interval-in-seconds=5
# 发呆时间，即服务续约到期时间（缺省为90s）
eureka.instance.lease-expiration-duration-in-seconds=10
# 开启健康检查（依赖spring-boot-starter-actuator）
#eureka.client.healthcheck.enabled=true
#eureka.instance.prefer-ip-address=true
eureka.instance.instance-id=${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}

```


### 3.3.服务调用者(Consumer)

#### pom依赖
> 这里包含了后续Feign的配置，可以先添加进来
  ```xml
      <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.2.4.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
    </dependencies>
  ```
#### 注解

```java
@EnableDiscoveryClient
@SpringBootApplication
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
}
```

#### 配置文件
```properties
server.port=8086
#eureka.client.serviceUrl.defaultZone=http://localhost:7777/eureka/ # eureka server的地址

spring.application.name=user
#================================eureka配置==============================
#注册到eureka中心，获取到配置服务
eureka.client.service-url.defaultZone=http://localhost:7777/eureka/
#设置实例的ID为ip:port
#================================续约配置============================
# 心跳时间，即服务续约间隔时间（缺省为30s）
eureka.instance.lease-renewal-interval-in-seconds=5
# 发呆时间，即服务续约到期时间（缺省为90s）
eureka.instance.lease-expiration-duration-in-seconds=10
# 开启健康检查（依赖spring-boot-starter-actuator）
#eureka.client.healthcheck.enabled=true
#eureka.instance.prefer-ip-address=true
eureka.instance.instance-id=${spring.cloud.client.ip-address}:${spring.application.name}:${server.port}
```
## 4.图说流程
> 一张图说明上述几个服务之间的流程。
![](https://tva1.sinaimg.cn/large/007S8ZIlly1giznd0amuaj31ii0iqgo1.jpg)
 - 1.Eureka启动集群
 - 2.user服务和order服务启动时，需要将自己注册到Eureka Server上。同时会把user和order相关节点的ip，port，serviceName等传递给Eureka上，并获取到注册信息。
 - 3.`User`服务调用`Order`服务时，会从Eureka中获取到一个对应的Order服务的List<Instance>,其中Instance包含了ip，port，sericeName等信息。
 - 4.通过一些策略同上述的list中选中一个节点，通过http完成访问的调用。

> 访问Eureka服务获取Eureka页面，可以查看所有注册到Eureka Server上的一些服务的信息。
![](https://tva1.sinaimg.cn/large/007S8ZIlly1giznihsn34j31xx0u00yn.jpg)
## 关于
欢迎关注本人公众号，不定时更新文章
![](https://tva1.sinaimg.cn/large/007S8ZIlly1giznpxhgdvj3076076gm3.jpg)
