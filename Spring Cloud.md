---

---

# Spring Cloud

## Eureka简介

### 什么是Eureka

Eureka 是 Netflix 开发的，一个基于 REST 服务的，服务注册与发现的组件，以实现中间层服务器的负载平衡和故障转移。

它主要包括两个组件：Eureka Server 和 Eureka Client

- Eureka Client：一个Java客户端，用于简化与 Eureka Server 的交互（通常就是微服务中的客户端和服务端）

- ```xml
  <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
  ```

- Eureka Server：提供服务注册和发现的能力（通常就是微服务中的注册中心）

- ```
  <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  ```

服务在Eureka上注册，然后每隔30秒发送心跳来更新它们的租约。如果客户端不能多次续订租约，那么它将在大约90秒内从服务器注册表中剔除。注册信息和更新被复制到集群中的所有eureka节点。来自任何区域的客户端都可以查找注册表信息（每30秒发生一次）来定位它们的服务（可能在任何区域）并进行远程调用

### 配置集合

```
eureka:
  instance:
    instance-id: eureka8001
    prefer-ip-address: true
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/

```



### 单列配置

service端

- 在yml配置文件里面加入

```yml
server:
  port: 7001
​
eureka:
  instance:
    hostname: localhsot #eureka服务端实例名称
  client:
    register-with-eureka: false #表示不像注册中心注册自己
    fetch-registry: false #false表示自己就是注册中心，我的职责就是维护服务实例,并不区检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

```

- 在启动类添加 @EnableEurekaServer

client端

- yml文件里面添加

```
eureka:
  client:
    register-with-eureka: true #表示向注册中心注册自己 默认为true
    fetch-registry: true #是否从EurekaServer抓取已有的注册信息，默认为true,单节点无所谓,集群必须设置为true才能配合ribbon使用负载均衡
    service-url:
      defaultZone: http://localhost:7001/eureka/ # 入驻地址
```

- 在启动类添加 @EnableEurekaClient

### Eureka集群

服务注册：将服务信息注册到注册中心

服务发现：从注册中心获取服务信息

实质：存key服务名，取value调用地址

步骤：

1. 先启动eureka注册中心

2. 启动服务提供者payment支付服务

3. 支付服务启动后，会把自身信息注册到eureka

4. 消费者order服务在需要调用接口时，使用服务别名去注册中心获取实际的远程调用地址

5. 消费者获得调用地址后，底层实际是调用httpclient技术实现远程调用

6. 消费者获得服务地址后会缓存在本地jvm中，默认每30秒更新异常服务调用地址
   

集群搭建步骤

- 修改C:\Windows\System32\drivers\etc下的hosts

```
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
127.0.0.1 eureka7003.com
```

- service1 applicaton.yml

```
server:
  port: 7001
​
eureka:
  instance:
    hostname: eureka7001.com #eureka服务端实例名称
  client:
    register-with-eureka: false #表示不向注册中心注册自己
    fetch-registry: false #false表示自己就是注册中心，我的职责就是维护服务实例,并不区检索服务
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/
```

-  service2 applicaton.yml

```
server:
  port: 7002
​
 eureka:
  instance:
    hostname: eureka7002.com #eureka服务端实例名称
  client:
    register-with-eureka: false #表示不向注册中心注册自己
    fetch-registry: false #false表示自己就是注册中心，我的职责就是维护服务实例,并不区检索服务
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

- client端修改yml文件,只修改service-url

```
    service-url:
      #defaultZone: http://localhost:7001/eureka/ # 入驻地址
      defaultZone: http://eureka7001.com:7001/eureka/, http://eureka7001.com:7001/eureka/ 
```

### 负载均衡

RestTemplate上添加注解

`@LoadBalanced` 注解表示 开启负载均衡

```
@LoadBalanced // 使RestTemplate有了负载均衡的能力
@Bean
public RestTemplate restTemplate(){
    return new RestTemplate();
}
```

因为加了eureka就会自动添加 ribbon依赖，所以不用加ribbon consumer端的restful调用别写urm -不能写 localhost:8001

```
public static  final String PAYMENT_URL = "http://PROVIDER-PAYMENT";

@GetMapping("/payment/{id}")
public CommonResult<Payment> get(@PathVariable Long id){

           return restTemplate.getForObject(PAYMENT_URL+"/payment/getById/"+id,CommonResult.class);

}
```

### actuator微服务信息完善

注意：需要有这些依赖

```
<!-- web -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 图形化监控 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

配置yaml

```
eureka:
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
   #新增配置
  instance:
    instance-id: payment8001
    prefer-ip-address: true   # 访问路径可以显示IP地址
```

配置instance-id效果图

![image-20230419182853401](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20230419182853401.png)

![image-20230419183223654](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20230419183223654.png)

配置 prefer-ip-addres 会在页面超链接显示ip地址

### 服务发现Discovery

需要在配置类上配置

@EnableDiscoveryClient

可以通过discoveryClient获得eureka服务中心里面注册的全部服务名 端口号 url等信息

```
 @Autowired
private DiscoveryClient discoveryClient;

@GetMapping(value = "/payment/discovery")
public Object discovery(){
    List<String> services = discoveryClient.getServices();
    for (String element : services) {
        log.info("***** element:"+element);
    }
    List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
    for (ServiceInstance instance : instances) {
        log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
    }
    return this.discoveryClient;
}
```



### Eureka自我保护模式

如果 Eureka 服务器检测到超过预期数量的注册客户端以一种不优雅的方式终止了连接，并且同时正在等待被驱逐，那么它们将进入自我保护模式。这样做是为了确保灾难性网络事件不会擦除eureka注册表数据，并将其向下传播到所有客户端。

任何客户端，如果连续3次心跳更新失败，那么它将被视为非正常终止，病句将被剔除。当超过当前注册实例15%的客户端都处于这种状态，那么自我保护将被开启。

当自我保护开启以后，eureka服务器将停止剔除所有实例，直到：

- 它看到的心跳续借的数量回到了预期的阈值之上，或者
- 自我保护被禁用

默认情况下，自我保护是启用的，并且，默认的阈值是要大于当前注册数量的15%

如何关闭自我保护机制
一、 服务端：

 配置eureka.server.enable-self-preservation = false关闭自我保护机制。

 配置eureka.server.eviction-interval-timer-in-ms = 2000让服务端每隔2秒扫描一次，是服务能尽快的剔除。

```
eureka:
  server:
    #服务端是否开启自我保护机制 （默认true）
    enable-self-preservation: false
    #扫描失效服务的间隔时间（单位毫秒，默认是60*1000）即60秒
    eviction-interval-timer-in-ms: 2000

```

客户端：

 客户端配置eureka.instance.lease-renewal-interval-in-seconds = 1 客户端向服务端发送心跳的间隔，设置为1秒一次。

 lease-expiration-duration-in-seconds: 2 服务端收到客户端服务后，等待下次心跳的超时时间，设置为2秒。如果超过2秒，移除该客户端。

```
 # 客户端向注册中心发送心跳的时间间隔，（默认30秒）
    lease-renewal-interval-in-seconds: 1
    #Eureka注册中心（服务端）在收到客户端心跳之后，等待下一次心跳的超时时间，如果在这个时间内没有收到下次心跳，则移除该客户端。（默认90秒）
    lease-expiration-duration-in-seconds: 2
```

## zookeeper

### 安装

先下载zookeeper安装包

```
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz
```

解压

```
tar -zxvf zookeeper-3.4.12.tar.gz
```

创建日志目录

```
cd /app/zookeeper-3.4.12
mkdir logs
mkdir data
```

修改配置文件

进入conf文件内 执行如下命令 复制并改名。

```
[root@VM-16-2-centos conf]# cp zoo_sample.cfg zoo.cfg
```

编辑zoo.fcg

```
#数据文件夹
dataDir=/usr/local/zookeeper-3.6.3/data
#日志文件夹
dataLogDir=/usr/local/zookeeper-3.6.3/logs
```

 然后我们修改系统配置文件：配置环境变量

```
[root@VM-16-2-centos conf]# vim /etc/profile

#zookeeper-3.6.2配置
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.6.3/
export PATH=$ZOOKEEPER_HOME/bin:$PATH
export PATH
```

运行文件：应用刚才的修改

```
[root@VM-16-2-centos conf]# source /etc/profile
```

启动zookeeper

```
[root@VM-16-2-centos bin]# ./zkServer.sh start
```

### 简单使用

1. 先引入jar包

   由于版本问题 需要排除自带的zookeeperjar包 否则跟服务器zookeeper版本不一致会报错

   ```
   <!--        spring cloud 整合 zookeeper client依赖-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
               <exclusions>
                   <exclusion>
                       <groupId>org.apache.zookeeper</groupId>
                       <artifactId>zookeeper</artifactId>
                   </exclusion>
               </exclusions>
           </dependency>
           <dependency>
               <groupId>org.apache.zookeeper</groupId>
               <artifactId>zookeeper</artifactId>
               <version>3.4.12</version>
           </dependency>
   ```

2. 配置链接地址

   ```
   spring:
     application:
       name: zookeeper-provider-payment
     cloud:
       zookeeper:
         connect-string: 1.15.111.144:2181
   ```

3. 启用注解

   ```
   @EnableDiscoveryClient //该注解用于向使用consul或者zookeeper作为注册中心时注册服
   ```

调用其他微服务方法

```
  	@Bean
    @LoadBalanced // 必须要加
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
	}
```

```
	 public static final String URL="http://zookeeper-provider-payment/"; // 注册服务名

    @Resource
    public RestTemplate restTemplate;

    @GetMapping("zk")
    public String paymentInfo(){
        return  restTemplate.getForObject(URL+"/payment/zk",String.class);
    }
```

## Consul

### 安装

```
 yum install -y yum-utils
 yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
 yum -y install consul
```

### 启动

```
consul agent -dev -client 0.0.0.0
```

配置

```
server:
  port: 8006
spring:
  application:
    name: consul-provider-payment
  cloud:
    consul:
      host: 1.15.85.115
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```

## Ribbon

客户端 配置 不在主启动类包及其子包下 不能被组启动类扫描到

```
@Configuration
public class MyselfRule {

    @Bean
    public IRule myRule(){
        return  new RandomRule(); //随机
    }
}
启动类上添加
@RibbonClient(name = "provider-payment",configuration = MyselfRule.class)
```

### 策略

| 序号 | 实现类                    | 负载均衡策略                                                 |
| ---- | ------------------------- | ------------------------------------------------------------ |
| 1    | RoundRobinRule            | 按照线性轮询策略，即按照一定的顺序依次选取服务实例           |
| 2    | RandomRule                | 随机选取一个服务实例                                         |
| 3    | RetryRule                 | 按照 RoundRobinRule（轮询）的策略来获取服务，如果获取的服务实例为 null 或已经失效，则在指定的时间之内不断地进行重试（重试时获取服务的策略还是 RoundRobinRule 中定义的策略），如果超过指定时间依然没获取到服务实例则返回 null 。 |
| 4    | WeightedResponseTimeRule  | WeightedResponseTimeRule 是 RoundRobinRule 的一个子类，它对 RoundRobinRule 的功能进行了扩展。  根据平均响应时间，来计算所有服务实例的权重，响应时间越短的服务实例权重越高，被选中的概率越大。刚启动时，如果统计信息不足，则使用线性轮询策略，等信息足够时，再切换到 WeightedResponseTimeRule。 |
| 5    | BestAvailableRule         | 继承自 ClientConfigEnabledRoundRobinRule。先过滤点故障或失效的服务实例，然后再选择并发量最小的服务实例。 |
| 6    | AvailabilityFilteringRule | 先过滤掉故障或失效的服务实例，然后再选择并发量较小的服务实例。 |
| 7    | ZoneAvoidanceRule         | 默认的负载均衡策略，综合判断服务所在区域（zone）的性能和服务（server）的可用性，来选择服务实例。在没有区域的环境下，该策略与轮询（RandomRule）策略类似。 |

## OpenFeign

### 简单使用

先导入jar包

```
      <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

写接口

```
@FeignClient("provider-payment")
public interface PaymentFeign {

    @GetMapping("/payment/getById/{id}")
    public CommonResult<Payment> getById(@PathVariable("id") Long id);
}

```

调用

```
    @Resource
    PaymentFeign paymentFeign;

    @GetMapping("getById/{id}")
    public CommonResult create(@PathVariable Long id){
        return paymentFeign.getById(id);
    }
```

启动类上面加注解

```
@EnableFeignClients
```

### 超时设置

```
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
#指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
#指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
#开启这个会使设置无效
feign:
#  hystrix:
#    enabled: true

```

### 日志查看

级别

- NONE：默认的，不显示任何日志；
- BASIC：仅记录请求方法、URL、响应状态码及执行时间；
- HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；
- FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据。

配置

```
@Configuration
public class FeignConfig
{
    @Bean
    Logger.Level feignLoggerLevel()
    {
        return Logger.Level.FULL;
    }
}
```

```
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.atguigu.springcloud.service.PaymentFeignService: debug
 
```

## Hystrix

### 服务降级 简单使用

jar包

```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

添加注解

```
 @HystrixCommand(
            fallbackMethod = "timeOut", //fallbackMethod 会找同类下的方法
            commandProperties = {
                    @HystrixProperty(
                            name = "execution.isolation.thread.timeoutInMilliseconds", //超时和抱错都会调用方法
                            value = "2000"
                    )
            }
    )
    @GetMapping("TimeOut/{id}")
    public String paymentInfoTimeOut(@PathVariable Integer id){
        return paymentService.paymentInfoTimeOut(id);
    }
    public String timeOut(@PathVariable Integer id){
        return "稍后重试";
    }

```

启动类添加注解

```
@EnableCircuitBreaker
```

解决每个方法上面的要加注解和写方法

feign 写一个feign实现类  然后在Service 添加 fallbacl

```
@FeignClient(value = "hystrix-payment",fallback = PaymentFeignFallBacK.class) 
```

```
@Component
public class PaymentFeignFallBacK implements PaymentFeign {
    @Override
    public String paymentInfoOK(Integer id) {
        return "FeignFallBacK 稍后重试";
    }

    @Override
    public String paymentInfoTimeOut(Integer id) {
        return "FeignFallBacK 稍后重稍后重试";
    }
}

```

yml文件添加

```
feign:
  hystrix:
    enabled: true
```

或者使用 @DefaultProperties(defaultFallback = "test") 没有降级处理 就默认test

controller 加上

```
@DefaultProperties(defaultFallback = "test")

    public String test(){
        return "default 稍后重试";
    }
```

### 切记

一定要在启动类上加

```
EnableHystrix //或者@EnableCircuitBreaker
```

### 服务熔断

```
@HystrixCommand(fallbackMethod = "str_fal1backMethod",
        groupKey = "strGroupCommand",
        commandKey = "strCommand",
        threadPoolKey = "strThreadPoo1",
        commandProperties = {
                //设置隔离策略，THREAD表示线程地SEMAPHORE:信号池隔离
                @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
                //当隔离策略选择信号池隔离的时候，用来设置信号池的大小(最大并发数)
                @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
                //配置命令执行的超时时间
                @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
                //是否启用超时时间
                @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
                //执行超时的时候是否中断
                @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
                //执行被取消的时候是否中断
                @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
                //允许回调方法执行的最大并发数
                @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
                //服务降级是否启用，是否执行回调函数
                @HystrixProperty(name = "fallback.enabled", value = "true"),
                ////是否开启熔断
                @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
                //该属性用来没置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为20的时候
                //，如果滚动时间窗（默认10秒）内仅收到了19个请求，即使这19个请求都失败了，断路器也不会打开。
                @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
                //该属性用来没置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过
                //circuitBreaker.requestVolumeThreshold的情况下，如果错误
                //请求数的百分比超过50,就把断路器设置为“打开”状态，否则就设置为“关闭”状态。
                @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
                //该属性用来设置当断路器打开之后的休眠时间窗。休眠时间窗结束之后，
                //会将断路器置为“半开”状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为“打开”状态，
                //如果成功就没置为"关闭”状态。
                @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
                //断路器强制打开
                @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
                //断路器强制关闭
                @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
                //滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
                @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
                //该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据
                //设置的时间窗长度拆分成多个“桶”来累计各度量值，每个”桶"记录了一段时间内的采集指标。
                //比如10秒内拆分成10个"桶"收集这样，所以 timeinMilliseconds必须能被numBuckets整除。否则会抛异常@HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
                @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
                //该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果没置为false，那么所有的概要统计都将返回-1。
                @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
                //该属性用来没置百分位统计的滚动窗口的持续时间，单位为毫秒。
                @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
                // 该属性用来没置百分位统计滚动窗口中使用“桶”的数量。
                @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
                //该属性用来没置在执行过程中每个“桶”中保留的最大执行次数。如果在添动时间窗内发生超过该设定值的执行次数，
                //就从最初的位置开始重写。例如，将该值设置为100，滚动窗口为10秒，若在10秒内一个“桶”中发生了500次执行，
                //那么该“桶”中只保留最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。
                @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
                //该属性用来设置采集影响断路器状态的健康快照（请求的成功、错误百分比）的间隔等待时间。
                @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
                //是否开启请求缓存
                @HystrixProperty(name = "requestCache.enabled", value = "true"),
                //HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中
                @HystrixProperty(name = "requestLog.enabled", value = "true"),},
        threadPoolProperties = {
                //该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
                @HystrixProperty(name = "coreSize", value = "10"),
                //该参数用来没置线程池的最大队列大小。当设置为-1时，线程池将使用SynchronousQueue实现的队列，
                //否则将使用LinkedBLockingQueue实现的队列。
                @HystrixProperty(name = "maxQueueSize", value = "-1"),
                //该参数用来为队列设置拒绝阈值。通过该参数，即使队列没有达到最大值也能拒绝请求。
                //该参数主要是对LinkedBLockingQueue 队列的补充,因为LinkedBLockingQueue
                //队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
                @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5")})

```

## Gateway

### 简单使用

jar包

```
   <!--gateway-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
```

yml

```
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: feign-hystrix-order #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://feign-hystrix-order  #匹配后提供服务的路由地址 可使用固定的ip地址也可以使用微服务名字
          predicates:
            - Path=/order/**         # 断言，路径相匹配的进行路由

```

启动类记得添加

```
@EnableEurekaClient
```

## Stream

### 简单使用

jar包

```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
```

yml  消费者把output 改成input

```
spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: 1.15.85.110
                port: 5672
                username: acszdxt
                password: acszdxt
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称 
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
```

生产者发送消息

```
@EnableBinding(Source.class) // 可以理解为是一个消息的发送管道的定义
public class MessageProviderImpl implements IMessage {
    @Resource
    private MessageChannel output; // 消息的发送管道

    @Override
    public String send() {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build()); // 创建并发送消息
        System.out.println("***serial: " + serial);

        return serial;
    }
}

```

消费者

```
@Component
@EnableBinding(Sink.class)
@Slf4j
public class receiverMsg {

    @StreamListener(Sink.INPUT)
    public void input(Message<String> msg){
        log.info("收到的消息{}",msg.getPayload());
    }
}
```

### 分组

可以避免重复消费的问题

```
   bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称 
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          group: aaa #分组名
```

## Nacos

jar

```
		<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
```

### 简单使用

```
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*' #暴露所有端点
```

加上注解即可

```
@EnableDiscoveryClient
```

### 配置中心

jar

```
  		<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
```

配置名称

```
${prefix}-${spring.profiles.active}.${file-extension}

```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

![image-20230523165140924](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20230523165140924.png)

可以设置不同的group和命名空间来对同一个服务进行不同的配置

```
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
      config:
        server-addr:  localhost:8848
        file-extension: yaml
        group: test  #不写就是默认的
        namespace:  #不写默认public 需要写命名空间id
```

#### 动态刷新

使用@Value需要添加注解

```
@RefreshScope
```

使用@ConfigurationProperties，添加配置获取类 不需要添加注解

```
@Data
@Component
@ConfigurationProperties(prefix = "xx.tool")
public class ToolProperties {

    private String url;

}

```

## Sentinel

jar包

```
 		<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

yml

```
spring:
  application:
    name: cloudaibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        #配置Sentinel dashboard地址
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719
#      filter:
#        url-patterns: /**
```

### 限流

![image-20230528175244239](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20230528175244239.png)

#### 流控

![image-20230528175716746](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20230528175716746.png)

- 资源名 可以写url或者SentinelResource value里面的值
- 针对来源：Sentinel 可以针对调用这进行限流，填写微服务名，默认 default（不区分来源）。
- 阈值类型/单击阈值：
  （1）QPS（每秒钟的请求数量）：当调用该 API 的 QPS 达到阈值的时候，进行限流。
  （2）线程数：当调用该 API 的线程数达到阈值的时候，进行限流。
- 流控模式：
  （1）直接：API 达到限流条件时，直接限流。
  （2）关联：当关联的资源达到阈值时，就限流自己。
  （3）链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流）【 API 级别的针对来源】
- 流控效果：
  （1）快速失败：直接失败，抛异常。
  （2）Warn Up：根据 coldFactor （冷加载因子，默认3）的值，从阈值/coldFactor，经过预热时长，才达到设置的 QPS 阈值。比如是阈值是10 预热时间五秒钟 那么刚开始就是每秒3dqps 到五秒之后提升到10
  （3）排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为 QPS，否则无效。

#### 降级

![](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20230528180819685.png)

- 资源名同上
- 降级策略
  - rt响应时间  一秒钟最少五次请求的响应时间大于设置的Rt值就会处罚降级 降级的时间是由时间窗口来定

#### 热点

![image-20230528181259307](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20230528181259307.png)

- 资源名和上面的解释一样
- 参数索引是 url地址后的参数
  - 比如设置0 就是第一参数 携带参数后就不能超过每秒钟的qps
- 参数类型 可以自己选择推荐string
- 参数值 设置限流的搜索的内容 比如所猫 限流阈值1 那么搜索的内容第一个参数为猫且每秒的访问量不能超过1

## @SentinelResource

```
 	/**
     * value 资源名
     * fallback java运行异常处理
     * blockHandler sentinel 控制台配置处理
     * exceptionsToIgnore 排除异常
     */
 @SentinelResource(value = "", fallback = "",blockHandler = exceptionsToIgnore = )
```

该注解不支持private

可以通过设置注解内的参数来设置资源名和返回友好提示的内容

```
    @GetMapping("/byResource")
    @SentinelResource(value = "byResource", blockHandler = "handlerException")
    public String byResource() {
        return "200";
    }

    public String handlerException(BlockException blockException) {
        return "404";
    }
```

也可以

自定义一个类

```
public class CustomerBlockerHandler {

    public static  String handlerException (BlockException exception){
        return "444";
    }

}

```

然后在SentinelResource注解里面blockHandlerClass =  自定义类名 blockHandler =   自定义类名里面的方法名

```
 @GetMapping("/customerBlockerHandler")
    @SentinelResource(value = "customerBlockerHandler",blockHandlerClass = CustomerBlockerHandler.class,blockHandler = "handlerException")
    public String customerBlockerHandler() {
        return "自定义";
    }
```

### 服务熔断  

#### ribbon fallback

```
@GetMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback", fallback = "handlerFallback")
    public CommonResult<Payment> fallback(@PathVariable Long id) {
        CommonResult<Payment> result = restTemplate.getForObject(url + "/paymentSQL/" + id, CommonResult.class, id);

        if (id == 4) {
            throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
        } else if (result.getData() == null) {
            throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }

```

必须有

```
public CommonResult handlerFallback(@PathVariable Long id, Throwable e) {
        Payment payment = new Payment(id, "null");
        return new CommonResult<>(444, "兜底异常handlerFallback,exception内容  " + e.getMessage(), payment);
    }
```

#### feign

先添加jar包 然后开启sentinel对feign的支持

```
#激活sentinel 对fegin支持
feign:
  sentinel:
    enabled: true
```

配置feign的fallback

![image-20230604205445336](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20230604205445336.png)

Payservice

```
@FeignClient(value = "nacos-payment-provider",fallback = PayServiceFallBack.class)
public interface PayService {
    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);
}

```

fallback类

```
@Component
public class PayServiceFallBack implements PayService {
    @Override
    public CommonResult<Payment> paymentSQL(Long id) {
        return new CommonResult<>(444,"feign fallback");
    }
}

```

主启动类上加上 @EnableFeignClients
