## Spring Cloud
微服务：即将原本布设在单一机器上的服务（可理解为某个Spring Boot项目），拆解成不同的服务（一般是按照业务逻辑进行拆解），布设在不同的地方。
### Eureka 微服务注册中心
微服务注册中心，将不同的微服务注册使用，包含其地址信息
#### 配制信息
##### Eureka 服务器
1. 需要在`pom.xml`中添加`spring-cloud-starter-netflix-eureka-server`依赖
2. `application.yml`中添加配制信息
``` yml
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false #是否注册到服务器，由于其自身是服务器，所以是false
    fetchRegistry: false #是否从服务器获取注册信息，由于其自身是服务器，所以是false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #服务器地址

spring:
  application:
    name: eureka-server #微服务名
```
##### Eureka客户端
1. 需要在`pom.xml`中添加`spring-cloud-starter-netflix-eureka-client`依赖
2. `application.yml`中添加配制信息
```yml
spring:
  application:
    name: product-data-service
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

```
### Ribbon 负载均衡
默认的使用RoundRobin轮询算法。首先Ribbon会从 Eureka Client里面获取到对应的服务注册表,也就知道了所有的服务都部署在了那台机器上,在监听哪些端口,然后Ribbon就可以使用默认的Round Robin算法,从中选择一台机器,Feigin就会针对这些机器构造并发送请求。


### Feign动态代理
如果你对某个接口定义了@FeignClient注解，Feign就会针对这个接口创建一个动态代理接着你要是调用那个接口，本质就是会调用 Feign创建的动态代理，这是核心中的核心Feign的动态代理会根据你在接口上的@RequestMapping等注解，来动态构造出你要请求的服务的地址最后针对这个地址，发起请求、解析响应。

### Hystrix熔断
使每个线程池只负责单一的服务，这样当某一服务不可用时，不会导致整个服务雪崩。当某个服务在一定时间内持续不可用时，直接进行熔断并降级（记录未进行的操作，后续手工进行）。

### Zuul网关
将不同REST请求转发至不同的微服务提供者，其作用类似于Nginx的反向代理。同时，也起到了统一端口的作用，将很多微服务提供者的不同端口统一到了Zuul的服务端口。并且还提供token认证和限流等功能。

## Spring Cloud Alibaba

??restTemoplate的使用
使用Rundashboard管理多服务启动
要注意版本依赖关系:
注意最新版springcloud的pom引入方式；
初始化向导:start.aliyun.com
### Nacos注册中心
Nacos Discover
服务注册
服务心跳
服务同步
服务发现
服务健康检查

需要单独下载启动，配置启动模式等

yml文件中的nacos配置信息：