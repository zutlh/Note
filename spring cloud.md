# 1.1 SOA与微服务的关系

**SOA**面向服务架构，他是一种设计方法，其中包含多个服务，服务之间通过相互依赖最终提供一系列的功能。一个服务通常以独立的形式存在于操作系统进程中。各个服务之间通过网络调用。

**微服务架构**:其实和SOA类似，微服务是在SOA上做的升华，微服务架构强调的一个重点是“业务需要彻底的组件化和服务化”，原有的单个业务系统会拆分为多个可以独立开发、设计、运行的小应用。这些小应用之间通过服务完成交互和集成。

| 功能     | SOA                  | 微服务                       |
| -------- | -------------------- | ---------------------------- |
| 组件大小 | 大块业务逻辑         | 单独任务或小块业务逻辑       |
| 耦合     | 通常松耦合           | 总是松耦合                   |
| 公司架构 | 任何类型             | 小型、专注于功能交叉团队     |
| 管理     | 着重中央管理         | 着重分散管理                 |
| 目标     | 确保应用能够交互操作 | 执行新功能、快速拓展开发团队 |

# 1.2 远程调用技术

**RPC:**

![WeChatc6d323301f33d636073a6c0d5af7f99d](/Users/apple/Library/Containers/com.tencent.xinWeChat/Data/Library/Caches/com.tencent.xinWeChat/2.0b4.0.9/1f989e79eb4808b23080e898a2a556e4/dragImgTmp/WeChatc6d323301f33d636073a6c0d5af7f99d.png)

**http**

RESTful

| 比较     | RESTful    | RPC         |
| -------- | ---------- | ----------- |
| 通讯协议 | HTTP       | 一般使用TCP |
| 性能     | 略低       | 较高        |
| 灵活度   | 高         | 低          |
| 应用     | 微服务架构 | SOA架构     |

# 1.3 Spring Cloud

**Spring Cloud Netflix组件**

| 组件名称 | 作用           |
| -------- | -------------- |
| Eureka   | 服务注册中心   |
| Ribbon   | 客户端负载均衡 |
| Feign    | 声明式服务调用 |
| Hystrix  | 客户端容错保护 |
| Zuul     | API服务网关    |

# 2 服务注册Eureka

## 2.1 微服务的注册中心

注册中心可以说是微服务架构中的“通讯录”，它记录了服务和服务地址的映射关系。在分布式架构中，服务会注册到这里，当服务需要调用其他服务时，就在这里找到服务地址，进行调用。

## 2.1.1 注册中心的主要作用

注册中心在微服务架构中主要起到了协调者的一个作用。注册中心一般包含如下几个功能：

1、服务发现：服务注册/反注册 服务订阅/取消订阅 服务路由

2、服务配置：配置订阅 配置下发

3、服务健康检测

## 2.1.2 常见的注册中心

zookeeper

eureka

consul

nacos

![](/Users/apple/Desktop/WX20210817-112204@2x.png)

### 使用Eureka

1、搭建eureka server

​	1.1、创建工程

​	1.2、导入坐标

​	1.3、配置application.yml

​	1.4、配置启动类

2、将服务提供者注册到eureka server上

​	2.1、引入eureka相应的左表

​	2.2、修改application.yml添加eureka server的信息

​	2.3、修改启动类 添加服务发现的支持

3、服务消费者通过注册中心获取服务列表，并调用

​	eureka中的元数据：服务的主机名，IP地址等信息，可以通过eureka server进行获取，用于服务之间调用



### eureka集群、高可用的引入

![](/Users/apple/Desktop/企业微信20210817165148.png)

1、准备2个eureka server，需要相互注册

​	1 号server：9000

​	2 号server：8000

2、需要将微服务注册到两个eureka server上

### 细节问题

#### 在控制台显示服务IP

在服务提供者，通过eureka.instance.instance-id配置控制台显示服务IP

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:9000/eureka/
  instance:
    prefer-ip-address: true # 使用ip地址注册
    instance-id: ${spring.cloud.client.ip-address}:${server.port} #想注册中心中注册服务id
```

#### eureka的服务剔除问题

默认每隔30s发送心跳，如果90s内没有发送心跳代表宕机

在服务的提供者，设置心跳间隔，设置续约到期时间

#### eureka的自我保护机制

在eureka中配置关闭自我保护和剔除服务间隔

```yml
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false #是否将自己注册到注册中心
    fetch-registry: false #是否要从eureka中获取注册信息
    service-url: #配置暴露给Eureka Client的请求地址
      defalutZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  server:
    enable-self-preservation: false # 关闭自我保护机制
    eviction-interval-timer-in-ms: 4000 # 设置剔除服务间隔
  
```

## 2.2 eureka源码解析

### 2.2.1 SpringBoot中的自动装载

#### (1) ImportSelector

ImportSelector接口是Spring导入外部配置的核心接口，在SpringBoot的自动化配置和@EnableXXX(功能性注解)中起到了决定性作用。当在@Configuration标注的Class上使用@Import引入了一个ImportSelector实现类后，会把实现类中返回的Class名称都定义为Bean。

![企业微信20210820162313](/Users/apple/Desktop/企业微信20210820162313.png)

#### (2) EnableEurekaServer

![企业微信20210823094837](/Users/apple/Desktop/企业微信20210823094837.png)

# 3. 服务调用Ribbon

经过以上的学习，已经实现了服务的注册和服务发现，当启动某个服务时，可以通过HTTP的形式将信息注册到注册中心，并且可以通过SpringCloud提供的工具获取注册中心的服务列表。但是服务之间的调用还存在很多的问题，如何更加方便的调用微服务，多个微服务的提供者如何选择，如何均衡负载等。

## 3.1 Ribbon概述

### 3.1.1 什么是Ribbon

是netflix发布的一个负载均衡器，有助于控制HTTP和TCP客户端行为。在SpringCloud中，Eureka一般配合Ribbon进行使用，Ribbon提供了客户端负载均衡的功能，Ribbon利用从Eureka中读取到的服务信息，在调用服务节点提供的服务时，会合理的进行负载。

在SpringCloud中可以将注册中心和Ribbon配合使用，Ribbon自动的从注册中心获取服务提供者的列表信息，并基于内置的负载均衡算法，请求服务。

### 3.1.2 Ribbon的主要作用

#### (1) 服务调用

基于Ribbon实现服务调用，是通过拉取到的所用服务列表组成(服务名-请求路径的)映射关系。借助RestTemplate最终进行调用。

#### (2) 负载均衡

当有多个服务提供者时，Ribbon可以根据负载均衡的算法自动的选择需要调用的服务地址。

Ribbon是一个典型的客户端负载均衡器。Ribbon会获取服务的所有地址，根据内部的负载均衡算法，获取本次请求的有效地址。

准备两个商品微服务(9001),(9011)

在订单系统中远程以负载均衡的形式调用商品服务

#### (3) 负载均衡策略

```java
com.netflix.loadbalancer.RoudRobinRule  //以轮询的方式进行负载均衡
com.netflix.loadbalancer.RandomRule //随机策略
com.netflix.loadbalancer.RetryRule //重试策略
com.netflix.loadbalancer.weightedResponseTimeRule //权重策略，会计算每个服务的权重，越高的调用的可能性越大
com.netflix.loadbalancer.BestAvailableRule //最佳策略，遍历所有的服务实例，过滤掉故障实例，并返回请求数最小的实例返回
com.netflix.loadbalancer.AvailabilityFilteringRule //可用过滤策略。过滤掉故障和请求数超过阈值的服务实例，再从剩下的实例中轮询调用
```

在服务消费者的application.yml配置文件中修改负载均衡策略

```yml
# 修改ribbon的负载均衡策略  服务名 - ribbon - NFLoadBalancerRuleClassName : 策略
product-service:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

#### (4) 重试机制

引入spring的重试组件

```xml
 <dependency>
            <groupId>org.springframework.retry</groupId>
            <artifactId>spring-retry</artifactId>
 </dependency>
```

对ribbon进行重试配置

```yml
product-service:
  ribbon:
#    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
    ConnectTimeout: 250 # Ribbon的连接超时时间
    ReadTimeout: 1000 # Ribbon的数据读取超时时间
    OkToRetryOnAllOperation: true # 是否对所有操作都进行重试
    MaxAutoRetriesNextServer: 1 # 切换实例的重试次数
    MaxAutoRetries: 1 # 对当前实例的重试次数
```

#### (5) Ribbon源码解析

![企业微信20210826114004](/Users/apple/Desktop/企业微信20210826114004.png)

# 4.Consul

特性：

服务发现

健康检查

K/V存储

多数据中心

### 4.1.1 consul和eureka区别

#### (1) 一致性

consul强一致性(CP)

​	服务注册相比eureka会稍慢一些。因为consul的raft协议要求必须过半数的结点都写入成功才认为注册成功

​	leader挂掉时，重选期间整个consul不可用，保证了强一致性但牺牲了可用性

eureka保证高可用和最终一致性(AP)

​	服务注册相对较快

​	当数据出现不一致时，虽然A,B上的注册信息不完全相同，但每个Eureka节点依然能够正常对外提供服务，这会出现查询服务信息时如果请求A查不到，但请求B就能查到，保证了可用性但牺牲了一致性

#### (2) 开发语言和使用

eureka就是个servlet程序，跑在servlet容器中

consul则是go编写而成，安装启动即可

# 5.服务调用Feign

## 5.1 Feign简介

Feign是netfilx开发的声明式，模板化的HTTP客户端，其灵感来自Retrofit, JAXR-2.0以及WebSocket

​	Feign可以帮助我们更加便捷，优雅的调用HTTP

​	在SpringCloud中，使用Feign非常简单--创建一个接口，并在接口上添加一些注解，代码就完成了。

​	Feign支持多种注解，例如Feign自带的注解或者JAX-RS注解等

​	SpringCloud对Feign进行了增强，使Feign支持了SprimgMVC注解，并整合了Ribbon和Eureka，从而让Feign的使用更加方便

​	Feign是在Ribbon的基础上进行了一次改进，是一个使用起来更加方便的HTTP客户端。采用接口的方式，只需要创建一个接口，在上面添加注解即可，将需要调用的其他服务的方法定义成抽象方法即可，不需要自己构建HTTP请求，然后就像是调用自身工程的方法调用，而感觉不到是调用远程方法，使得编写客户端变得非常容易

## 5.2 基于Feign的服务调用

#### (1) 引入依赖

在服务消费者shop_order-service添加Feign依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>

```

#### (2) 配置调用接口

```java
/**
 * 声明需要调用的微服务名称
 * @FeignClient
 *      * name: 服务提供者的名称
 *
 */
@FeignClient(name = "product-sercvice")
public interface ProductFeignClient {
    /**
     * 配置需要调用的微服务接口
     */
    @RequestMapping(value = "/product/{id}",method = RequestMethod.GET)
    public Product findById(@PathVariable("id") Long id);
}
```



#### (3) 在启动类激活Feign

```java
@SpringBootApplication
@EntityScan("cn.itcast.order.entity")
@EnableEurekaClient
//激活feign
@EnableFeignClients
public class OrderApplication
```

#### (4) 通过自动的接口调用远程服务

```java
    @Autowired
    private ProductFeignClient productFeignClient;

    @RequestMapping(value = "/buy/{id}",method = RequestMethod.GET)
    public Product findById(@PathVariable Long id){


        Product product = null;
        product = productFeignClient.findById(id);
        return product;
    }
```

## 5.3 负载均衡

Feign中本身已经集成了Ribbon依赖和自动配置，因此我们不需要额外引入依赖，也不需要再注册RestTemplate对象。

# 6 服务调用Feign高级

## 6.1 Feign的配置

从SpringCloud Edgware开始，Feign支持使用属性自定义Feign。对于一个指定名称的FeignClient，Feign支持如下配置

​	feignName: FeignClient的名称

​	connectTimeout：建立链接的超时时长

​	readTimeout：读取超时时长

​	loggerLevel：Feign的日志级别

​	errorDecoder：Feign的错误解码器

​	retryer: 配置重试

​	requestInterceptors：添加请求拦截器

​	decoder404：配置熔断不处理404异常

## 6.2 请求压缩

SpringCloud Feign支持对请求和响应进行GZIP压缩，以减少通信过程中的性能损耗，通过下面的参数即可开启请求与响应的压缩功能：

```yml
feign:
	compression:
		request:
			enabled: true #开启请求压缩
		response:
			enabled: true #开启响应压缩
```

## 6.3 日志级别

在开发或者运行阶段往往希望看到Feign请求过程的日志记录，默认情况下Feign的日志是没有开启的。要想用属性配置方式来达到日志效果，只需在application.yml中添加如下内容即可

```yml
feign:
	client:
		config:
			shop-product-service:
				loggerLevel: FULL
logging:
	level:
		cn.itcast.order.feign.ProductFeignClient: debug
```

## 6.4 Feign和Ribbon的区别

Ribbon是一个客户端的负载均衡器

Feign是在Ribbon的基础上进行了封装

## 6.5 组件的使用方式

### 6.5.1 注册中心

#### (1) Eureka

​	搭建注册中心

​		引入依赖 spring-cloud-starter-netflix-eureka-server

​		配置EurekaServer

​		通过@EnableEurekaServer激活Eureka Server端配置

​	服务注册

​		服务提供者引入 spring-cloud-starter-netflix-eureka-client依赖

​		通过eureka.client.serviceUrl.defaultZone 配置注册中心地址

#### (2) consul

​	搭建注册中心

​		下载安装consul

​		启动consul consul agent -dev

​	服务注册

​		服务提供者引入spring-cloud-starter-consul-discovery

​		通过spring.cloud.consul.host和spring.cloud.consul.port指定Consul Server的请求地址

### 6.5.2 服务调用

#### (1) Ribbon

​		通过Ribbon结合RestTemplate方式进行服务调用只需要在声明RestTemplate的方法上添加注解@LoadBalanced 即可

​		可以通过{服务名称}.ribbon.NFLoadBalancerRuleClassName配置负载均衡策略

#### (2) Feign

​		服务消费者引入spring-cloud-starter-openfeign依赖

​		通过@FeignClient声明一个调用远程微服务接口

​		启动类上通过@EnableFeignClient激活Feign

## 6.1 微服务架构的高并发问题

通过注册中心已经实现了微服务的服务注册和服务发现,并且通过Ribbon实现了负载均衡，已经借助Feign可以优雅的进行微服务调用。那么我们编写的微服务的性能怎么样呢？是否存在问题？

### 6.1.1 性能测试工具Jmeter

用来进行压力测试

## 6.2 系统负载过高存在的问题

### 6.2.1 问题分析

在微服务架构中，我们将业务拆分成一个个的服务，服务与服务之间可以相互调用，由于网络原因或者自身的原因，服务并不能保证服务的100%可用，如果单个服务出现问题，调用这个服务就会出现网络延迟，此时若有大量的网络涌入，会形成任务累计，导致服务瘫痪。

在SpringBoot程序中，默认使用内置tomcat作为web服务器。单tomcat支持最大的并发请求是有限的，如果某一接口阻塞，待执行的任务积压越来越多，那么势必会影响其他接口的调用。

### 6.2.2 线程池的形式实现服务隔离



![企业微信20210830111824](/Users/apple/Desktop/企业微信20210830111824.png)

# 7.服务熔断Hystrix入门

## 7.1 服务容错的核心知识

### 7.1.1 雪崩效应

在微服务架构中，一个请求需要调用多个服务是非常常见的。如客户端访问A服务，而A服务需要调用B服务，B服务需要调用C服务，由于网络原因或者自身的原因，如果B服务或者C服务不能及时响应，A服务将处于阻塞状态，直到B服务C服务响应。此时若有大量的请求涌入，容器的线程资源会被消耗完毕，导致服务瘫痪。服务于服务之间的依赖性，故障会传播，造成连锁反应，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。

雪崩是系统中的蝴蝶效应导致其发生的原因多种多样，有不合理的容量设计，或者是高并发下某一个方法响应变慢，亦或是某台机器的资源耗尽。在源头上我们无法完全杜绝雪崩源头的发生，但是雪崩的根本原因来源于服务之间的强依赖，所以我们可以提前评估，做好**熔断，隔离，限流**

### 7.1.2 服务隔离

顾名思义，它是指将系统按照一定的原则划分为若干个服务模块，各个模块之间相对独立，无强依赖。当有故障发生时，能将问题和影响隔离在某个模块内部，而不扩散风险，不波及其他模块，不影响整体的系统服务。

### 7.1.3 熔断降级

熔断这一概念来源于电子工程中的断路器。在互联网系统中，当下游服务因访问压力过大而响应变慢或失败，上游服务为了保护系统整体的可用性，可以暂时切断对下游服务的调用。这种牺牲局部，保全整体的措施就叫熔断。![企业微信20210831095001](/Users/apple/Desktop/企业微信20210831095001.png)

所谓降级，就是当某个服务熔断之后，服务器不再被调用，此时客户端可以准备一个本地的fallback回调，返回一个缺省值，也可以理解为兜底。

### 7.1.4 服务限流

限流可以认为服务降级的一种，限流就是限制系统的输入和输出流量以达到保护系统的目的。一般来说系统的吞吐量是可以被测算的，为了保证系统的稳固运行，一旦达到需要限制的阈值，就需要限制流量并采取少量措施以完成限制流量的目的。比方：推迟解决，拒绝解决，或者部分拒绝解决等等。

## 7.2 Hystrix介绍

Hystrix是由Netflix开源的一个延迟和容错库，用于隔离访问远程系统，服务或者第三方库，防止级联失败，从而提升系统的可用性与容错性。Hystrix主要通过以下几点实现延迟和容错。

- 包裹请求：使用HystrixCommand包裹对依赖的调用逻辑，每个命令在独立线程中执行。这是用了设计模式中的“命令模式”。
- 跳闸机制：当某服务的错误率超过一定的阈值时，Hystrix可以自动或者手动跳闸，停止请求该服务一段时间。
- 资源隔离：Hystrix为每个依赖都维护了一个小型的线程池(或者信号量)。如果该线程池已满，发往该依赖的请求就被立刻拒绝，而不是派对等待，从而加速失败判定。
- 监控：Hystrix可以近乎实时地监控运行指标和配置的变化，例如成功，失败，超时，以及被决绝的请求。
- 回退机制：当请求失败、超时、被拒绝，或当断路器打开时，执行回退逻辑。回退逻辑由开发人员自行提供，例如返回一个缺省值。
- 自我修复：断路器打开一段时间后，会自动进入“半开”状态。

## 7.3 Hystrix组件

### 对RestTemplate的支持

- 引入hystrix依赖

  ```xml
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
          </dependency>
  
  ```

- 在启动类中激活hystrix

  ```java
  @EnableCircuitBreaker
  ```

- 配置熔断触发的降级逻辑

  ```java
   /**
       * 降级方法
       * 和需要收到保护的方法的返回值一致
       */
      public Product orderFallback(Long id) {
          Product product = new Product();
          product.setProductName("触发降级方法");
          return product;
      }
  ```

- 在需要收到保护的接口上使用@HystrixCommand配置

  ```java
   		@HystrixCommand
      @RequestMapping(value = "/buy/{id}", method = RequestMethod.GET)
      public Product findById(@PathVariable Long id) {
          Product product = null;
          product = restTemplate.getForObject("http://product-service/product/" + id, Product.class);
          return product;
      }
  ```



### 对Feign的支持

- 引入依赖(Feign已经集成了Hystrix)

- 在Feign中配置开启Hystrix

  ```yml
  feign:
    client:
      config:
        product-service: # 需要调用的服务名称
          loggerLevel: FULL
    hystrix:
      enabled: true
  ```

  

- 自定义一个接口的实现类，这个实现类就是熔断触发的降级逻辑

  ```java
  @Component
  public class ProductFeignClientCallback implements ProductFeignClient {
      /**
       * 熔断降级的方法
       * @param id
       * @return
       */
      public Product findById(Long id) {
          Product product = new Product();
          product.setProductName("Feign调用触发了熔断降级方法");
          return product;
      }
  }
  ```

  

- 修改FeignClient接口添加降级方法的支持

  ```java
  @FeignClient(name = "product-service",fallback = ProductFeignClientCallback.class)
  public interface ProductFeignClient {
      /**
       * 配置需要调用的微服务接口
       */
      @RequestMapping(value = "/product/{id}",method = RequestMethod.GET)
      public Product findById(@PathVariable("id") Long id);
  }
  ```

### Hystrix的超时时间

```yml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInmilliseconds: 3000 #默认的连接超时时间1秒，若1秒没有返回数据，自动的触发降级方法
          timeout:
            enabled: true #必须设置true，否则会交给Ribbon
```

当在规定时间内，没有获取到微服务的数据，这个时候会自动的触发熔断降级方法。



### Hystrix的监控平台

除了实现容错功能外，Hystrix还提供了近乎实时的监控，HystrixCommand和HystrixObservableCommand在执行时，会生成执行结果和运行指标。比如每秒的请求数量，成功数量等。这些状态会暴露在Actuator提的/health端点中。只需为项目添加spring-boot-actuator依赖，重启项目，访问http://localhost:XXXX/actuator/hystrix.stream,即可看到实时的监控数据

### Hystrix设置监控信息

引入坐标

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
            <version>2.5.4</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
            <version>2.2.9.RELEASE</version>
        </dependency>
```

在启动类上配置

```java
//激活hystrix
@EnableCircuitBreaker
```

暴露所有actuator监控的端点

```yml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

在页面上进行访问http://localhost:XXXX/actuator/hystrix.stream,即可看到实时的监控数据

搭建Hystrix DashBoard监控

刚刚讨论了Hystrix的监控，但访问/hystrix.stream接口获取的都是以文字形式展示的信息，很难通过文字直观的展示系统的运行状态，所以Hystrix官方还提供了基于图形化的DashBoard监控平台。Hystrix仪表板可以显示每个断路器(被@HystrixCommand注解的方法)的状态。

（1) 导入依赖

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
            <version>2.2.9.RELEASE</version>
        </dependency>
```

（2）添加EnableHystrixDashboard 注解

在启动类使用@EnableHystrixDashboard注解激活仪表盘项目

```java
@EnableHystrixDashboard
public class FeignOrderApplication {
```

(3) 访问测试

hystrix可以对请求失败的请求，以及被拒绝，或者超时的请求进行统一的降级处理

### 熔断器

熔断器有三种状态closed、open、half_open熔断器默认关闭状态，当触发熔断后状态变更为open，在等待到指定的时间，Hystrix会放请求检测服务是否开启，这期间熔断器会变为half_open半开启状态，熔断探测服务可用则继续变更为closed关闭熔断器。

![企业微信20210903162244](/Users/apple/Desktop/企业微信20210903162244.png)

- closed：关闭状态，所有请求都正常访问。代理类维护了最近调用失败的次数，如果其次调用失败，则使失败次数加1。如果最近失败次数超过了在给定时间内允许失败的阈值，则代理类切换到断开(open)状态。此时代理开启了一个超时时钟，当该时钟超过了该时间，则切换到半断开(half_open)状态。该超时时间的设定是给了系统一次机会来修正导致调用失败的错误。
- open：打开状态(断路器打开)，所有请求都会被降级，Hystrix会针对请求情况计数，当一定时间内请求百分比达到阈值，则触发熔断，断路器会完全关闭。默认失败比例的阈值是50%，请求次数最少不低于20次。
- half_open：半开状态，open状态不是永久的，打开后悔进入休眠时间(默认是5s)。随后断路器会自动进入半开状态。此时会释放一次请求通过，若这个请求是健康的，则会关闭断路器,否则让open状态在持续5s。



环境准备：

​	在订单系统中加入逻辑

​		判断请求的id：

​			如果id=1：正常执行（正常调用商品微服务）

​			如果id=2：抛出异常

​	2、默认hystrix中触发断路器状态转换的阈值

​		触发熔断的最小请求次数：20

​		触发熔断的请求失败的比率：50%

​		断路器开启的时长：5s

### 断路器聚合监控Turbine

在微服务架构体系中，每个服务都需要配置Hystrix DashBoard监控。如果每次只能查看单个实例的监控数据，就需要不断切换监控地址，这显然很不方便。要想看这个系统的Hystrix DashBoard数据就需要用到Hystrix Turbine。Turbine是一个聚合Hystrix监控数据的工具，他可以将所有相关微服务的Hystrix监控数据聚合到一起，方便使用。引入Turbine后，整个监控系统架构如下：

![企业微信20210902102922](/Users/apple/Desktop/企业微信20210902102922.png)

（1）搭建TurbineServer

创建工程shop-hystrix-turbine引入相关坐标

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
            <version>2.5.4</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
            <version>2.2.9.RELEASE</version>
        </dependency>
```

(2) 配置多个微服务的hystrix监控

在application.yml的配置文件中开启turbine并进行相关配置

```yml
				
turbine:
  # 要监控的微服务列表，多个用，分隔
  appConfig: feign-order-service
  clusterNameExpression: "'default'"
```

turbine会自动的从注册中心中获取需要监控的微服务，并聚合所有微服务中的/hystrix.stream数据

（3）配置启动类

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
//turbine配置
@EnableTurbine
@EnableHystrixDashboard
public class TurbineApplication {
    public static void main(String[] args) {
        SpringApplication.run(TurbineApplication.class,args);
    }
}

```

作为一个独立的监控项目，需要配置启动类，开启HystrixDashboard监控平台，并激活turbine

（4）测试

浏览器访问 http://localhost:8031/hystrix 展示HystrixDashboard。并在url位置输入	，动态根据turbine.stream数据展示多个微服务的监控数据

### 熔断器的隔离策略

微服务使用Hystrix熔断器实现了服务的自动降级，让微服务具有自我保护的能力，提升了系统的稳定性，也较好的解决了雪崩效应。**其使用方式目前支持两种策略**

- 线程池隔离策略：使用一个线程池来存储当前的请求，线程池对请求作处理，设置任务返回处理超时时间，堆积的请求堆积入线程池队列。这种方式需要为每个依赖的服务申请线程池，有一定的资源消耗，好处是可以应对突发流量（流量洪峰来临时，处理不完可将数据存储到线程池里慢慢处理）
- 信号量隔离策略：使用一个原子计数器（或信号量）来记录当前有多少个线程在运行，请求来先判断计数器的数值，若超过设置的最大线程个数则丢弃该类型的新请求，若不超过则执行计数操作请求来计数器+1，请求返回计数器-1。这种方式是严格的控制线程且立即返回模式，无法应对突发流量（流量洪峰来临时，处理的线程超过数量，其他的请求会直接返回，不继续去请求依赖的服务）

##### 线程池和信号量两种策略功能支持对比如下：



![企业微信20210905175706](/Users/apple/Desktop/企业微信20210905175706.png)

- hystrix.command.default.execution.isolation.strategy：配置隔离策略
  - ExecutionIsolationStrategy.SEMAPHORE 信号量隔离
  - ExecutionIsolationStrategy.THREAD 线程池隔离
- hystrix.command.default.execution.isolation.maxConcurrentRequests: 最大信号量上限



# 8.微服务网关概述

在学习完前面的知识后，微服务架构已经初具雏形。但还有一些问题：不同的微服务一般会有不同的网络地址，客户端在访问这些微服务时必须记住几十甚至几百个地址，这对于客户端方来说太复杂也难以维护。如下图![企业微信20210906092424](/Users/apple/Desktop/企业微信20210906092424.png)

如果让客户端直接与各个微服务通讯，可能会有很多问题：

- 客户端会请求多个不同的服务，需要维护不同的请求地址，增加开发难度
- 在某些场景下存在跨域请求的问题
- 加大身份认证的难度，每个微服务需要独立认证

因此，我们需要一个微服务网关，介于客户端与服务器之间的中间层，所有的外部请求都会先经过微服务网关。客户端只需要与网关交互，只知道一个网关地址即可，这样简化了开发还有以下优点：

​	1、易于监控

​	2、易于认证

​	3、减少了客户端与各个微服务之间的交互次数

![企业微信20210906093455](/Users/apple/Desktop/企业微信20210906093455.png)

## 8.1服务网关的概念

### 8.1.1 什么是微服务网关

​	API网关是一个服务器，是系统对外的唯一入口。API网关封装了系统内部架构，为每个客户端提供一个定制的API。API网关方式的核心要点是，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。通常，网关也是提供REST/HTTP的访问API。服务端通过API-GW注册和管理服务

### 8.1.2 作用和应用场景

网关具有的职责，如身份验证、监控、负载均衡、缓存、请求分片与管理、静态响应处理。当然，最主要的职责还是与“外界联系”。

## 8.2 常见的API网关实现方式

- Kong

  基于Nginx+Lua开发，性能高，稳定，有多个可用的插件（限流，鉴权等）可以开箱即用。

  问题：只支持Http协议；二次开发，自由扩展困难；提供管理API，缺乏更易用的管控，配置方式。

- Zuul

  Netflix开源，功能丰富，使用Java开发，易于二次开发；需要运行在web容器中，如tomcat。

  问题：缺乏管控，无法动态配置；依赖组件较多；处理Http请求依赖的是web容器，性能不如Nginx

- SpringCloud Gateway

  SpringCloud提供 的网关服务

## Zuul网关

#### 介绍

Zuul是Netflix开源的微服务网关，它可以和Eureka、Ribbon、Hystrix等组件配合使用，Zuul组件的核心是一系列的过滤器，这些过滤器可以完成以下功能：

- 动态路由：动态请求路由到不同后端集群
- 压力测试：逐渐增加指向集群的流量，以了解性能
- 负载分配：为每一种负载类型分配对应容量，并弃用超出限定值的请求
- 静态响应处理：边缘位置进行响应，避免转发到内部集群
- 身份认证和安全：识别每一个资源的验证请求，并拒绝那些不符的请求。SpringCloud对Zuul进行了整合和增强。



#### 搭建zuul网关服务器

##### （1）创建工程导入依赖

在idea中创建Zuul网关工程shop-zuul-server,并添加相应依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
```

##### （2）配置启动类 ，开启网关服务器功能

```java
// 开启zuul网关功能
@EnableZuulProxy
public class ZuulServerApplication 
```



##### （3）配置文件

```yml
server:
  port: 8080
  tomcat:
    max-threads: 10
spring:
  application:
    name: zuul-server
```



#### 路由

路由：根据请求的URL，将请求分配到对应的微服务中进行处理。

##### 基础路由配置

```yml
# 路由配置
zuul:
  routes:
    # 以商品微服务举例
    product-service: # 路由id，随便写
      path: /product-service/** # 映射路径  #localhost:8080/product-service/dadsawdadw
      url: http://localhost:9001 #映射路径对应的实际微服务url地址
```

##### 面向服务的路由

（1）添加eureka的依赖

```xml
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```



（2）开启eureka的客户端注册发现

```java
@EnableEurekaClient
public class ZuulServerApplication 
```



（3）在zuul网关服务中配置eureka的注册中心相关信息

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:9000/eureka/
  instance:
    prefer-ip-address: true # 使用ip地址注册
```



（4）修改路由中的映射配置

```yml
zuul:
  routes:
    # 以商品微服务举例
    product-service: # 路由id，随便写
      path: /product-service/** # 映射路径  #localhost:8080/product-service/dadsawdadw
#      url: http://localhost:9001 #映射路径对应的实际微服务url地址
      serviceId: product-service # 配置转发的微服务的名称
```

##### 简化路由配置

```yml
zuul:
  routes:
    # 以商品微服务举例
#    product-service: # 路由id，随便写
#      path: /product-service/** # 映射路径  #localhost:8080/product-service/dadsawdadw
##      url: http://localhost:9001 #映射路径对应的实际微服务url地址
#      serviceId: product-service # 配置转发的微服务的名称
      # 如果路由id和对应的微服务的serviceId一致的话
      product-service: /product-service/**
      # zuul中的默认路由配置
      # 如果当前的微服务名称为 service-product，默认的请求映射路径 /service-product/**
```



#### 过滤器

![企业微信20210906144923](/Users/apple/Desktop/企业微信20210906144923.png)

通过之前的学习，我们得知Zuul它包含了两个核心功能：对请求的**路由和过滤**。其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础；而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础。其实，路由功能在真正运行时，它的路由映射和请求转发同样也由几个不同的过滤器完成的。所以，过滤器可以说是Zuul实现API网关功能最为核心的部件，每一个进入Zuul的HTTP请求都会经过一系列的过滤器处理链得到请求响应并返回给客户端。

那么接下来，我们重点学习的就是Zuul的第二个核心功能：**过滤器**。

#### ZuulFilter简介

Zuul中的过滤器跟我们之前使用的javax.servlet.Filter不一样，javax.servlet.Filter只有一种类型，可以通过配置urlPatterns来拦截对应的请求。而Zuul中的过滤器总共有4种类型，且每种类型都有对应的使用场景。

​	1.**PRE**：这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。

​	2.**ROUTING**：这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用Apache HttpClient或Netflix Ribbon请求微服务。

​	3.**POST**：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。

​	4.**ERROR**：在其他阶段发送错误时执行该过滤器。

Zuul提供了自定义过滤器的功能实现起来也十分简单，只需要编写一个类去实现zuul提供的接口

#### 内部源码

