# 微服务架构理论入门

# SpringCloud 初步构建

## Cloud组件停更说明

- 停更引发的“升级惨案”
  - 停更不停用
  - 被动修复bugs
  - 不再接受合并请求
  - 不再发布新版本
- Cloud升级

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220811/image.5g6chrwarzs0.webp)

# Eureka 服务注册与发现

## 什么是服务注册与发现

**Eureka包含两个组件:Eureka Server和Eureka Client**

 ![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220813/image.5e5dv8ys3mc0.webp)

Eureka Server提供服务注册服务

各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

EurekaClient通过注册中心进行访问

它是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒)

## EurekaServer服务端安装

1.创建名为cloud-eureka-server7001的Maven工程

2.修改pom.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>cloud-eureka-server7001</artifactId>
    <dependencies>
        <!--eureka-server-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.frx01.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--boot web actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般通用配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
    </dependencies>
</project>

```

3.添加application.yml

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: locathost #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

4.主启动

```java
package com.frx01.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/**
 * @author frx
 * @version 1.0
 * @date 2022/8/13  21:37
 */
@SpringBootApplication
@EnableEurekaServer
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class,args);
    }
}
```

5.测试运行`EurekaMain7001`，浏览器输入`http://localhost:7001/`回车，会查看到Spring Eureka服务主页。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220813/image.6u9bkkovwc80.webp)

## 支付微服务8001入驻进EurekaServer

EurekaClient端cloud-provider-payment8001将注册进EurekaServer成为服务提供者provider，类似学校对外提供授课服务。

1.修改cloud-provider-payment8001

2.改POM

添加spring-cloud-starter-netflix-eureka-client依赖

3.写YML

```yaml
eureka:
  client:
    #表示是否将自己注册进Eurekaserver默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

4.主启动

```java
@SpringBootApplication
@EnableEurekaClient//<-----添加该注解
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

5.测试

- 启动cloud-provider-payment8001和cloud-eureka-server7001工程。
- 浏览器输入 - http://localhost:7001/ 主页内的Instances currently registered with Eureka会显示cloud-provider-payment8001的配置文件application.yml设置的应用名`cloud-payment-service`

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.4dmsitp0l9g0.webp)

```xml
spring:
  application:
    name: cloud-payment-service
```

## 订单微服务80入驻进EurekaServer

EurekaClient端cloud-consumer-order80将注册进EurekaServer成为服务消费者consumer，类似来上课消费的同学

1. cloud-consumer-order80
2. POM

3. YML

```yml
server:
  port: 80

spring:
  application:
    name: cloud-order-service

eureka:
  client:
    #表示是否将自己注册进Eurekaserver默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

4. 主启动

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient//<--- 添加该标签
public class OrderMain80
{
    public static void main( String[] args ){
        SpringApplication.run(OrderMain80.class, args);
    }
}
```

测试

- 启动cloud-provider-payment8001、cloud-eureka-server7001和cloud-consumer-order80这三工程。
- 浏览器输入 http://localhost:7001 , 在主页的Instances currently registered with Eureka将会看到cloud-provider-payment8001、cloud-consumer-order80两个工程名。

> 因为我的80端口被进程4占用了，我把80端口修改为81

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.4njizhxa1fg0.webp)

> 注意，application.yml配置中层次缩进和空格，两者不能少，否则，会抛出异常`Failed to bind properties under 'eureka.client.service-url' to java.util.Map <java.lang.String, java.lang.String>`。

## Eureka集群原理说明

1.Eureka集群原理说明

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.68sh6sha7jk0.webp)

服务注册：将服务信息注册进注册中心

服务发现：从注册中心上获取服务信息

实质：存key服务命取value闭用地址

##  Eureka集群环境构建

创建cloud-eureka-server7002工程，过程参考[EurekaServer服务端安装](https://frxcat.fun/Spring/SpringCloud/Eureka_/#eurekaserver服务端安装)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.6yctjdoyzgc0.webp)

- 找到C:\Windows\System32\drivers\etc路径下的hosts文件，修改映射配置添加进hosts文件

```sh
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
```

- 修改cloud-eureka-server7001配置文件

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
    #集群指向其它eureka
      defaultZone: http://eureka7002.com:7002/eureka/
    #单机就是7001自己
      #defaultZone: http://eureka7001.com:7001/eureka/
```

- 修改cloud-eureka-server7002配置文件

```yaml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
    #集群指向其它eureka
      defaultZone: http://eureka7001.com:7001/eureka/
    #单机就是7002自己
      #defaultZone: http://eureka7002.com:7002/eureka/
```

- 访问:[http://eureka7001.com:7001/(opens new window)](http://eureka7001.com:7001/)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.78r2yimub340.webp)

- 访问:[http://eureka7002.com:7002/(opens new window)](http://eureka7002.com:7002/)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.3kkbenq2jtq0.webp)

## 订单支付两微服务注册进Eureka集群

- 将支付服务8001微服务，订单服务80微服务发布到上面2台Eureka集群配置中

将它们的配置文件的eureka.client.service-url.defaultZone进行修改

```yaml
eureka:
  client:
    #表示是否将自己注册进Eurekaserver默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka, http://eureka7002.com:7002/eureka
```

- 测试
  1. 先要启动EurekaServer，7001/7002服务
  2. 再要启动服务提供者provider，8001
  3. 再要启动消费者，80
  4. 浏览器输入 - [http://localhost:81/consumer/payment/get/1(opens new window)](http://localhost:81/consumer/payment/get/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.3j7veuxdbc60.webp)

## 支付微服务集群配置

.新建cloud-provider-payment8002

2.改POM

3.写YML - 端口8002

4.主启动

5.业务类

6.修改8001/8002的Controller，添加serverPort

```java
@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @Value("${server.port}")
    private String serverPort;//添加serverPort

    @PostMapping("/payment/create")
    public CommonResult create(@RequestBody Payment payment){
        int result = paymentService.create(payment);
        log.info("插入结果:"+result);
        if(result>0){
            return new CommonResult(200,"插入数据库成功,serverPort:"+serverPort,result);
        }
        return new CommonResult(444,"插入数据库失败",null);
    }
}
```

### **负载均衡**

cloud-consumer-order80订单服务访问地址不能写死

```java
@Slf4j
@RestController
public class OrderController {

    //public static final String PAYMENT_URL = "http://localhost:8001";
    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";//eureka注册的服务名称
    
    ...
}
```

使用@LoadBalanced注解赋予RestTemplate负载均衡的能力

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced//使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

测试

结果：负载均衡效果达到，**8001/8002端口交替出现**

Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号，且该服务还有负载功能。

## actuator微服务信息完善

主机名称：服务名称修改（也就是将IP地址，换成可读性高的名字）

修改cloud-provider-payment8001，cloud-provider-payment8002

修改部分 - YML - eureka.instance.instance-id

```yaml
eureka:
  ...
  instance:
    instance-id: payment8001 #添加此处
```

```yaml
eureka:
  ...
  instance:
    instance-id: payment8002 #添加此处
```

修改之后

eureka主页将显示payment8001，payment8002代替原来显示的IP地址。

访问信息有IP信息提示，（就是将鼠标指针移至payment8001，payment8002名下，会有IP地址提示）

修改部分 - YML - eureka.instance.prefer-ip-address

```yaml
eureka:
  ...
  instance:
    instance-id: payment8001 
    prefer-ip-address: true #添加此处
```

```yaml
eureka:
  ...
  instance:
    instance-id: payment8002
    prefer-ip-address: true #添加此处
```

## 服务发现Discovery

对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息

- 修改cloud-provider-payment8001的Controller

```java
@RestController
@Slf4j
public class PaymentController {

    ...
        
    @Resource
    private DiscoveryClient discoveryClient;

    ...
        
    @GetMapping(value = "/payment/discovery")
    public Object discovery(){
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            log.info("element:"+element);
        }

        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
            log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
        }
        return this.discoveryClient;
    }
}
```

- 8001主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient//<-----添加该注解
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

- 自测

先要启动EurekaServer

再启动8001主启动类，需要稍等一会儿

浏览器输入http://localhost:8001/payment/discovery

浏览器输出：

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.s1rj2eu7u1c.webp)

后台输出：

```java
element:cloud-order-service
CLOUD-PAYMENT-SERVICE	localhost	8002	http://localhost:8002
```

## Eureka自我保护理论知识

**概述**

保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。

如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式:

EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY’RE NOT. RENEWALS ARE LESSER THANTHRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUSTTO BE SAFE

**导致原因**

一句话：某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存。

属于CAP里面的AP分支。

**为什么会产生Eureka自我保护机制?**

为了EurekaClient可以正常运行，防止与EurekaServer网络不通情况下，EurekaServer不会立刻将EurekaClient服务剔除

**什么是自我保护模式?**

默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例(默认90秒)。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时(可能发生了网络分区故障)，那么这个节点就会进入自我保护模式。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.23g8spa37tz4.webp)

自我保护机制∶默认情况下EurekaClient定时向EurekaServer端发送心跳包

如果Eureka在server端在一定时间内(默认90秒)没有收到EurekaClient发送心跳包，便会直接从服务注册列表中剔除该服务，但是在短时间( 90秒中)内丢失了大量的服务实例心跳，这时候Eurekaserver会开启自我保护机制，不会剔除该服务（该现象可能出现在如果网络不通但是EurekaClient为出现宕机，此时如果换做别的注册中心如果一定时间内没有收到心跳会将剔除该服务，这样就出现了严重失误，因为客户端还能正常发送心跳，只是网络延迟问题，而保护机制是为了解决此问题而产生的)。

**在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。**

它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话讲解：**好死不如赖活着。**

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

## 怎么禁止自我保护

- 在eurekaServer端7001处设置关闭自我保护机制

出厂默认，自我保护机制是开启的

使用`eureka.server.enable-self-preservation = false`可以禁用自我保护模式

```yaml
eureka:
  ...
  server:
    #关闭自我保护机制，保证不可用服务被及时踢除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

关闭效果：

spring-eureka主页会显示出一句：

**THE SELF PRESERVATION MODE IS TURNED OFF. THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS.**

- 生产者客户端eureakeClient端8001

默认：

```
eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90
```

```yaml
eureka:
  ...
  instance:
    instance-id: payment8001
    prefer-ip-address: true
    #心跳检测与续约时间
    #开发时没置小些，保证服务关闭后注册中心能即使剔除服务
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```

- 测试
  - 7001和8001都配置完成
  - 先启动7001再启动8001

结果：先关闭8001，马上被删除了

## Eureka停更说明

我们用ZooKeeper代替Eureka功能。

# ZooKeeper 服务注册与发现

## 支付服务注册进ZooKeeper

- 注册中心Zookeeper

zookeeper是一个分布式协调工具，可以实现注册中心功能

关闭Linux服务器防火墙后，启动zookeeper服务器

用到的Linux命令行：

- `systemctl stop firewalld`关闭防火墙
- `systemctl status firewalld`查看防火墙状态
- `ipconfig`查看IP地址
- `ping`查验结果

zookeeper服务器取代Eureka服务器，zk作为服务注册中心

- [CentOS安装ZooKeeper步骤](https://frxcat.fun/middleware/Dubbo/Dubbo_Geting_start/#zookeeper安装)

```sh
[root@master ~]# cd /opt/zookeeper/apache-zookeeper-3.5.6-bin/bin/
[root@master bin]# ls
README.txt    zkCli.cmd  zkEnv.cmd  zkServer.cmd            zkServer.sh          zkTxnLogToolkit.sh
zkCleanup.sh  zkCli.sh   zkEnv.sh   zkServer-initialize.sh  zkTxnLogToolkit.cmd
[root@master bin]# ./zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/apache-zookeeper-3.5.6-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

建名为cloud-provider-payment8004的Maven工程。

2.POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-provider-payment8004</artifactId>
    <dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.frx01.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper3.5.3 防止与3.5.6起冲突-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.5.6版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.5.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

3. 编写YML

```yaml
#8004表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 8004

#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 192.168.91.200:2181 # zookeeper服务IP加端口号
```

4. 主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8004 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8004.class,args);
    }
}
```

5. Controller

```java
@RestController
@Slf4j
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/zk")
    public String paymentzk(){
        return "springcloud with zookeeper: "+serverPort+"\t"+ UUID.randomUUID().toString();
    }
}
```

6. 启动8004注册进zookeeper（要先启动zookeeper的server）

- 验证测试：浏览器 - http://localhost:8004/payment/zk
- 验证测试2 ：接着用zookeeper客户端操作

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.79ugu9at2540.webp)

```sh
[zk: localhost:2181(CONNECTED) 1] ls /
[dubbo, services, zookeeper]
[zk: localhost:2181(CONNECTED) 2] ls /services/cloud-provider-payment
[54ba4644-818d-404f-ac73-e76f525b10a4]
[zk: localhost:2181(CONNECTED) 3] get /services/cloud-provider-payment/54ba4644-818d-404f-ac73-e76f525b10a4
{"name":"cloud-provider-payment","id":"54ba4644-818d-404f-ac73-e76f525b10a4","address":"localhost","port":8004,"sslPort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"application-1","name":"cloud-provider-payment","metadata":{}},"registrationTimeUTC":1660484409499,"serviceType":"DYNAMIC","uriSpec":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true},{"value":":","variable":false},{"value":"port","variable":true}]}}
```

json格式化`get /services/cloud-provider-payment/54ba4644-818d-404f-ac73-e76f525b10a4`的结果：

```json
{
  "name": "cloud-provider-payment",
  "id": "54ba4644-818d-404f-ac73-e76f525b10a4",
  "address": "localhost",
  "port": 8004,
  "sslPort": null,
  "payload": {
    "@class": "org.springframework.cloud.zookeeper.discovery.ZookeeperInstance",
    "id": "application-1",
    "name": "cloud-provider-payment",
    "metadata": {}
  },
  "registrationTimeUTC": 1660484409499,
  "serviceType": "DYNAMIC",
  "uriSpec": {
    "parts": [
      {
        "value": "scheme",
        "variable": true
      },
      {
        "value": "://",
        "variable": false
      },
      {
        "value": "address",
        "variable": true
      },
      {
        "value": ":",
        "variable": false
      },
      {
        "value": "port",
        "variable": true
      }
    ]
  }
}
```

## 临时还是持久节点

ZooKeeper的服务节点是**临时节点**，没有Eureka那含情脉脉

## 订单服务注册进zookeeper

1.新建cloud-consumerzk-order80

2.POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-consumerzk-order80</artifactId>
    <dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.5.6版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.5.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

3. YML

```yaml
server:
  port: 81

#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-consumer-order
  cloud:
    zookeeper:
      connect-string: 192.168.91.200:2181
```

> 80端口被占用了，我换为81

4. 主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderZKMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderZKMain80.class,args);   
    }
}
```

5. 业务类

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

```java
@RestController
@Slf4j
public class OrderZKController {

    public static final String INVOKE_URL = "http://cloud-provider-payment";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/payment/zk")
    public String paymentInfo(){
        String result = restTemplate.getForObject(INVOKE_URL+"/payment/zk",String.class);
        return result;
    }
}
```

6.验证测试

运行ZooKeeper服务端，cloud-consumerzk-order80，cloud-provider-payment8004。

打开ZooKeeper客户端：

```sh
[zk: localhost:2181(CONNECTED) 4] ls /services
[cloud-consumer-order]
[zk: localhost:2181(CONNECTED) 5] ls /services
[cloud-consumer-order, cloud-provider-payment]
```

7. 访问测试地址 - [http://localhost:81/consumer/payment/zk(opens new window)](http://localhost:81/consumer/payment/zk)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220814/image.22vnyckev000.webp)

# Consul 服务注册与发现

## Consul 简介

## 安装并运行Consul

[官网安装说明(opens new window)](https://learn.hashicorp.com/tutorials/consul/get-started-install)

windows版解压缩后，得consul.exe，打开cmd

- 查看版本`consul -v`：

```sh
DELL@FRXcomputer MINGW64 /d/DevelopTools/Consul/consul_1.13.1_windows_amd64
$ ./consul.exe --version
Consul v1.13.1
Revision c6d0f9ec
Build Date 2022-08-11T19:07:00Z
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
```

- 开发模式启动`consul agent -dev`：

浏览器输入 - http://localhost:8500/ - 打开Consul控制页。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.5dwbr6vcxb40.webp)

## 服务提供者注册进Consul

1. 新建Module支付服务cloud-providerconsul-payment8006
2. POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-providerconsul-payment8006</artifactId>

    <dependencies>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.frx01.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--SpringCloud consul-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>RELEASE</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>RELEASE</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

3. YML

```yaml
###consul服务端口号
server:
  port: 8006

spring:
  application:
    name: consul-provider-payment
  ####consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}
```

4. 主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8006 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8006.class,args);
    }
}
```

5. 业务类Controller

```java
@RestController
@Slf4j
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/consul")
    public String paymentConsul(){
        return "springcloud with consul:"+serverPort+"\t"+ UUID.randomUUID().toString();
    }
}
```

6.验证测试

- [http://localhost:8006/payment/consul(opens new window)](http://localhost:8006/payment/consul)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.1cgi418bn9gg.webp)

- [http://localhost:8500(opens new window)](http://localhost:8500/)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.5o6cr4jeom40.webp)

## 服务消费者注册进Consul

1. 新建Module消费服务order80 - cloud-consumerconsul-order80
2. POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-consumerconsul-order80</artifactId>
    <dependencies>
        <!--SpringCloud consul-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

3. YML

```yaml
###consul服务端口号
server:
  port: 81

spring:
  application:
    name: cloud-consumer-payment
  ####consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}
```

> 80端口被占用了 ，我换为了81

4. 主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderConsulMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderConsulMain80.class,args);
    }
}
```

5. 配置类

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

6. Controller

```java
@RestController
public class OrderConsulController {

    public static final String INVOKE_URL = "http://consul-provider-payment";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/payment/consul")
    public String paymentInfo(){
        String result = restTemplate.getForObject(INVOKE_URL+"/payment/consul",String.class);
        return result;
    }
}
```

7. 测试

- 运行consul，cloud-providerconsul-payment8006，cloud-consumerconsul-order80

  http://localhost:8500/ 主页会显示出consul，cloud-providerconsul-payment8006，cloud-consumerconsul-order80三服务。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.aof43930jgo.webp)

- 访问测试地址 - [http://localhost:81/consumer/payment/consul(opens new window)](http://localhost:81/consumer/payment/consul)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.4tnw181x9hu0.webp)

## 三个注册中心异同点

| 组件名    | 语言CAP | 服务健康检查 | 对外暴露接口 | Spring Cloud集成 |
| --------- | ------- | ------------ | ------------ | ---------------- |
| Eureka    | Java    | AP           | 可配支持     | HTTP             |
| Consul    | Go      | CP           | 支持         | HTTP/DNS         |
| Zookeeper | Java    | CP           | 支持客户端   | 已集成           |

CAP：

- C：Consistency (强一致性)
- A：Availability (可用性)
- P：Partition tolerance （分区容错性)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.94hpz9x1yb0.webp)

**最多只能同时较好的满足两个**。

CAP理论的核心是：**一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求**。

因此，根据CAP原理将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类:

- CA - 单点集群，满足—致性，可用性的系统，通常在可扩展性上不太强大。
- CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。
- AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

AP架构（Eureka）

当网络分区出现后，为了保证可用性，系统B可以返回旧值，保证系统的可用性。

结论：违背了一致性C的要求，只满足可用性和分区容错，即AP

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.edjjxibsaag.webp)

CP架构（ZooKeeper/Consul）

当网络分区出现后，为了保证一致性，就必须拒接请求，否则无法保证一致性。

结论：违背了可用性A的要求，只满足一致性和分区容错，即CP。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.29aygpmd0wu8.webp)

CP 与 AP 对立同一的矛盾关系。

# Ribbon 负载均衡服务调用

## Ribbon入门介绍

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**客户端负载均衡的工具**。

主要功能是提供**客户端的软件负载均衡算法和服务调用**。

**Ribbon本地负载均衡客户端VS Nginx服务端负载均衡区别**

Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即负载均衡是由服务端实现的。Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。

**集中式LB**

即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx)，由该设施负责把访问请求通过某种策略转发至服务的提供方;

**进程内LB**

将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

**Ribbon就属于进程内LB**，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。

**一句话**

负载均衡 + RestTemplate调用

## Ribbon的负载均衡和Rest调用

**架构说明**

总结：Ribbon其实就是一个软负载均衡的客户端组件，它可以和其他所需请求的客户端结合使用，和Eureka结合只是其中的一个实例。

Ribbon在工作时分成两步：

第一步先选择EurekaServer ,它优先选择在同一个区域内负载较少的server。

第二步再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。

其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。

**POM**

先前工程项目没有引入spring-cloud-starter-ribbon也可以使用ribbon。

```xml
<dependency>
    <groupld>org.springframework.cloud</groupld>
    <artifactld>spring-cloud-starter-netflix-ribbon</artifactid>
</dependency>
```

这是因为spring-cloud-starter-netflix-eureka-client自带了spring-cloud-starter-ribbon引用。

## RestTemplate的使用

**getForObject() / getForEntity() **- GET请求方法

getForObject()：返回对象为响应体中数据转化成的对象，基本上可以理解为Json。

getForEntity()：返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等。

```java
    @GetMapping("/consumer/payment/getForEntity/{id}")
    public CommonResult<Payment> getPayment2(@PathVariable("id") Long id){
        ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL + "/payment/get", CommonResult.class);

        if(entity.getStatusCode().is2xxSuccessful()){
            return entity.getBody();
        }else {
            return new CommonResult<>(444,"操作失败");
        }
    }
```

**postForObject() / postForEntity()** - POST请求方法

- 测试

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.5idpiwxwga40.webp)

## Ribbon默认自带的负载规则

lRule：根据特定算法中从服务列表中选取一个要访问的服务

- RoundRobinRule 轮询
- RandomRule 随机
- RetryRule 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试
- WeightedResponseTimeRule 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- BestAvailableRule 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
- AvailabilityFilteringRule 先过滤掉故障实例，再选择并发较小的实例
- ZoneAvoidanceRule 默认规则,复合判断server所在区域的性能和server的可用性选择服务

## Ribbon负载规则替换

1. 修改cloud-consumer-order80
2. 注意配置细节

官方文档明确给出了警告:

这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，

否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了。

（**也就是说不要将Ribbon配置类与主启动类同包**）

1. 新建package - com.frx01.myrule
2. 在com.frx01.myrule下新建MySelfRule规则类

```java
@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule(){
        return new RandomRule();
    }
}
```

1. 主启动类添加@RibbonClient

```java
@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}
```

1. 测试

- 开启cloud-eureka-server7001，cloud-consumer-order80，cloud-provider-payment8001，cloud-provider-payment8002
- 浏览器-输入[http://localhost:81/consumer/payment/get/1(opens new window)](http://localhost:81/consumer/payment/get/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/image.295ji74h92v4.webp)

返回结果中的serverPort在8001与8002两种间反复横跳。

## Ribbon默认负载轮询算法原理

## Ribbon默认负载轮询算法原理

默认负载轮训算法: rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标，每次服务重启动后rest接口计数从1开始。

```
List<Servicelnstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
```

如:

- List [0] instances = 127.0.0.1:8002
- List [1] instances = 127.0.0.1:8001

8001+ 8002组合成为集群，它们共计2台机器，集群总数为2，按照轮询算法原理：

- 当总请求数为1时:1%2=1对应下标位置为1，则获得服务地址为127.0.0.1:8001
- 当总请求数位2时:2%2=0对应下标位置为0，则获得服务地址为127.0.0.1:8002
- 当总请求数位3时:3%2=1对应下标位置为1，则获得服务地址为127.0.0.1:8001
- 当总请求数位4时:4%2=0对应下标位置为0，则获得服务地址为127.0.0.1:8002
- 如此类推…

## RoundRobinRule源码分析

```java
package com.netflix.loadbalancer;

public interface IRule {
    Server choose(Object var1);

    void setLoadBalancer(ILoadBalancer var1);

    ILoadBalancer getLoadBalancer();
}
```

```java
package com.netflix.loadbalancer;

import com.netflix.client.config.IClientConfig;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * The most well known and basic load balancing strategy, i.e. Round Robin Rule.
 *
 * @author stonse
 * @author Nikos Michalakis <nikos@netflix.com>
 *
 */
public class RoundRobinRule extends AbstractLoadBalancerRule {

    private AtomicInteger nextServerCyclicCounter;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;

    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        nextServerCyclicCounter = new AtomicInteger(0);
    }

    public RoundRobinRule(ILoadBalancer lb) {
        this();
        setLoadBalancer(lb);
    }

    //重点关注这方法。
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }

        Server server = null;
        int count = 0;
        while (server == null && count++ < 10) {
            List<Server> reachableServers = lb.getReachableServers();
            List<Server> allServers = lb.getAllServers();
            int upCount = reachableServers.size();
            int serverCount = allServers.size();//2

            if ((upCount == 0) || (serverCount == 0)) {
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }

            int nextServerIndex = incrementAndGetModulo(serverCount);//下标位置
            server = allServers.get(nextServerIndex);//0 1 0 1 0 1...

            if (server == null) {
                /* Transient. */
                Thread.yield();
                continue;
            }

            if (server.isAlive() && (server.isReadyToServe())) {
                return (server);
            }

            // Next.
            server = null;
        }

        if (count >= 10) {
            log.warn("No available alive servers after 10 tries from load balancer: "
                    + lb);
        }
        return server;
    }

    /**
     * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
     *
     * @param modulo The modulo to bound the value of the counter.
     * @return The next value.
     */
    private int incrementAndGetModulo(int modulo) {
        for (;;) {
            int current = nextServerCyclicCounter.get();
            int next = (current + 1) % modulo;//求余法
            if (nextServerCyclicCounter.compareAndSet(current, next))//比较并交换
                return next;
        }
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}
```

##  Ribbon之手写轮询算法

自己试着写一个类似RoundRobinRule的本地负载均衡器。

- 7001/7002集群启动
- 8001/8002微服务改造- controller

```java
@RestController
@Slf4j
public class PaymentController{

    ...
    
	@GetMapping(value = "/payment/lb")
    public String getPaymentLB() {
        return serverPort;//返回服务接口
    }
    
    ...
}
```

- 80订单微服务改造

1. ApplicationContextConfig去掉注解@LoadBalanced，OrderMain80去掉注解@RibbonClient
2. 创建LoadBalancer接口

```java
/**
 * @author frx
 * @version 1.0
 * @date 2022/8/16  0:55
 desc:收集现在服务器集群上总共有多少台能够提供服务的机器，放到List里面
 */
public interface LoadBalancer {

    ServiceInstance instances(List<ServiceInstance> serviceInstance);

}
```

3. 创建实现类MyLB，实现LoadBalancer接口

```java
/**
 * @author frx
 * @version 1.0
 * @date 2022/8/16  0:58
 */
@Component
public class MyLB implements LoadBalancer {

    private AtomicInteger atomicInteger = new AtomicInteger(0);//初始值为0

    public final int getAndIncrement(){
        int current;
        int next;
        do{
            current = this.atomicInteger.get();//得到初始值
            next = current >= 2147483647 ? 0 : current + 1;
        //比较，如果当前值current与期望值一致，就修改 返回true 取反 跳出循环
        }while (!this.atomicInteger.compareAndSet(current,next));
        System.out.println("-----第几次访问，次数next-----:"+next);//
        return next;
    }

    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstance) {

        int index = getAndIncrement() % serviceInstance.size();
        return serviceInstance.get(index);
    }
}
```

4. OrderController

```java
@RestController
@Slf4j
public class OrderController {

    //public static final String PAYMENT_URL= "http://localhost:8001";
    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";//eureka注册的服务名称
	
    //...

    @Resource
    private LoadBalancer loadBalancer;

    @Resource
    private DiscoveryClient discoveryClient;    
	@GetMapping(value = "/consumer/payment/lb")
    public String getPaymentLB(){
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        if(instances == null || instances.size() <= 0){
            return null;
        }
        ServiceInstance serviceInstance = loadBalancer.instances(instances);
        URI uri = serviceInstance.getUri();
        return restTemplate.getForObject(uri+"/payment/lb",String.class);
    }
    
    //...
}
```

5. 测试 不停地刷新[http://localhost:81/consumer/payment/lb (opens new window)](http://localhost:81/consumer/payment/lb)，可以看到8001/8002交替出现。

![QQ22918914922917714320220816013642](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220815/QQ22918914922917714320220816013642.2mrhybjr73g0.gif)

# OpenFeign 简化服务调用

eign是一个声明式WebService客户端。使用Feign能让编写Web Service客户端更加简单。它的使用方法是**定义一个服务接口然后在上面添加注解**。

**Feign集成了Ribbon**

利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，**通过feign只需要定义服务绑定接口且以声明式的方法**，优雅而简单的实现了服务调用。

OpenFeign是Spring Cloud在Feign的基础上支持了SpringMVC的注解，如@RequesMapping等等。OpenFeign的@Feignclient可以解析SpringMVc的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。

## OpenFeign服务调用

接口+注解：微服务调用接口 + @FeignClient

1. 新建cloud-consumer-feign-order80
2. POM

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-consumer-feign-order80</artifactId>

    <dependencies>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--eureka client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.frx01.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般基础通用配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>

```

3. YML

```yaml
server:
  port: 81

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```

4. 主启动

```java
@SpringBootApplication
@EnableFeignClients
public class OrderFeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class,args);
    }
}
```

5. 业务类

业务逻辑接口+@FeignClient配置调用provider服务

新建PaymentFeignService接口并新增注解@FeignClien

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {

    @GetMapping(value = "/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
}
```

控制层Controller

```java
@RestController
@Slf4j
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        return paymentFeignService.getPaymentById(id);
    }
}
```

6. 测试

先启动2个eureka集群7001/7002

再启动2个微服务8001/8002

启动OpenFeign启动

- [http://localhost:81/consumer/payment/get/1(opens new window)](http://localhost:81/consumer/payment/get/1)

![QQ22918914922917714320220816235147](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220816/QQ22918914922917714320220816235147.2cgl1whlgoys.gif)

Feign自带负载均衡配置项

## OpenFeign超时控制

**超时设置，故意设置超时演示出错情况**

1. 服务提供方8001/8002故意写暂停程序

```java
@RestController
@Slf4j
public class PaymentController {
    
    //...
    
    @Value("${server.port}")
    private String serverPort;

    //...
    
    @GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeout()
    {
        // 业务逻辑处理正确，但是需要耗费3秒钟
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return serverPort;
    }
    
    //...
}
```

2. 服务消费方80添加超时方法PaymentFeignService

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {
	
    //...
    @GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeout();
}
```

3. 服务消费方80添加超时方法OrderFeignController

```java
@RestController
@Slf4j
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    //...

    @GetMapping(value = "/consumer/payment/feign/timeout")
    public String paymentFeignTimeout(){
        //openfeign-ribbon，客户端一般默认等待1秒钟
        return paymentFeignService.paymentFeignTimeout();
    }
}
```

4. 测试

多次刷新+ [http://localhost:81/consumer/payment/feign/timeout(opens new window)](http://localhost:81/consumer/payment/feign/timeout)

将会跳出错误Spring Boot默认错误页面，主要异常：`feign.RetryableException:Read timed out executing GET http://CLOUD-PAYMENT-SERVCE/payment/feign/timeout`。

**OpenFeign默认等待1秒钟，超过后报错**

**YML文件里需要开启OpenFeign客户端超时控制**

```yaml
#设置feign客户端超时时间(OpenFeign默认支持ribbon)(单位：毫秒)
ribbon:
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

- 重新访问

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220816/image.3e2gqggpype0.webp)

## OpenFeign日志增强

**日志打印功能**

Feign提供了日志打印功能，我们可以通过配置来调整日恙级别，从而了解Feign 中 Http请求的细节。

说白了就是对Feign接口的调用情况进行监控和输出

**日志级别**

- NONE：默认的，不显示任何日志;
- BASIC：仅记录请求方法、URL、响应状态码及执行时间;
- HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息;
- FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据。

**配置日志bean**

```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}
```

**YML文件里需要开启日志的Feign客户端**

```yaml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.frx01.springcloud.service.PaymentFeignService: debug
```

**后台日志查看**

再次访问[http://localhost:81/consumer/payment/get/1 (opens new window)](http://localhost:81/consumer/payment/get/1)，得到更多日志信息。

<details class="custom-block details" open="" style="display: block; position: relative; border-radius: 2px; margin: 1em 0px; padding: 1.6em; background-color: var(--customBlockBg); border: 1px solid rgb(221, 221, 221);"><summary style="outline: none; cursor: pointer;">控制台打印</summary><div class="language-java line-numbers-mode" style="overflow: hidden; transition: height 0.3s ease 0s; margin-top: 0.85rem; position: relative; background-color: var(--codeBg); border-radius: 6px; padding-bottom: 0.5rem; height: 331px;"><i class="code-copy" title="复制失败" style="color: rgb(170, 170, 170); fill: rgba(238, 255, 255, 0.8); font-size: 14px; display: inline-block; cursor: pointer; position: absolute; top: 0.8rem; right: 2rem; opacity: 1;"><svg style="color:#aaa;font-size:14px" t="1572422231464" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3201" width="14" height="14"><path d="M866.461538 39.384615H354.461538c-43.323077 0-78.769231 35.446154-78.76923 78.769231v39.384616h472.615384c43.323077 0 78.769231 35.446154 78.769231 78.76923v551.384616h39.384615c43.323077 0 78.769231-35.446154 78.769231-78.769231V118.153846c0-43.323077-35.446154-78.769231-78.769231-78.769231z m-118.153846 275.692308c0-43.323077-35.446154-78.769231-78.76923-78.769231H157.538462c-43.323077 0-78.769231 35.446154-78.769231 78.769231v590.769231c0 43.323077 35.446154 78.769231 78.769231 78.769231h512c43.323077 0 78.769231-35.446154 78.76923-78.769231V315.076923z m-354.461538 137.846154c0 11.815385-7.876923 19.692308-19.692308 19.692308h-157.538461c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h157.538461c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z m157.538461 315.076923c0 11.815385-7.876923 19.692308-19.692307 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h315.076923c11.815385 0 19.692308 7.876923 19.692307 19.692308v39.384615z m78.769231-157.538462c0 11.815385-7.876923 19.692308-19.692308 19.692308H216.615385c-11.815385 0-19.692308-7.876923-19.692308-19.692308v-39.384615c0-11.815385 7.876923-19.692308 19.692308-19.692308h393.846153c11.815385 0 19.692308 7.876923 19.692308 19.692308v39.384615z" p-id="3202"></path></svg></i><pre class="language-java codecopy-enabled" style="position: relative !important; user-select: text; line-height: 1.4; padding: 1.25rem 1.5rem 1.25rem 3.5rem; margin: 30px 0px 0.85rem; background: transparent; border-radius: 6px; overflow: auto; z-index: 1; color: rgb(204, 204, 204); text-shadow: none; font-family: Consolas, Monaco, &quot;Andale Mono&quot;, &quot;Ubuntu Mono&quot;, monospace; font-size: 1em; text-align: left; white-space: pre; word-spacing: normal; word-break: normal; overflow-wrap: normal; tab-size: 4; hyphens: none; vertical-align: middle;"><code style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; color: var(--codeColor); padding: 0px; margin: 0px; font-size: 0.9em; background-color: transparent; border-radius: 0px;"><span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.362</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-&gt;</span> <span class="token constant" style="color: rgb(248, 197, 85);">GET</span> http<span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">/</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">/</span><span class="token constant" style="color: rgb(248, 197, 85);">CLOUD</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token constant" style="color: rgb(248, 197, 85);">PAYMENT</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token constant" style="color: rgb(248, 197, 85);">SERVICE</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">/</span>payment<span class="token operator" style="color: rgb(103, 205, 204); background: none;">/</span>get<span class="token operator" style="color: rgb(103, 205, 204); background: none;">/</span><span class="token number" style="color: rgb(240, 141, 73);">1</span> <span class="token constant" style="color: rgb(248, 197, 85);">HTTP</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">/</span><span class="token number" style="color: rgb(240, 141, 73);">1.1</span>
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.362</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-&gt;</span> <span class="token class-name" style="color: rgb(248, 197, 85);">END</span> <span class="token constant" style="color: rgb(248, 197, 85);">HTTP</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token number" style="color: rgb(240, 141, 73);">0</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token keyword" style="color: rgb(204, 153, 205);">byte</span> body<span class="token punctuation" style="color: rgb(204, 204, 204);">)</span>
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.369</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">&lt;</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token constant" style="color: rgb(248, 197, 85);">HTTP</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">/</span><span class="token number" style="color: rgb(240, 141, 73);">1.1</span> <span class="token number" style="color: rgb(240, 141, 73);">200</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token number" style="color: rgb(240, 141, 73);">6</span>ms<span class="token punctuation" style="color: rgb(204, 204, 204);">)</span>
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.369</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> connection<span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> keep<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>alive
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.369</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> content<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>type<span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> application<span class="token operator" style="color: rgb(103, 205, 204); background: none;">/</span>json
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.369</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> date<span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Tue</span><span class="token punctuation" style="color: rgb(204, 204, 204);">,</span> <span class="token number" style="color: rgb(240, 141, 73);">16</span> <span class="token class-name" style="color: rgb(248, 197, 85);">Aug</span> <span class="token number" style="color: rgb(240, 141, 73);">2022</span> <span class="token number" style="color: rgb(240, 141, 73);">16</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09</span> <span class="token constant" style="color: rgb(248, 197, 85);">GMT</span>
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.370</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> keep<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>alive<span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> timeout<span class="token operator" style="color: rgb(103, 205, 204); background: none;">=</span><span class="token number" style="color: rgb(240, 141, 73);">60</span>
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.370</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> transfer<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>encoding<span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> chunked
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.370</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> 
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.370</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">{</span><span class="token string" style="color: rgb(126, 198, 153);">"code"</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">200</span><span class="token punctuation" style="color: rgb(204, 204, 204);">,</span><span class="token string" style="color: rgb(126, 198, 153);">"message"</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token string" style="color: rgb(126, 198, 153);">"查询成功,serverPort:8002"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">,</span><span class="token string" style="color: rgb(126, 198, 153);">"data"</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token punctuation" style="color: rgb(204, 204, 204);">{</span><span class="token string" style="color: rgb(126, 198, 153);">"id"</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">1</span><span class="token punctuation" style="color: rgb(204, 204, 204);">,</span><span class="token string" style="color: rgb(126, 198, 153);">"serial"</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token string" style="color: rgb(126, 198, 153);">"水果"</span><span class="token punctuation" style="color: rgb(204, 204, 204);">}</span><span class="token punctuation" style="color: rgb(204, 204, 204);">}</span>
<span class="token number" style="color: rgb(240, 141, 73);">2022</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">08</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">17</span> <span class="token number" style="color: rgb(240, 141, 73);">00</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">41</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span><span class="token number" style="color: rgb(240, 141, 73);">09.370</span> <span class="token constant" style="color: rgb(248, 197, 85);">DEBUG</span> <span class="token number" style="color: rgb(240, 141, 73);">18640</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span>p<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>nio<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">81</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span>exec<span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token number" style="color: rgb(240, 141, 73);">6</span><span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token class-name" style="color: rgb(248, 197, 85);"><span class="token namespace" style="opacity: 0.7; color: rgb(226, 119, 122);">c<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>f<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>s<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span>service<span class="token punctuation" style="color: rgb(204, 204, 204);">.</span></span>PaymentFeignService</span>        <span class="token operator" style="color: rgb(103, 205, 204); background: none;">:</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">[</span><span class="token class-name" style="color: rgb(248, 197, 85);">PaymentFeignService</span>#getPaymentById<span class="token punctuation" style="color: rgb(204, 204, 204);">]</span> <span class="token operator" style="color: rgb(103, 205, 204); background: none;">&lt;</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">--</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span> <span class="token class-name" style="color: rgb(248, 197, 85);">END</span> <span class="token constant" style="color: rgb(248, 197, 85);">HTTP</span> <span class="token punctuation" style="color: rgb(204, 204, 204);">(</span><span class="token number" style="color: rgb(240, 141, 73);">87</span><span class="token operator" style="color: rgb(103, 205, 204); background: none;">-</span><span class="token keyword" style="color: rgb(204, 153, 205);">byte</span> body<span class="token punctuation" style="color: rgb(204, 204, 204);">)</span>
</code></pre><div class="line-numbers-wrapper" style="position: absolute; top: 0px; width: 2.5rem; text-align: center; color: rgb(158, 158, 158); padding: 1.25rem 0px; line-height: 1.4; margin-top: 30px;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">1</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">2</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">3</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">4</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">5</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">6</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">7</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">8</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">9</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">10</span><br style="user-select: none;"><span class="line-number" style="font-family: source-code-pro, Menlo, Monaco, Consolas, &quot;Courier New&quot;, monospace; position: relative; z-index: 4; user-select: none; font-size: 0.85em;">11</span><br style="user-select: none;"></div><div class="expand icon-xiangxiajiantou iconfont" style="font-size: 16px; font-style: normal; -webkit-font-smoothing: antialiased; font-family: iconfont !important; width: 16px; height: 16px; cursor: pointer; position: absolute; z-index: 3; top: 0.8em; right: 0.5em; color: rgba(238, 255, 255, 0.8); font-weight: 900; transition: transform 0.3s ease 0s;"></div></div></details>

# Hystrix 服务降级|熔断

Hystrix是一个用于处理分布式系统的**延迟**和**容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，**不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性**。

## Hystrix停更进维

**能干嘛**

- 服务降级
- 服务熔断
- 接近实对的监控

## Hystrix的服务降级熔断限流概念初讲

**服务降级**

服务器忙，请稍后再试，不让客户端等待并立刻返回一个友好提示，fallback

**哪些情况会出发降级**

- 程序运行导常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满也会导致服务降级

**服务熔断**

**类比保险丝**达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示。

服务的降级 -> 进而熔断 -> 恢复调用链路

**服务限流**

秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行。

## Hystrix支付微服务构建

将cloud-eureka-server7001改配置成单机版

1. 新建cloud-provider-hygtrix-payment8001
2. POM

3. yml

```yaml
server:
  port: 8001

spring:
  application:
    name: cloud-provider-hystrix-payment

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
      defaultZone: http://eureka7001.com:7001/eureka
```

4. 主启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentHystrixMain8001 {

    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class,args);
    }
}
```

5. 业务类

service

```java
@Service
public class PaymentService {

    /**
     * 正常访问，肯定OK
     * @param id
     * @return
     */
    public String paymentInfo_OK(Integer id){
        return "线程池: "+Thread.currentThread().getName()+"    paymentInfo_OK,id: "+id+"\t"+"^_^";
    }

    public String paymentInfo_TimeOut(Integer id){

        int timeNumber = 3000;
        try {
            Thread.sleep(timeNumber);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池: "+Thread.currentThread().getName()+"    paymentInfo_OK,id: "+id+"\t"+"^_^"+"  耗时(秒):"+timeNumber;
    }
}
```

Controller

```java
@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "payment/hystrix/ok/{id}")
    public String paymentInfo_Ok(@PathVariable("id") Integer id) {
        String result = paymentService.paymentInfo_OK(id);
        log.info("---result:" + result);
        return result;
    }


    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
        String result = paymentService.paymentInfo_TimeOut(id);
        log.info("---result: " + result);
        return result;
    }
}
```

6. 正常测试

启动eureka7001

启动cloud-provider-hystrix-payment8001

访问

success的方法 - [http://localhost:8001/payment/hystrix/ok/1(opens new window)](http://localhost:8001/payment/hystrix/ok/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220818/image.3fzl480bxq20.webp)

每次调用耗费3秒钟 - [http://localhost:8001/payment/hystrix/timeout/1(opens new window)](http://localhost:8001/payment/hystrix/timeout/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220818/image.11b5wbupqofk.webp)

上述module均OK

以上述为根基平台，从正确 -> 错误 -> 降级熔断 -> 恢复。

## JMeter高并发压测后卡顿

**述在非高并发情形下，还能勉强满足**

**Jmeter压测测试**

开启Jmeter，来20000个并发压死8001，20000个请求都去访问paymentInfo_TimeOut服务

1. 测试计划中右键添加-》线程-》线程组（线程组202102，线程数：200，线程数：100，其他参数默认）
2. 刚刚新建线程组202102，右键它-》添加-》取样器-》Http请求-》基本 输入http://localhost:8001/payment/hystrix/ok/1
3. 点击绿色三角形图标启动。

看演示结果：拖慢，原因：tomcat的默认的工作线程数被打满了，没有多余的线程来分解压力和处理。

Jmeter压测结论

上面还是服务提供者8001自己测试，假如此时外部的消费者80也来访问，那消费者只能干等，最终导致消费端80不满意，服务端8001直接被拖慢

##  订单微服务调用支付服务出现卡顿

**看热闹不嫌弃事大，80新建加入**

1. 新建 - cloud-consumer-feign-hystrix-order80
2. POM

3. YML

```YML
server:
  port: 81

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

4. 主启动类

```java
@SpringBootApplication
@EnableFeignClients
public class OrderHystrixMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderHystrixMain80.class,args);
    }
}
```

5. 业务类

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT")
public interface PaymentHystrixService {

    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}
```

```java
@RestController
@Slf4j
public class OrderHystrixController {

    @Resource
    private PaymentHystrixService paymentHystrixService;

    @GetMapping(value ="/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    @GetMapping(value ="/consumer/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }
}
```

6. 正常测试

启动eureka7001

启动cloud-provider-hystrix-payment8001

启动OrderHystrixMain80

7. 高并发测试

2W个线程压8001

消费端80微服务再去访问正常的Ok微服务8001地址

消费者80被拖慢

原因：8001同一层次的其它接口服务被困死，因为tomcat线程池里面的工作线程已经被挤占完毕。

正因为有上述故障或不佳表现才有我们的降级/容错/限流等技术诞生。

## 降级容错解决的维度要求

时导致服务器变慢(转圈) - 超时不再等待

出错(宕机或程序运行出错) - 出错要有兜底

解决：

- 对方服务(8001)超时了，调用者(80)不能一直卡死等待，必须有服务降级。
- 对方服务(8001)down机了，调用者(80)不能一直卡死等待，必须有服务降级。
- 对方服务(8001)OK，调用者(80)自己出故障或有自我要求(自己的等待时间小于服务提供者)，自己处理降级。

## Hystrix之服务降级支付侧fallback

降级配置 - @HystrixCommand

8001先从自身找问题

**设置自身调用超时时间的峰值，峰值内可以正常运行，超过了需要有兜底的方法处埋，作服务降级fallback。**

**8001fallback**

业务类启用 - @HystrixCommand报异常后如何处理

—旦调用服务方法失败并抛出了错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法

```java
@Service
public class PaymentService {

    /**
     * 正常访问，肯定OK
     * @param id
     * @return
     */
    public String paymentInfo_OK(Integer id){
        return "线程池: "+Thread.currentThread().getName()+"    paymentInfo_OK,id: "+id+"\t"+"^_^";
    }
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value="3000")
    })
    public String paymentInfo_TimeOut(Integer id){

        int timeNumber = 5000;
        //int age = 10/0;
        try {
            Thread.sleep(timeNumber);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池: "+Thread.currentThread().getName()+"    paymentInfo_TimeOut,id: "+id+"\t"+"^_^"+"  耗时(秒):"+timeNumber;
    }

    public String paymentInfo_TimeOutHandler(Integer id){
        return "线程池: "+Thread.currentThread().getName()+"8001系统繁忙或者运行报错，请稍后再试,,id: "+id+"\t"+"呜呜";
    }
}
```

上面故意制造两种异常:

1. int age = 10/0，计算异常
2. 我们能接受3秒钟，它运行5秒钟，超时异常。

当前服务不可用了，做服务降级，兜底的方案都是paymentInfo_TimeOutHandler

**主启动类激活**

添加新注解@EnableCircuitBreaker

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixMain8001 {

    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class,args);
    }
}
```

- 测试启动
- [http://localhost:8001/payment/hystrix/timeout/1(opens new window)](http://localhost:8001/payment/hystrix/timeout/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220818/image.vgg8bktmvsw.webp)

## Hystrix之服务降级订单侧fallback

80订单微服务，也可以更好的保护自己，自己也依样画葫芦进行客户端降级保护

题外话，切记 - 我们自己配置过的热部署方式对java代码的改动明显

但对@HystrixCommand内属性的修改建议重启微服务

YML

```yaml
erver:
  port: 81

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/

#开启
feign:
  hystrix:
    enabled: true
```

主启动

```java
@SpringBootApplication
@EnableFeignClients
@EnableHystrix
public class OrderHystrixMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderHystrixMain80.class,args);
    }
}
```

业务类

```java
@RestController
@Slf4j
public class OrderHystrixController {

    @Resource
    private PaymentHystrixService paymentHystrixService;

    @GetMapping(value ="/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1500")
    })
    @GetMapping(value ="/consumer/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        int age = 10/0;
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }

    //善后方法
    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id){
        return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }
}
```

- 测试 [http://localhost:81/consumer/payment/hystrix/timeout/1(opens new window)](http://localhost:81/consumer/payment/hystrix/timeout/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220818/image.73z2czmcp2c0.webp)

## Hystrix之全局服务降级DefaultProperties

**目前问题1** 每个业务方法对应一个兜底的方法，代码膨胀

**解决方法**

1:1每个方法配置一个服务降级方法，技术上可以，但是不聪明

1:N除了个别重要核心业务有专属，其它普通的可以通过`@DefaultProperties(defaultFallback = “”)`统一跳转到统一处理结果页面

通用的和独享的各自分开，避免了代码膨胀，合理减少了代码量

```java
@RestController
@Slf4j
@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
public class OrderHystrixController {

    @Resource
    private PaymentHystrixService paymentHystrixService;

    @GetMapping(value ="/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    //@HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
    //        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1500")
    //})
    @HystrixCommand///用全局的fallback方法
    @GetMapping(value ="/consumer/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        int age = 10/0;
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }

    //善后方法
    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id){
        return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }

    //下面全局fallback方法
    public String payment_Global_FallbackMethod(){
        return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
    }
}
```

- 测试 [http://localhost:81/consumer/payment/hystrix/timeout/1(opens new window)](http://localhost:81/consumer/payment/hystrix/timeout/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220818/image.45t34hbzs9c0.webp)

##  Hystrix之通配服务降级FeignFallback

**目前问题2** 统一和自定义的分开，代码混乱

**服务降级，客户端去调用服务端，碰上服务端宕机或关闭**

本次案例服务降级处理是在客户端80实现完成的，与服务端8001没有关系，只需要为[Feign](https://frxcat.fun/Spring/SpringCloud/OpenFeign_/#openfeign服务调用)客户端定义的接口添加一个服务降级处理的实现类即可实现解耦

**未来我们要面对的异常**

- 运行
- 超时
- 宕机

**修改cloud-consumer-feign-hystrix-order80**

根据cloud-consumer-feign-hystrix-order80已经有的PaymentHystrixService接口， 重新新建一个类(AaymentFallbackService)实现该接口，统一为接口里面的方法进行异常处理

PaymentFallbackService类实现PaymentHystrixService接口

```java
@SuppressWarnings("all")
@Component
public class PaymentFallbackService implements PaymentHystrixService{
    @Override
    public String paymentInfo_OK(Integer id) {
        return "-----PaymentFallbackService fall back-paymentInfo_OK ,o(╥﹏╥)o";
    }

    @Override
    public String paymentInfo_TimeOut(Integer id) {
        return "-----PaymentFallbackService fall back-paymentInfo_TimeOut ,o(╥﹏╥)o";
    }
}
```

YML

```yaml
server:
  port: 81

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/

#开启
feign:
  hystrix:
    enabled: true
```

PaymentHystrixService接口

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",
        fallback = PaymentFallbackService.class)//指定PaymentFallbackService类
public interface PaymentHystrixService {

    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}
```

**测试**

单个eureka先启动7001

PaymentHystrixMain8001启动

正常访问测试

- [http://localhost:81/consumer/payment/hystrix/ok/1(opens new window)](http://localhost:81/consumer/payment/hystrix/ok/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220818/image.4l885zrrwmc0.webp)

- 故意关闭微服务8001，再次访问

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220818/image.48uhagvgpc80.webp)

客户端自己调用提示 - 此时服务端provider已经down了，但是我们做了服务降级处理，让客户端在服务端不可用时也会获得提示信息而不会挂起耗死服务器。

##  Hystrix之服务熔断理论

断路器，相当于保险丝。

**熔断机制概述**

熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。**当检测到该节点微服务调用响应正常后，恢复调用链路。**

在Spring Cloud框架里，熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是`@HystrixCommand`。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220819/image.1uqftpm75ri8.webp)

## Hystrix之服务熔断案例(上)

修改cloud-provider-hystrix-payment8001

```java
@Service
public class PaymentService {

    //===========================服务熔断===========================
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),//是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),//请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),//时间窗口期
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")//失败率达到多少后跳闸 //6次
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id){
        if(id < 0){
            throw new RuntimeException("=======id 不能为负数");
        }
        String serialNumber = IdUtil.simpleUUID();//==UUID.randomUUID().toString()
        return Thread.currentThread().getName()+"\t"+"调用成功，流水号："+serialNumber;
    }

    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id){
        return "id 不能负数，请稍后再试，，/(ㄒoㄒ)/~~   id: "+id;
    }
}
```

The precise way that the circuit opening and closing occurs is as follows:

1. Assuming the volume across a circuit meets a certain threshold : `HystrixCommandProperties.circuitBreakerRequestVolumeThreshold()`
2. And assuming that the error percentage, as defined above exceeds the error percentage defined in :`HystrixCommandProperties.circuitBreakerErrorThresholdPercentage()`
3. Then the circuit-breaker transitions from `CLOSED` to `OPEN`.
4. While it is open, it short-circuits all requests made against that circuit-breaker.
5. After some amount of time (`HystrixCommandProperties.circuitBreakerSleepWindowInMilliseconds()`), the next request is let through. If it fails, the command stays OPEN for the sleep window. If it succeeds, it transitions to CLOSED and the logic in 1) takes over again.
6. 假设整个电路的音量满足某个阈值：`HystrixCommandProperties.circuitBreakerRequestVolumeThreshold()`
7. 并假设上面定义的错误百分比超过定义的错误百分比：`HystrixCommandProperties.circuitBreakerErrorThresholdPercentage()`
8. 然后断路器从 CLOSED 转变为 OPEN。
9. 当它打开时，它会将针对该断路器的所有请求短路。
10. 一段时间后（`HystrixCommandProperties.circuitBreakerSleepWindowInMilliseconds()`），下一个请求被允许通过。如果失败，该命令将在睡眠窗口中保持打开状态。如果成功，则转换为 CLOSED 并且 1) 中的逻辑再次接管。

HystrixCommandProperties配置类



```java
package com.netflix.hystrix;

...

public abstract class HystrixCommandProperties {
    private static final Logger logger = LoggerFactory.getLogger(HystrixCommandProperties.class);

    /* defaults */
    /* package */ static final Integer default_metricsRollingStatisticalWindow = 10000;// default => statisticalWindow: 10000 = 10 seconds (and default of 10 buckets so each bucket is 1 second)
    private static final Integer default_metricsRollingStatisticalWindowBuckets = 10;// default => statisticalWindowBuckets: 10 = 10 buckets in a 10 second window so each bucket is 1 second
    private static final Integer default_circuitBreakerRequestVolumeThreshold = 20;// default => statisticalWindowVolumeThreshold: 20 requests in 10 seconds must occur before statistics matter
    private static final Integer default_circuitBreakerSleepWindowInMilliseconds = 5000;// default => sleepWindow: 5000 = 5 seconds that we will sleep before trying again after tripping the circuit
    private static final Integer default_circuitBreakerErrorThresholdPercentage = 50;// default => errorThresholdPercentage = 50 = if 50%+ of requests in 10 seconds are failures or latent then we will trip the circuit
    private static final Boolean default_circuitBreakerForceOpen = false;// default => forceCircuitOpen = false (we want to allow traffic)
    /* package */ static final Boolean default_circuitBreakerForceClosed = false;// default => ignoreErrors = false 
    private static final Integer default_executionTimeoutInMilliseconds = 1000; // default => executionTimeoutInMilliseconds: 1000 = 1 second
    private static final Boolean default_executionTimeoutEnabled = true;

    ...
}
```

## Hystrix之服务熔断案例(下)

```java
@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @Value("${server.port}")
    private String serverPort;
    
	//===========================服务熔断===========================
    @GetMapping("/payment/circuit/{id}")
    public String paymentCircuitBreaker(@PathVariable("id") Integer id){
        String result = paymentService.paymentCircuitBreaker(id);
        log.info("======result:"+result);
        return result;
    }

}
```

自测cloud-provider-hystrix-payment8001

正确 - [http://localhost:8001/payment/circuit/1(opens new window)](http://localhost:8001/payment/circuit/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220819/image.3oqulij7oc8.webp)

错误 - [http://localhost:8001/payment/circuit/-1(opens new window)](http://localhost:8001/payment/circuit/-1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220819/image.304htzvukzu0.webp)

多次错误，再来次正确，但错误得显示

重点测试 - 多次错误，然后慢慢正确，发现刚开始不满足条件，就算是正确的访问地址也不能进行

## Hystrix之服务熔断总结

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220819/image.mtzs1kresfk.webp)

**熔断类型**

- 熔断打开：请求不再进行调用当前服务，内部设置时钟一般为MTTR(平均故障处理时间)，当打开时长达到所设时钟则进入半熔断状态。
- 熔断关闭：熔断关闭不会对服务进行熔断。
- 熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断。

电路开闭发生的具体方式如下：

1. 假设整个电路的音量满足某个阈值：`HystrixCommandProperties.circuitBreakerRequestVolumeThreshold()`
2. 并假设上面定义的错误百分比超过定义的错误百分比：`HystrixCommandProperties.circuitBreakerErrorThresholdPercentage()`
3. 然后断路器从 CLOSED 转变为 OPEN。
4. 当它打开时，它会将针对该断路器的所有请求短路。
5. 一段时间后（`HystrixCommandProperties.circuitBreakerSleepWindowInMilliseconds()`），下一个请求被允许通过。如果失败，该命令将在睡眠窗口中保持打开状态。如果成功，则转换为 CLOSED 并且 1) 中的逻辑再次接管。

如果您包含单元测试，我可以查看您的特定请求序列，并更好地了解您希望看到的更改。

**断路器在什么情况下开始起作用**

```java
//=====服务熔断
@HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
    @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),// 请求次数
    @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), // 时间窗口期
    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),// 失败率达到多少后跳闸
})
public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
    ...
}
```

涉及到断路器的三个重要参数：

1. **快照时间窗**：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。
2. **请求总数阀值**：在快照时间窗内，必须满足请求总数阀值才有资格熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用次数不足20次7,即使所有的请求都超时或其他原因失败，断路器都不会打开。
3. **错误百分比阀值**：当请求总数在快照时间窗内超过了阀值，比如发生了30次调用，如果在这30次调用中，有15次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%阀值情况下，这时候就会将断路器打开。

**断路器开启或者关闭的条件**

- 到达以下阀值，断路器将会开启：
  - 当满足一定的阀值的时候（默认10秒内超过20个请求次数)
  - 当失败率达到一定的时候（默认10秒内超过50%的请求失败)
- 当开启的时候，所有请求都不会进行转发
- 一段时间之后（默认是5秒)，这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若失败，继续开启。

**断路器打开之后**

1. 再有请求调用的时候，将不会调用主逻辑，而是直接调用降级fallback。通过断路器，实现了自动地发现错误并将降级逻辑切换为主逻辑，减少响应延迟的效果。
2. 原来的主逻辑要如何恢复呢？

对于这一问题，hystrix也为我们实现了自动恢复功能。

当断路器打开，对主逻辑进行熔断之后，hystrix会启动一个休眠时间窗，在这个时间窗内，降级逻辑是临时的成为主逻辑，当休眠时间窗到期，断路器将进入半开状态，释放一次请求到原来的主逻辑上，如果此次请求正常返回，那么断路器将继续闭合，主逻辑恢复，如果这次请求依然有问题，断路器继续进入打开状态，休眠时间窗重新计时。

## Hystrix工作流程最后总结

**服务限流** - 后面高级篇讲解alibaba的Sentinel说明

## Hystrix图形化Dashboard搭建

**仪表盘9001**

1. 新建cloud-consumer-hystrix-dashboard9001
2. POM

3. YML

```yaml
server:
  port: 9001
```

4. 主启动类

```java
@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardMain9001.class,args);
    }
}
```

5. 所有Provider微服务提供类(8001/8002/8003)都需要监控依赖配置

```xml
<dependency
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

6. 启动cloud-consumer-hystrix-dashboard9001该微服务后续将监控微服务8001

浏览器输入[http://localhost:9001/hystrix(opens new window)](http://localhost:9001/hystrix)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220819/image.18mc8a651jo.webp)

## Hystrix图形化Dashboard监控实战

**修改cloud-provider-hystrix-payment8001**

注意：新版本Hystrix需要在主启动类PaymentHystrixMain8001中指定监控路径

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixMain8001 {

    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class,args);
    }

    /**
     *此配置是为了服务监控而配置，与服务容错本身无关，springcloud升级后的坑
     *ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
     *只要在自己的项目里配置上下面的servlet就可以了
     *否则，Unable to connect to Command Metric Stream 404
     */
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```

监控测试

启动1个eureka

启动8001，9001

观察监控窗口

9001监控8001 - 填写监控地址 - http://localhost:8001/hystrix.stream 到 http://localhost:9001/hystrix页面的输入框。

测试地址

- http://localhost:8001/payment/circuit/1
- http://localhost:8001/payment/circuit/-1
- 测试通过
- 先访问正确地址，再访问错误地址，再正确地址，会发现图示断路器都是慢慢放开的。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220819/image.5ylp5yy7ym80.webp)

**如何看?**

- 7色

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220819/image.4fh23x8szy00.webp)

- 1圈

实心圆：共有两种含义。它通过颜色的变化代表了实例的健康程度，它的健康度从绿色<黄色<橙色<红色递减。

该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化，**流量越大该实心圆就越大**。所以通过该实心圆的展示，就可以在大量的实例中快速的发现故障实例和高压力实例。

- 1线

曲线：用来记录2分钟内流量的相对变化，可以通过它来观察到流量的上升和下降趋势。

- 整图说明![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220819/image.703qlrecqrc0.webp)
- 整图说明2

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220819/image.luqrwab8ws0.webp)

# GateWay 服务网关

概述

Cloud全家桶中有个很重要的组件就是网关，在1.x版本中都是采用的Zuul网关;

但在2.x版本中，zuul的升级一直跳票，SpringCloud最后自己研发了一个网关替代Zuul，那就是SpringCloud Gateway—句话：gateway是原zuul1.x版的替代

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220820/image.73vmanwmamk0.webp)

**SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。**

**作用**

- 方向代理
- 鉴权
- 流量控制
- 熔断
- 日志监控
- …

**微服务架构中网关的位置**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220820/image.ardgdjmh40g.webp)

## GateWay非阻塞异步模型

1. SpringCloud Gateway具有如下特性
   1. 基于Spring Framework 5，Project Reactor和Spring Boot 2.0进行构建；
   2. 动态路由：能够匹配任何请求属性；
   3. 可以对路由指定Predicate (断言)和Filter(过滤器)；
   4. 集成Hystrix的断路器功能；
   5. 集成Spring Cloud 服务发现功能；
   6. 易于编写的Predicate (断言)和Filter(过滤器)；
   7. 请求限流功能；
   8. 支持路径重写。

##  GateWay工作流程

1. oute(路由) - 路由是构建网关的基本模块,它由ID,目标URI,一系列的断言和过滤器组成,如断言为true则匹配该路由；
2. Predicate(断言) - 参考的是Java8的java.util.function.Predicate，开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数),如果请求与断言相匹配则进行路由；
3. Filter(过滤) - 指的是Spring框架中GatewayFilter的实例,使用过滤器,可以在请求被路由前或者之后对请求进行修

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220820/image.3aoykivbqx40.webp)

web请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。

predicate就是我们的匹配条件；而fliter，就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标uri，就可以实现一个具体的路由了

**Gateway工作流程**

**核心逻辑**：路由转发 + 执行过滤器链。

## GateWay9527搭建

1. 新建Module - cloud-gateway-gateway9527
2. POM

3. YML

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

4. 主启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class GateWayMain9527
{
    public static void main(String[] args) {
        SpringApplication.run(GateWayMain9527.class, args);
    }
}
```

5. 业务类

无

1. 9527网关如何做路由映射?

cloud-provider-payment8001看看controller的访问地址

- get
- lb

我们目前不想暴露8001端口，希望在8001外面套一层9527

6. YML新增网关配置

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  #############################新增网关配置###########################
  cloud:
    gateway:
      routes:
        - id: payment_routh #payment_route     #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001           #匹配后提供服务的路由地址
          #uri: lb://cloud-payment-service     #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**             # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001           #匹配后提供服务的路由地址
          #uri: lb://cloud-payment-service     #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**              # 断言，路径相匹配的进行路由
####################################################################

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

7. 测试

- 启动7001

- 启动8001-cloud-provider-payment8001

- 启动9527网关

- 访问说明

  - 添加网关前 - [http://localhost:8001/payment/get/1(opens new window)](http://localhost:8001/payment/get/1)

  ![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220820/image.ju6bt2cf84w.webp)

  - 添加网关后 - [http://localhost:9527/payment/get/1(opens new window)](http://localhost:9527/payment/get/1)

  ![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220820/image.71br3536nu00.webp)

  - 两者访问成功，返回相同结果

## GateWay配置路由的两种方式

**在配置文件yml中配置，见上一章节**

**代码中注入RouteLocator的Bean**

```java
RemoteAddressResolver resolver = XForwardedRemoteAddressResolver
    .maxTrustedIndex(1);

...

.route("direct-route",
    r -> r.remoteAddr("10.1.1.1", "10.10.1.1/24")
        .uri("https://downstream1")
.route("proxied-route",
    r -> r.remoteAddr(resolver, "10.10.1.1", "10.10.1.1/24")
        .uri("https://downstream2")
)
```

**自己写一个**

业务需求 - 通过9527网关访问到外网的百度新闻网址

**编码**

cloud-gateway-gateway9527业务实现



```java
@Configuration
public class GateWayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();

        routes.route("path_route_frx01",
                r -> r.path("/guonei")
                    .uri("http://news.baidu.com/guonei")).build();
        return routes.build();
    }
}
```

**测试**

浏览器输入http://localhost:9527/guonei ，返回http://news.baidu.com/guonei相同的页面

## GateWay配置动态路由

默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建**动态路由进行转发，从而实现动态路由的功能**（不写死一个地址）。

**启动**

- eureka7001
- payment8001/8002

**POM**

```xml
<!--eurekalient-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**YML**

需要注意的是uri的协议为`lb`，表示启用Gateway的负载均衡功能。

lb://serviceName是spring cloud gateway在微服务中自动为我们创建的负载均衡uri。











 

 

 





 







 

















```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
#############################新增网关配置###########################
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route     #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service      #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**             # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service      #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**              # 断言，路径相匹配的进行路由
####################################################################

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

测试

启动7001

启动8001-cloud-provider-payment8001,8002-cloud-provider-payment8002

启动9527网关

访问说明

- **测试**

  浏览器输入 - [http://localhost:9527/payment/lb(opens new window)](http://localhost:9527/payment/lb)

  ![QQ22918914922917714320220820204231](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220820/QQ22918914922917714320220820204231.44rrojg3qyi0.gif)

  结果:不停刷新页面，8001/8002两个端口切换。

## GateWay常用的Predicate

**讨论几个Route Predicate Factory**

**The After Route Predicate Factory**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        # 这个时间后才能起效
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

可以通过下述方法获得上述格式的时间戳字符串

```java
import java.time.ZonedDateTime;


public class T2
{
    public static void main(String[] args)
    {
        ZonedDateTime zbj = ZonedDateTime.now(); // 默认时区
        System.out.println(zbj);
       //2022-08-20T21:02:40.570+08:00[Asia/Shanghai]

    }
}
```

**The Between Route Predicate Factory**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        # 两个时间点之间
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[Ame
```

**The Cookie Route Predicate Factory**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: http://localhost:8001
        predicates:
        - Cookie=chocolate, ch.p
```

测试

```sh
# 该命令相当于发get请求，且没带cookie
curl http://localhost:9527/payment/lb

# 带cookie的
curl http://localhost:9527/payment/lb --cookie "chocolate=chip"
```

**The Header Route Predicate Factory**

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: http://localhost:8001
        predicates:
        - Header=X-Request-Id, \d+
```

```sh
# 带指定请求头的参数的CURL命令
curl http://localhost:9527/payment/lb -H "X-Request-Id:123"
```

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220820/image.dix5ffvzpfk.webp)

**小结**

说白了，Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理。

## GateWay的Filter

路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用。Spring Cloud Gateway内置了多种路由过滤器，他们都由GatewayFilter的工厂类来产生。

Spring Cloud Gateway的Filter:

- 生命周期：
  - pre
  - post
- 种类（具体看官方文档）：
  - GatewayFilter - 有31种
  - GlobalFilter - 有10种 常用的GatewayFilter：AddRequestParameter GatewayFilter

自定义全局GlobalFilter：

两个主要接口介绍：

1. GlobalFilter
2. Ordered

能干什么：

1. 全局日志记录
2. 统一网关鉴权
3. …

代码案例：

GateWay9527项目添加MyLogGateWayFilter类：

```java
@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("***********come in MyLogGateWayFilter:  "+new Date());
        String name = exchange.getRequest().getQueryParams().getFirst("uname");
        if(name==null){
            log.info("*******用户名为null，非法用户，o(╥﹏╥)o");
            exchange.getResponse().setRawStatusCode(HttpStatus.HTTP_NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
    
    @Override
    public int getOrder() {
        return 0;
    }
}
```

测试：

启动：

- EurekaMain7001
- PaymentMain8001
- GateWayMain9527
- PaymentMain8002

浏览器输入：

- http://localhost:9527/payment/lb [错误。406]
- http://localhost:9527/payment/lb?uname=abc

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220820/image.1stezk4kaoqo.webp)

# Config 服务配置中心 与 BUS 消息总线

## Config分布式配置中心介绍

**怎么玩**

SpringCloud Config分为**服务端**和**客户端**两部分。

**与GitHub整合配置**

由于SpringCloud Config默认使用Git来存储配置文件(也有其它方式,比如支持SVN和本地文件)，但最推荐的还是Git，而且使用的是http/https访问的形式。

## Config配置总控中心搭建

用你自己的账号在GitHub上新建一个名为springcloud-config的新Repository。

由上一步获得刚新建的git地址 - `git@github.com:xustudyxu/springcloud-config.git`。

本地硬盘目录上新建git仓库并clone。

- 工作目录为 F：\SpringCloud-project
- `git clone git@github.com:xustudyxu/springcloud-config.git`

此时在工作目录会创建名为springcloud-config的文件夹。

在springcloud-config的文件夹种创建三个配置文件（为本次教学使用的）,随后`git add .`，`git commit -m "sth"`等一系列上传操作上传到springcloud-config的新Repository。

- config-dev.yml 

- config-prod.yml

- config-test.yml

```yaml
config:
  info: "master branch,springcloud-config/config-dev.yml version=1"
```

新建Module模块cloud-config-center-3344，它即为Cloud的配置中心模块CloudConfig Center

改pom

YML

```yaml
server:
  port: 3344

spring:
  application:
    name:  cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: git@github.com:xustudyxu/springcloud-config.git #GitHub上面的git仓库名字
        ####搜索目录
          search-paths:
            - springcloud-config
      ####读取分支
      label: master

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

主启动类

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344
{
    public static void main(String[] args) {
            SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```

windows下修改hosts文件，增加映射

```sh
127.0.0.1 config-3344.com
```

测试通过Config微服务是否可以从GitHub上获取配置内容

- 启动7001,ConfigCenterMain3344
- 浏览器防问 - [http://config-3344.com:3344/master/config-dev.yml(opens new window)](http://config-3344.com:3344/master/config-dev.yml)
- 页面返回结果：

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220821/image.20jh8d20lxb4.webp)

配置读取规则

- ```
  /{label(分支)}/{application}-{profile}.yml
  ```

  （推荐）

  - master分支
    - http://config-3344.com:3344/master/config-dev.yml
    - http://config-3344.com:3344/master/config-test.yml
    - http://config-3344.com:3344/master/config-prod.yml
  - dev分支 - http://config-3344.com:3344/dev/config-dev.yml - http://config-3344.com:3344/dev/config-test.yml - http://config-3344.com:3344/dev/config-prod.yml

- `/{application}-{profile}.yml`
  - http://config-3344.com:3344/config-dev.yml
  - http://config-3344.com:3344/config-test.yml
  - http://config-3344.com:3344/config-prod.yml
  - http://config-3344.com:3344/config-xxxx.yml(不存在的配置)
- `/{application}/{profile}[/{label}]`
  - http://config-3344.com:3344/config/dev/master
  - http://config-3344.com:3344/config/test/master
  - http://config-3344.com:3344/config/test/dev

成功实现了用SpringCloud Config通过GitHub获取配置信息

## Config客户端配置与测试

**新建cloud-config-client-3355**

**POM**

bootstrap.yml

applicaiton.yml是用户级的资源配置项

bootstrap.yml是系统级的，优先级更加高

Spring Cloud会创建一个Bootstrap Context，作为Spring应用的Application Context的父上下文。

初始化的时候，**BootstrapContext负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的Environment。**

Bootstrap属性有高优先级，默认情况下，它们不会被本地配置覆盖。Bootstrap context和Application Context有着不同的约定，所以新增了一个bootstrap.yml文件，保证Bootstrap Context和Application Context配置的分离。

要将Client模块下的application.yml文件改为bootstrap.yml,这是很关键的，因为bootstrap.yml是比application.yml先加载的。`bootstrap.yml`优先级高于`application.yml`。

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://localhost:3344/ #配置中心地址k


#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

**修改config-dev.yml配置并提交到GitHub中，比如加个变量age或者版本号version**

主启动

```java
@EnableEurekaClient
@SpringBootApplication
public class ConfigClientMain3355 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3355.class,args);
    }
}
```

业务类

```java
@RestController
@Slf4j
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

测试

- 启动Config配置中心3344微服务并自测
  - http://config-3344.com:3344/master/config-prod.yml
  - http://config-3344.com:3344/master/config-dev.yml
- 启动3355作为Client准备访问
  - [http://localhost:3355/configlnfo(opens new window)](http://localhost:3355/configlnfo)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220821/image.2hh9l2apgja0.webp)

**成功实现了客户端3355访问SpringCloud Config3344通过GitHub获取配置信息可题随时而来**

**分布式配置的动态刷新问题**

- Linux运维修改GitHub上的配置文件内容做调整
- 刷新3344，发现ConfigServer配置中心立刻响应

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220821/image.33cvo9016bm0.webp)

- 刷新3355，发现ConfigClient客户端没有任何响应

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220821/image.2hh9l2apgja0.webp)

- 3355没有变化除非自己重启或者重新加载
- 难到每次运维修改配置文件，客户端都需要重启??噩梦

## Config动态刷新之手动版

避免每次更新配置都要重启客户端微服务3355

**动态刷新步骤**：

修改3355模块

POM引入actuator监控

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

修改YML，添加暴露监控端口配置：

```yaml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

`@RefreshScope`业务类Controller修改

```java
@RestController
@RefreshScope//<----添加该注解
@Slf4j
public class ConfigClientController {

}
```

测试

此时修改github配置文件内容 -> 访问3344 -> 访问3355

- http://config-3344.com:3344/master/config-dev.yml

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220821/image.6ckulqy33mg0.webp)

- [http://localhost:3355/configInfo(opens new window)](http://localhost:3355/configInfo)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220821/image.69kqiokyb900.webp)

3355改变没有??? **没有**，还需一步

How

需要运维人员发送Post请求刷新3355

```sh
curl -X POST "http://localhost:3355/actuator/refresh"
```

1

再次测试

[http://localhost:3355/configInfo(opens new window)](http://localhost:3355/configInfo)

3355改变没有??? **改了**。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220821/image.6u7coe9nn6c0.webp)

成功实现了客户端3355刷新到最新配置内容，避免了服务重启

想想还有什么问题?

- 假如有多个微服务客户端3355/3366/3377
- 每个微服务都要执行—次post请求，手动刷新?
- **可否广播，一次通知，处处生效?**
- 我们想大范围的自动刷新，求方法

## Bus消息总线是什么

**上—讲解的加深和扩充**

一言以蔽之，分布式自动刷新配置功能。

**是什么**

Spring Cloud Bus 配合Spring Cloud Config 使用可以实现配置的动态刷新。

Spring Cloud Bus是用来将分布式系统的节点与轻量级消息系统链接起来的框架，它整合了Java的事件处理机制和消息中间件的功能。Spring Clud Bus目前支持`RabbitMQ`和`Kafka`。

**能干嘛**

Spring Cloud Bus能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道。

**为何被称为总线**

什么是总线

在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。

基本原理

ConfigClient实例都监听MQ中同一个topic(默认是Spring Cloud Bus)。当一个服务刷新数据的时候，它会把这个信息放入到Topic中，这样其它监听同一Topic的服务就能得到通知，然后去更新自身的配置。

## Bus之RabbitMQ环境配置

- 安装Erlang，

- 安装RabbitMQ
- 打开cmd进入RabbitMQ安装目录下的sbin目录，如：D:\DevelopTools\RabbitMQ\install\rabbitmq_server-3.8.3\sbin
- 输入以下命令启动管理功能

```sh
rabbitmq-plugins enable rabbitmq_management
```

这样就可以添加可视化插件。

- 访问地址查看是否安装成功：http://localhost:15672/
- 输入账号密码并登录：guest guest

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.2mc1fnqmr1i0.webp)

## Bus动态刷新全局广播的设计思想和选型

须先具备良好的RabbitMQ环境先

演示广播效果，增加复杂度，再以3355为模板再制作一个3366

1. 新建cloud-config-client-3366
2. POM

3. YML

```yaml
server:
  port: 3366

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://localhost:3344 #配置中心地址

#rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

4. 主启动

```java
@SpringBootApplication
@EnableEurekaClient
public class ConfigClientMain3366 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3366.class,args);
    }
}
```

5. 业务类

```java
@RestController
@RefreshScope
public class ConfigClientController {

    @Value("${server.port}")
    private String serverPort;

    @Value("${config.info}")
    private String configInfo;

    @GetMapping(value = "/configInfo")
    public String getConfigInfo(){
        return "serverPort:"+serverPort+"\t\n\n configInfo:"+configInfo;
    }
}
```

**设计思想**

1. 利用消息总线触发一个客户端/bus/refresh,而刷新所有客户端的配置

2. 利用消息总线触发一个服务端ConfigServer的/bus/refresh端点，而刷新所有客户端的配置

图二的架构显然更加适合，图—不适合的原因如下：

- 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新的职责。
- 破坏了微服务各节点的对等性。
- 有一定的局限性。例如，微服务在迁移时，它的网络地址常常会发生变化，此时如果想要做到自动刷新，那就会增加更多的修改。

## Bus动态刷新全局广播配置实现

**给cloud-config-center-3344配置中心服务端添加消息总线支持**

POM

注意：AMQP依赖版本不能太高

```xml
<!--添加消息总线RabbitNQ支持-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-amap</artifactId>
</dependency>
<dependency>
	<groupId>org-springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

application.yml

```yaml
server:
  port: 3344

spring:
  application:
    name:  cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: git@github.com:xutudyxu/springcloud-config.git #GitHub上面的git仓库名字
          ####搜索目录
          search-paths:
            - springcloud-config
      ####读取分支
      label: master
#rabbitmq相关配置<--------------------------
rabbitmq:
  host: localhost
  port: 5672
  username: guest
  password: guest

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

##rabbitmq相关配置,暴露bus刷新配置的端点<--------------------------
management:
  endpoints: #暴露bus刷新配置的端点
    web:
      exposure:
        include: 'bus-refresh'
```

**给cloud-config-client-3355客户端添加消息总线支持**

POM

bootstrap.yml

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://localhost:3344 #配置中心地址k

#rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口<----------------------
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

**给cloud-config-client-3366客户端添加消息总线支持**

POM

bootstrap.yml

```yaml
server:
  port: 3366

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://localhost:3344 #配置中心地址

#rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口<-----------------------
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

**测试**

- 启动
  - EurekaMain7001
  - ConfigcenterMain3344
  - ConfigclientMain3355
  - ConfigclicntMain3366
- 运维工程师
  - 修改Github上配置文件内容，增加版本号为 4
  - 发送POST请求
    - `curl -X POST "http://localhost:3344/actuator/bus-refresh"`
    - **—次发送，处处生效**(触发服务端3344，刷新客户端)
- 配置中心
  - [http://config-3344.com:3344/config-dev.yml(opens new window)](http://config-3344.com:3344/config-dev.yml)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.23kdoaxiyf6o.webp)

- 客户端
- [http://localhost:3355/configlnfo(opens new window)](http://localhost:3355/configlnfo)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.6hprk3uwzws0.webp)

- [http://localhost:3366/configInfo(opens new window)](http://localhost:3366/configlnfo)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.moh9x7y9hts.webp)

- 获取配置信息，发现都已经刷新了

**—次修改，广播通知，处处生效**

## Bus动态刷新定点通知

不想全部通知，只想定点通知

- 只通知3355
- 不通知3366

简单一句话 - 指定具体某一个实例生效而不是全部

- 公式：http://localhost:3344/actuator/bus-refresh/{destination}
- /bus/refresh请求不再发送到具体的服务实例上，而是发给config server通过destination参数类指定需要更新配置的服务或实例

案例

- 我们这里以刷新运行在3355端口上的config-client（配置文件中设定的应用名称）为例，只通知3355，不通知3366
- curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"
- 将版本改为5

配置中心

- [http://config-3344.com:3344/config-dev.yml(opens new window)](http://config-3344.com:3344/config-dev.yml)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.65syde1wxr40.webp)

- 客户端
- [http://localhost:3355/configlnfo(opens new window)](http://localhost:3355/configlnfo)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.1rfw022lykkg.webp)

- [http://localhost:3366/configlnfo(opens new window)](http://localhost:3366/configlnfo)

![1661102449228](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/1661102449228.190io4tjo7y.webp)

通知总结

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.4drpraucccq0.webp)



# Stream 消息驱动

## Stream为什么被引入

常见MQ(消息中间件)：

- ActiveMQ
- RabbitMQ
- RocketMQ
- Kafka

有没有一种新的技术诞生，让我们不再关注具体MQ的细节，我们只需要用一种适配绑定的方式，自动的给我们在各种MQ内切换。（类似于Hibernate）

## Stream是什么及Binder介绍

官方定义Spring Cloud Stream是一个构建消息驱动微服务的框架。

应用程序通过inputs或者 outputs 来与Spring Cloud Stream中binder对象交互

## Stream的设计思想

**为什么用Cloud Stream？**

比方说我们用到了RabbitMQ和Kafka，由于这两个消息中间件的架构上的不同，像RabbitMQ有exchange，kafka有Topic和Partitions分区。

**通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离。**

**Binder：**

- INPUT对应于消费者
- OUTPUT对应于生产者

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.be4a4zivseo.webp)

**Stream中的消息通信方式遵循了发布-订阅模式**

Topic主题进行广播

- 在RabbitMQ就是Exchange
- 在Kakfa中就是Topic

## Stream编码常用注解简介

**Spring Cloud Stream标准流程套路**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.20rnfxuan4n4.webp)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.19ouvdp6wnog.webp)

- Binder - 很方便的连接中间件，屏蔽差异。
- Channel - 通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置。
- Source和Sink - 简单的可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入。

编码API和常用注解

| 组成            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Middleware      | 中间件，目前只支持RabbitMQ和Kafka                            |
| Binder          | Binder是应用与消息中间件之间的封装，目前实行了Kafka和RabbitMQ的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息类型(对应于Kafka的topic,RabbitMQ的exchange)，这些都可以通过配置文件来实现 |
| @Input          | 注解标识输入通道，通过该输乎通道接收到的消息进入应用程序     |
| @Output         | 注解标识输出通道，发布的消息将通过该通道离开应用程序         |
| @StreamListener | 监听队列，用于消费者的队列的消息接收                         |
| @EnableBinding  | 指信道channel和exchange绑定在一起                            |

案例说明

准备RabbitMQ环境（[Bus之RabbitMQ环境配置](https://frxcat.fun/Spring/SpringCloud/Config_And_BUS/#bus之rabbitmq环境配置)有提及）

工程中新建三个子模块

- cloud-stream-rabbitmq-provider8801，作为生产者进行发消息模块
- cloud-stream-rabbitmq-consumer8802，作为消息接收模块
- cloud-stream-rabbitmq-consumer8803，作为消息接收模块

## Stream消息驱动之生产者

新建Module：cloud-stream-rabbitmq-provider8801

POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-stream-rabbitmq-provider8801</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <!--基础配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

YML

```yaml
server:
  port: 8801

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
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          output: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

主启动类StreamMQMain8801

```java
@SpringBootApplication
public class StreamMQMain8801 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8801.class,args);
    }
}
```

业务类

1. 发送消息接口

```java
public interface IMessageProvider {
    public String send();
}
```

1. 发送消息接口实现类

```java
@EnableBinding(Source.class)//定义消息发送管道
public class MessageProviderImpl implements IMessageProvider {

    @Resource
    private MessageChannel output;//消息发送管道

    @Override
    public String send() {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        System.out.println("-------------serial:"+serial);
        return null;
    }
}
```

1. Controller

```java
@RestController
public class SendMessageController {

    @Resource
    private IMessageProvider messageProvider;

    @GetMapping(value = "/sendMessage")
    public String sendMessage(){
        return messageProvider.send();
    }
}
```

测试

- 启动 7001eureka
- 启动 RabpitMq（Bus之RabbitMQ环境配置）
  - `rabbitmq-plugins enable rabbitmq_management`
  - [http://localhost:15672/(opens new window)](http://localhost:15672/)
- 启动 8801
- 访问 - http://localhost:8801/sendMessage
  - 后台将打印serial: UUID字符串

```java
-------------serial:5e1053f8-1b8f-4f0b-ad04-82d1184281f6
```

## Stream消息驱动之消费者

新建Module：cloud-stream-rabbitmq-consumer8802

POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-stream-rabbitmq-consumer8802</artifactId>
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
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--基础配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

YML

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          input: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

主启动类StreamMQMain8802

```java
@SpringBootApplication
public class StreamMQMain8802 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8802.class,args);
    }
}
```

业务类

Controller

```java
@Component
@EnableBinding(Sink.class)
public class ReceiveMessageListenerController {

    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message){
        System.out.println("消费者1号------------------>接收到的消息:"+message.getPayload()+"\t port:"+serverPort);
    }
}
```

测试

- 启动EurekaMain7001
- 启动StreamMQMain8801
- 启动StreamMQMain8802
- 8801发送8802接收消息

```java
消费者1号------------------>接收到的消息:ed7ceea5-075a-46c2-afa3-16d172266d83	 port:8802
消费者1号------------------>接收到的消息:4cfe1056-ed5d-4f4e-9f55-9861332a3d01	 port:8802
消费者1号------------------>接收到的消息:da627e6b-26e0-41a4-95d4-b2dee62c6236	 port:8802
消费者1号------------------>接收到的消息:5c21478f-4caf-42bf-825c-df48977331d5	 port:8802
消费者1号------------------>接收到的消息:9b583b78-da1c-432e-9acf-030c77c0b95c	 port:8802
消费者1号------------------>接收到的消息:2fd30d34-eec9-48a7-ba8e-9d8ac6461102	 port:8802
消费者1号------------------>接收到的消息:37e6176d-3060-486f-b54c-c07743c73b17	 port:8802
消费者1号------------------>接收到的消息:38de987c-7c56-4ba7-9863-92f071e981d0	 port:8802
消费者1号------------------>接收到的消息:cd14ee77-6a95-43c4-9fc0-f82fb876f848	 port:8802
```

## Stream之消息重复消费

依照8802，克隆出来一份运行8803 - cloud-stream-rabbitmq-consumer8803。

**启动**

- RabbitMQ
- 服务注册 - 8801
- 消息生产 - 8801
- 消息消费 - 8802
- 消息消费 - 8802

运行后有两个问题

8802:

```java
消费者1号------------------>接收到的消息:3c30f4e3-2f94-48f1-8e75-3a9f0f1107a7	 port:8802
消费者1号------------------>接收到的消息:a365103a-f116-4596-8215-e78912cb1d66	 port:8802
```

8803:

```java
消费者2号,----->接受到的消息: 3c30f4e3-2f94-48f1-8e75-3a9f0f1107a7	  port: 8803
消费者2号,----->接受到的消息: a365103a-f116-4596-8215-e78912cb1d66	  port: 8803
```

1. 有重复消费问题
2. 消息持久化问题

**消费**

- [http://localhost:8801/sendMessage(opens new window)](http://localhost:8801/sendMessage)
- 目前是8802/8803同时都收到了，存在重复消费问题
- 如何解决：分组和持久化属性group（重要）

**生产实际案例**

比如在如下场景中，订单系统我们做集群部署，都会从RabbitMQ中获取订单信息，那如果一个订单同时被两个服务获取到，那么就会造成数据错误，我们得避免这种情况。这时我们就可以**使用Stream中的消息分组来解决**。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.1j9z25exmu80.webp)

注意在Stream中处于同一个group中的多个消费者是竞争关系，就能够保证消息只会被其中一个应用消费一次。不同组是可以全面消费的(重复消费)。

## Stream之group解决消息重复消费

原理

微服务应用放置于同一个group中，就能够保证消息只会被其中一个应用消费一次。

**不同的组**是可以重复消费的，**同一个组**内会发生**竞争关系**，只有其中一个可以消费。

**8802/8803都变成不同组，group两个不同**

group: A_Group、B_Group

8802修改YML

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          group: A_Group #<----------------------------------------关键
eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

8803修改YML（与8802的类似位置 group: B_Group）

结论：**还是重复消费**

8802/8803实现了轮询分组，每次只有一个消费者，8801模块的发的消息只能被8802或8803其中一个接收到，这样避免了重复消费。

group: A_Group

8802修改YML`group: A_Group`

8803修改YML`group: A_Group`

8002:

```java
消费者1号------------------>接收到的消息:6ceb411d-370f-4ef8-9894-6b77a663e869	 port:8802
消费者1号------------------>接收到的消息:8d9fe804-c93e-4c20-954f-00c23f2b8fa0	 port:8802
消费者1号------------------>接收到的消息:14b3d70d-3564-4035-878d-cdd0460c0f2f	 port:8802
消费者1号------------------>接收到的消息:510a6159-be63-4dbf-a2ce-7827c76b395f	 port:8802
```

8003:

```java
消费者2号,----->接受到的消息: 9a599132-9bc2-4d5e-921b-670db688855f	  port: 8803
消费者2号,----->接受到的消息: 3015e820-341f-4956-80b7-a813413fb1f5	  port: 8803
消费者2号,----->接受到的消息: eeb9b458-f68e-4911-b132-57cdb4795cd4	  port: 8803
```

结论：同一个组的多个微服务实例，每次只会有一个拿到

## Stream之消息持久化

通过上述，解决了重复消费问题，再看看持久化。

停止8802/8803并**去除掉**8802的分组`group: A_Group`，8803的分组`group: A_Group`没有去掉。

8801先发送4条消息到RabbitMq。

先启动8802，**无分组属性配置**，后台没有打出来消息。

再启动8803，**有分组属性配置**，后台打出来了MQ上的消息。(消息持久化体现)

```java
消费者2号,----->接受到的消息: 402b4095-4911-41e1-bf92-f146d8f0efa9	  port: 8803
消费者2号,----->接受到的消息: b24a7a0e-2504-4248-92ff-e95777617a4c	  port: 8803
消费者2号,----->接受到的消息: 1b821e91-9798-461d-ba6e-7a0260cc0cfa	  port: 8803
消费者2号,----->接受到的消息: 6d7e1145-bfab-494b-a9c5-8dc71ab6751e	  port: 8803
```

# Sleuth 分布式请求链路追踪

## Sleuth是什么

**为什么会出现这个技术？要解决哪些问题？**

在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.5re5bq8z1fc0.webp)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.6ta641owfto0.webp)

**是什么**

- https://github.com/spring-cloud/spring-cloud-sleuth
- Spring Cloud Sleuth提供了一套完整的服务跟踪的解决方案
- 在分布式系统中提供追踪解决方案并且兼容支持了zipkin

**解决**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.7afjeo1qgkg0.webp)

产品调库存，发送链路数据，谁调谁，zipkin就记录下来，以图形或网页的形式展现。

## Sleuth之zipkin搭建安装

1. zipkin

下载

- SpringCloud从F版起已不需要自己构建Zipkin Server了，只需调用jar包即可
- https://repo1.maven.org/maven2/io/zipkin/zipkin-server/
- zipkin-server-2.12.9-exec.jar

运行jar



```sh
java -jar zipkin-server-2.12.9-exec.jar
```

1

**运行控制台**

http://localhost:9411/zipkin/

**术语**

完整的调用链路

表示一请求链路，一条链路通过Trace ld唯一标识，Span标识发起的请求信息，各span通过parent id关联起来

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.6l5xfhx822k0.webp)

—条链路通过Trace ld唯一标识，Span标识发起的请求信息，各span通过parent id关联起来。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.122mvcbvbqsg.webp)

整个链路的依赖关系如下：

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.4sehxbkyiju0.webp)

名词解释

- Trace：类似于树结构的Span集合，表示一条调用链路，存在唯一标识
- span：表示调用链路来源，通俗的理解span就是一次请求信息

## Sleuth链路监控展现

1. 服务提供者

cloud-provider-payment8001

POM

```xml
<!--包含了sleuth+zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

YML

```yaml
spring:
  application:
    name: cloud-payment-service

  zipkin: #<-------------------------------------关键 
      base-url: http://localhost:9411
  sleuth: #<-------------------------------------关键
    sampler:
    #采样率值介于 0 到 1 之间，1 则表示全部采集
    probability: 1
    
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: hsp
```

业务类PaymentController

```java
@RestController
@Slf4j
public class PaymentController {
    
    ...
    
 	@GetMapping("/payment/zipkin")
    public String paymentZipkin() {
        return "hi ,i'am paymentzipkin server fall back，welcome to here, O(∩_∩)O哈哈~";
    }    
}
```

1. 服务消费者(调用方)

cloue-consumer-order80

POM

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

YML

```yaml
spring:
    application:
        name: cloud-order-service
    zipkin:
      base-url: http://localhost:9411
    sleuth:
      sampler:
        probability: 1
```

业务类

```java
    // ====================> zipkin+sleuth
    @GetMapping("/consumer/payment/zipkin")
    public String paymentZipkin()
    {
        String result = restTemplate.getForObject("http://localhost:8001"+"/payment/zipkin/", String.class);
        return result;
    }
}
```

1. 依次启动eureka7001/8001/80 - 80调用8001几次测试下
   - 发送请求[http://localhost:81/consumer/payment/zipkin(opens new window)](http://localhost:81/consumer/payment/zipkin)
2. 打开浏览器访问: [http://localhost:9411(opens new window)](http://localhost:9411/)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.7dtxh37vn600.webp)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220822/image.72ph5v5byno0.webp)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220823/image.2cfayzjw29hc.webp)

# SpringCloud Alibaba 简介

## 为什么会出现SpringCloud alibaba

Spring Cloud Netflix项目进入维护模式

Netflix:Eureka,ribbon,feign,zuul,config

https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now

什么是维护模式？

将模块置于维护模式，意味着Spring Cloud团队将不会再向模块添加新功能。

他们将修复block级别的 bug 以及安全问题，他们也会考虑并审查社区的小型pull request。

## SpringCloud Alibaba带来了什么

**是什么**

[官网(opens new window)](https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md)

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。

诞生：2018.10.31，Spring Cloud Alibaba 正式入驻了Spring Cloud官方孵化器，并在Maven 中央库发布了第一个版本。

**能干嘛**

- **服务限流降级**：默认支持 WebServlet、WebFlux, OpenFeign、RestTemplate、Spring Cloud Gateway, Zuul, Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- **服务注册与发现**：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。
- **消息驱动能力**：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- **分布式事务**：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。
- **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
- **阿里云短信服务**：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

**去哪下**

如果需要使用已发布的版本，在 `dependencyManagement` 中添加如下配置。

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.5.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

然后在 `dependencies` 中添加自己所需使用的依赖即可使用。

**怎么玩**

- [Sentinel (opens new window)](https://github.com/alibaba/Sentinel):把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- [Nacos (opens new window)](https://github.com/alibaba/Nacos):一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- [RocketMQ (opens new window)](https://rocketmq.apache.org/):一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- [Dubbo (opens new window)](https://github.com/apache/dubbo):Apache Dubbo™ 是一款高性能 Java RPC 框架。
- [Seata (opens new window)](https://github.com/seata/seata):阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。
- [Alibaba Cloud OSS (opens new window)](https://www.aliyun.com/product/oss):阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- [Alibaba Cloud SchedulerX (opens new window)](https://help.aliyun.com/document_detail/148185.html):阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。
- [Alibaba Cloud SMS (opens new window)](https://www.aliyun.com/product/sms):覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

## Spring Cloud Alibaba学习资料获取

- 官网
  - [https://spring.io/projects/spring-cloud-alibaba#overview(opens new window)](https://spring.io/projects/spring-cloud-alibaba#overview)
- 英文
  - [https://github.com/alibaba/spring-cloud-alibaba(opens new window)](https://github.com/alibaba/spring-cloud-alibaba)
  - [https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html(opens new window)](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html)
- 中文
  - https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md

# Nacos 服务发现、配置管理和服务管理平台

## Nacos简介和下载

**为什么叫Nacos**

- 前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service。

**是什么**

- 一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- Nacos: Dynamic Naming and Configuration Service
- Nacos就是注册中心＋配置中心的组合 -> Nacos = **Eureka+Config+Bus**

**能干嘛**

- 替代Eureka做服务注册中心
- 替代Config做服务配置中心

去哪下

- [https://github.com/alibaba/nacos/releases(opens new window)](https://github.com/alibaba/nacos/releases)
- [官网文档](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring cloud alibaba nacos_discovery)

**各中注册中心比较**

| 服务注册与发现框架 | CAP模型 | 控制台管理 | 社区活跃度      |
| ------------------ | ------- | ---------- | --------------- |
| Eureka             | AP      | 支持       | 低(2.x版本闭源) |
| Zookeeper          | CP      | 不支持     | 中              |
| consul             | CP      | 支持       | 高              |
| Nacos              | AP      | 支持       | 高              |

据说Nacos在阿里巴巴内部有超过10万的实例运行，已经过了类似双十一等各种大型流量的考验。

## Nacos安装

- 本地Java8+Maven环境已经OK先

- 从[官网 (opens new window)](https://github.com/alibaba/nacos/releases)下载Nacos

- 解压安装包，直接运行bin目录下的startup.cmd

  ```sh
  startup.cmd -m standalone
  ```

- 命令运行成功后直接访问[http://localhost:8848/nacos (opens new window)](http://localhost:8848/nacos)，默认账号密码都是nacos

- 结果页面

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.4i5gdhcivla0.webp)

## Nacos之服务提供者注册

[官方文档(opens new window)](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery)

新建Module - cloudalibaba-provider-payment9001

POM

父POM

```xml
<dependencyManagement>
    <dependencies>
        <!--spring cloud alibaba 2.1.0.RELEASE-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.1.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

本模块POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-provider-payment9001</artifactId>
    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>

```

YML

```yaml
server:
  port: 9001

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
        include: '*'
```

主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9001.class,args);
    }
}
```

Controller

```java
@RestController
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id){
        return "Nacos registry, serverPort: "+serverPort+"\t id:"+id;
    }
}
```

测试

- [http://localhost:9001/payment/nacos/1(opens new window)](http://localhost:9001/payment/nacos/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.66o61k206k00.webp)

- nacos控制台

![1661334507137](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220826/1661334507137.5qur0ovy3600.webp)

- nacos服务注册中心+服务提供者9001都OK了

为了下一章节演示nacos的负载均衡，参照9001新建9002

- 新建cloudalibaba-provider-payment9002
- 9002其它步骤你懂的
- 或者**取巧**不想新建重复体力劳动，可以利用IDEA功能，直接拷贝虚拟端口映射

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.3mzundrnfq00.webp)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.o3dvyjfk15c.webp)

- [http://localhost:9011/payment/nacos/1(opens new window)](http://localhost:9011/payment/nacos/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.67mts1qd01w0.webp)

- nacos控制台

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.3w15p0upay0.webp)

- 新建cloudalibaba-provider-payment9002

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.15ljzsnlk480.webp)

## Nacos之服务消费者注册和负载

新建Module - cloudalibaba-consumer-nacos-order83

POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-consumer-nacos-order83</artifactId>
    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.frx01.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

为什么nacos支持负载均衡？因为spring-cloud-starter-alibaba-nacos-discovery内含netflix-ribbon包。

YML

```yaml
server:
  port: 83

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848

#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider
```

主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderNacosMain83 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain83.class,args);
    }
}
```

业务类

ApplicationContextConfig

```java
@Configuration
public class ApplicationConfig {
    
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

OrderNacosController

```java
@RestController
public class OrderNacosController {

    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverURL;

    @GetMapping("/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Integer id){
        return restTemplate.getForObject(serverURL+"/payment/nacos/"+id,String.class);
    }
}
```

测试

- 启动nacos控制台
- [http://localhost:83/consumer/payment/nacos/1(opens new window)](http://localhost:83/consumer/payment/nacos/1)

![QQ22918914922917714320220824204517](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/QQ22918914922917714320220824204517.158d055ketts.gif)

- 83访问9001/9002，轮询负载OK

## Nacos服务注册中心对比提升

**Nacos全景图**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.4hatbbv9zz4.webp)

**Nacos和CAP**

Nacos与其他注册中心特性对比

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.6zg3vithqws0.webp)

**Nacos服务发现实例模型**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/image.2pleq09bklk0.webp)

**Nacos支持AP和CP模式的切换**

C是所有节点在同一时间看到的数据是一致的;而A的定义是所有的请求都会收到响应。

何时选择使用何种模式?

—般来说，如果不需要存储服务级别的信息且服务实例是通过nacos-client注册，并能够保持心跳上报，那么就可以选择AP模式。当前主流的服务如Spring cloud和Dubbo服务，都适用于AP模式，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例。

如果需要在服务级别编辑或者存储配置信息，那么CP是必须，K8S服务和DNS服务则适用于CP模式。CP模式下则支持注册持久化实例，此时则是以Raft协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。

切换命令：

```
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP
```

## Nacos之服务配置中心

基础配置

cloudalibaba-config-nacos-client3377

POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-config-nacos-client3377</artifactId>
    <dependencies>
        <!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--nacos-discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般基础配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

YML

Nacos同springcloud-config一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动。

springboot中配置文件的加载是存在优先级顺序的，bootstrap优先级高于application

bootstrap

```yaml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置


# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# nacos-config-client-dev.yaml

# nacos-config-client-test.yaml   ----> config.info
```

application

```yaml
spring:
  profiles:
    active: dev # 表示开发环境
    #active: test # 表示测试环境
    #active: info
```

主启动

```java
@EnableDiscoveryClient
@SpringBootApplication
public class NacosConfigClientMain3377
{
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientMain3377.class, args);
    }
}
```

Controller

```java
@RestController
@RefreshScope   //支持Nacos的动态刷新功能
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

**在Nacos中添加配置信息**

Nacos中的dataid的组成格式及与SpringBoot配置文件中的匹配规则

[官方文档(opens new window)](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)

说明：之所以需要配置spring.application.name，是因为它是构成Nacos配置管理dataId 字段的一部分。

在 Nacos Spring Cloud中,dataId的完整格式如下：



```yaml
${prefix}-${spring-profile.active}.${file-extension}
```

1

- `prefix`默认为`spring.application.name`的值，也可以通过配置项`spring.cloud.nacos.config.prefix`来配置。
- `spring.profile.active`即为当前环境对应的 `profile`，详情可以参考 Spring Boot文档。注意：当`spring.profile.active`为空时，对应的连接符 - 也将不存在，`datald` 的拼接格式变成{file-extension}
- file-exetension为配置内容的数据格式，可以通过配置项`spring .cloud.nacos.config.file-extension`来配置。目前只支持`properties`和`yaml`类型。
- 通过Spring Cloud 原生注解`@RefreshScope`实现配置自动更新。

最后公式：



```java
${spring.application.name)}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
```

1

配置新增

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.4ssyzwcxh5o0.webp)

Nacos界面配置对应 - 设置DataId

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.5im2tpxeos00.webp)



```yaml
config:
    info: config info for dev,from nacos config center,version=1
```

配置小结

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.6qdb03c1it40.webp)

- 测试

- 启动前需要在nacos客户端-配置管理-配置管理栏目下有对应的yaml配置文件
- 运行cloud-config-nacos-client3377的主启动类
- 调用接口查看配置信息 - [http://localhost:3377/config/info(opens new window)](http://localhost:3377/config/info)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.3dzpheksvm20.webp)

**自带动态刷新**

修改下Nacos中的yaml配置文件，再次调用查看配置的接口，就会发现配置已经刷新。

- 将version改为2

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.79gz7serx840.webp)

## Nacos之命名空间分组和DataID三者关系

**问题 - 多环境多项目管理**

问题1:

实际开发中，通常一个系统会准备

1. dev开发环境
2. test测试环境
3. prod生产环境。

如何保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件呢?

问题2:

一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境…那怎么对这些微服务配置进行管理呢?

Nacos的图形化管理界面

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.76s386tg3800.webp)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.o6mv2ha8yls.webp)

**Namespace+Group+Data lD三者关系？为什么这么设计？**

1. 是什么

类似Java里面的package名和类名最外层的namespace是可以用于区分部署环境的，Group和DatalD逻辑上区分两个目标对象。

1. 三者情况

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.6bvqpd9va3s0.webp)

默认情况：Namespace=public，Group=DEFAULT_GROUP，默认Cluster是DEFAULT

- Nacos默认的Namespace是public，Namespace主要用来实现隔离。
  - 比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个Namespace，不同的Namespace之间是隔离的。
- Group默认是DEFAULT_GROUP，Group可以把不同的微服务划分到同一个分组里面去
- Service就是微服务:一个Service可以包含多个Cluster (集群)，Nacos默认Cluster是DEFAULT，Cluster是对指定微服务的一个虚拟划分。
  - 比方说为了容灾，将Service微服务分别部署在了杭州机房和广州机房，这时就可以给杭州机房的Service微服务起一个集群名称(HZ) ，给广州机房的Service微服务起一个集群名称(GZ)，还可以尽量让同一个机房的微服务互相调用，以提升性能。
- 最后是Instance，就是微服务的实例。

## Nacos之DataID配置

指定spring.profile.active和配置文件的DatalD来使不同环境下读取不同的配置

默认空间+默认分组+新建dev和test两个DatalD

- 新建dev配置DatalD

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.40mozznilxa0.webp)

- 新建test配置DatalD

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.4cy2sw2mh9q0.webp)

通过spring.profile.active属性就能进行多环境下配置文件的读取

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.1xrtzpe911z4.webp)

**测试**

- [http://localhost:3377/config/info(opens new window)](http://localhost:3377/config/info)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.4rda0znq9sw0.webp)

- 配置是什么就加载什么 test/dev

## Nacos之Group分组方案

通过Group实现环境区分 - 新建Group

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.3av1ooy8gmy.webp)

在nacos图形界面控制台上面新建配置文件DatalD

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.4wgtctg0c0m0.webp)

bootstrap+application

在config下增加一条group的配置即可。可配置为DEV_GROUP或TEST GROUP

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.66fekbqg4k00.webp)

## Nacos之Namespace空间方案

新建dev/test的Namespace

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.3o8cut4xnvq0.webp)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.iif5a0jkjpc.webp)

回到服务管理-服务列表查看

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.7r2a3ibwem4.webp)

按照域名配置填写

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.13e3f1v7rvao.webp)

YML

```yaml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP
        namespace: 5ca47874-779f-4dfc-a0b6-f70b21108638  #<----指定namespace

# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# nacos-config-client-dev.yaml

# nacos-config-client-test.yaml   ----> config.info
```

- 测试访问[http://localhost:3377/config/info(opens new window)](http://localhost:3377/config/info)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.2rvq3f9rwqs0.webp)

## Nacos集群_架构说明

> [官方文档(opens new window)](https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html)
>
> 官网架构图
>
> 集群部署架构图
>
> 因此开源的时候推荐用户把所有服务列表放到一个vip下面，然后挂到一个域名下面
>
> http://ip1:port/openAPI直连ip模式，机器挂则需要修改ip才可以使用。
>
> http://VIP:port/openAPI挂载VIP模式，直连vip即可，下面挂server真实ip，可读性不好。
>
> http://nacos.com:port/openAPI域名＋VIP模式，可读性好，而且换ip方便，推荐模式
>
> ![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.79l43dr3wp80.webp)

上图官网翻译，真实情况

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.16zo8yeotl5s.webp)

> 官网说明
>
> 默认Nacos使用嵌入式数据库实现数据的存储。所以，如果启动多个默认配置下的Nacos节点，数据存储是存在一致性问题的。为了解决这个问题，**Nacos采用了集中式存储的方式来支持集群化部署，目前只支持MySQL的存储。**
>
> Nacos支持三种部署模式
>
> - 单机模式-用于测试和单机试用。
> - 集群模式-用于生产环境，确保高可用。
> - 多集群模式-用于多数据中心场景。
>
> **Windows**
>
> cmd startup.cmd或者双击startup.cmd文件
>
> **单机模式支持mysql**
>
> 在0.7版本之前，在单机模式时nacos使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。0.7版本增加了支持mysql数据源能力，具体的操作步骤:
>
> 1. 安装数据库，版本要求:5.6.5+
> 2. 初始化mysq数据库，数据库初始化文件: nacos-mysql.sql
> 3. 修改conf/application.properties文件，增加支持mysql数据源配置（目前只支持mysql)，添加mysql数据源的url、用户名和密码。
>
> 
>
> ```properties
> spring.datasource.platform=mysql
> 
> db.num=1
> db.url.0=jdbc:mysql://11.162.196.16:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
> db.user=nacos_devtest
> db.password=youdontknow
> ```
>
> 再以单机模式启动nacos，nacos所有写嵌入式数据库的数据都写到了mysql。

## Nacos持久化切换配置

Nacos默认自带的是嵌入式数据库derby，[nacos的pom.xml (opens new window)](https://github.com/alibaba/nacos/blob/develop/pom.xml)中可以看出。

derby到mysql切换配置步骤：

1. nacos-server-1.1.4\nacos\conf录下找到nacos-mysql.sql文件，执行脚本。
2. nacos-server-1.1.4\nacos\conf目录下找到application.properties，添加以下配置（按需修改对应值）。

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=hsp
```

启动Nacos，可以看到是个全新的空记录界面，以前是记录进derby。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.1pbry18pjosg.webp)

- 添加配置，查询MySQL数据库

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.1codosk59qxs.webp)

## Nacos之Linux版本安装

预计需要，1个Nginx+3个[nacos (opens new window)](https://so.csdn.net/so/search?q=nacos&spm=1001.2101.3001.7020)注册中心+1个mysql

> 请确保是在环境中安装使用:
>
> 1. 64 bit OS Linux/Unix/Mac，推荐使用Linux系统。
> 2. 64 bit JDK 1.8+；[下载 (opens new window)](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).[配置 (opens new window)](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)。
> 3. Maven 3.2.x+；[下载 (opens new window)](https://maven.apache.org/download.cgi).[配置 (opens new window)](https://maven.apache.org/settings.html)。
> 4. 3个或3个以上Nacos节点才能构成集群。
>
> [link(opens new window)](https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html)

Nacos下载Linux版

- [https://github.com/alibaba/nacos/releases/tag/1.1.4(opens new window)](https://github.com/alibaba/nacos/releases/tag/1.1.4)
- nacos-server-1.1.4.tar.gz 解压后安装

- 使用XFTP工具将压缩包上传到Linux系统

## Nacos集群配置(上)

集群配置步骤(重点)

1. **Linux服务器上mysql数据库配置**

SQL脚本在哪里 - 目录/usr/local/nacos/nacos/conf/nacos-mysql.sql

```sh
[root@master conf]# pwd
/usr/local/nacos/nacos/conf
[root@master conf]# ll
总用量 52
-rw-r--r-- 1 502 games  1564 11月  4 2019 application.properties
-rw-r--r-- 1 502 games   408 10月 11 2019 application.properties.example
-rw-r--r-- 1 502 games    58 10月 11 2019 cluster.conf.example
-rw-r--r-- 1 502 games 20210 11月  4 2019 nacos-logback.xml
-rw-r--r-- 1 502 games  9788 10月 11 2019 nacos-mysql.sql
-rw-r--r-- 1 502 games  7196 10月 11 2019 schema.sql
```

自己Linux机器上的Mysql数据库上运行

```sh
mysql> show tables;
+------------------------+
| Tables_in_nacos_config |
+------------------------+
| config_info            |
| config_info_aggr       |
| config_info_beta       |
| config_info_tag        |
| config_tags_relation   |
| group_capacity         |
| his_config_info        |
| roles                  |
| tenant_capacity        |
| tenant_info            |
| users                  |
+------------------------+
11 rows in set (0.00 sec)
```

1. **application.properties配置**

位置

```sh
[root@master conf]# pwd
/usr/local/nacos/nacos/conf
[root@master conf]# cp application.properties.example application.properties
cp：是否覆盖"application.properties"？ y
[root@master conf]# ls
application.properties  application.properties.example  cluster.conf.example  nacos-logback.xml  nacos-mysql.sql  schema.sql
[root@master conf]# vim application.properties
```

添加以下内容，设置数据源

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=12345678
```

1. **Linux服务器上nacos的集群配置cluster.conf**

梳理出3台nacos集器的不同服务端口号，设置3个端口：

- 3333
- 4444
- 5555

复制出cluster.conf

```sh
[root@master conf]# pwd
/usr/local/nacos/nacos/conf
[root@master conf]# cp cluster.conf.example cluster.conf
[root@master conf]# ll
总用量 56
-rw-r--r-- 1  502 games   431 8月  25 20:22 application.properties
-rw-r--r-- 1  502 games   408 10月 11 2019 application.properties.example
-rw-r--r-- 1 root root     58 8月  25 20:22 cluster.conf
-rw-r--r-- 1  502 games    58 10月 11 2019 cluster.conf.example
-rw-r--r-- 1  502 games 20210 11月  4 2019 nacos-logback.xml
-rw-r--r-- 1  502 games  9788 10月 11 2019 nacos-mysql.sql
-rw-r--r-- 1  502 games  7196 10月 11 2019 schema.sql
[root@master conf]# vim cluster.conf
```

内容

```sh
192.168.91.200:3333
192.168.91.200:4444
192.168.91.200:5555
```

**注意**，这个IP不能写127.0.0.1，必须是Linux命令`hostname -i`能够识别的IP

```sh
[root@master conf]# hostname -i
192.168.91.200
```

1. **编辑Nacos的启动脚本startup.sh，使它能够接受不同的启动端口**

/usr/local/nacos/nasos/bin目录下有startup.sh

```sh
[root@master conf]# cd ../bin/
[root@master bin]# ls
shutdown.cmd  shutdown.sh  startup.cmd  startup.sh
```

平时单机版的启动，都是./startup.sh即可

但是，集群启动，我们希望可以类似其它软件的shell命令，传递不同的端口号启动不同的nacos实例。

命令: `./startup.sh -p 3333`表示启动端口号为3333的nacos服务器实例，和上一步的cluster.conf配置的一致。

修改内容

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.3vd5fzqvuuu0.webp)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.3x80aqonafo0.webp)

```sh
while getopts ":m:f:s:p:" opt
do
    case $opt in
        m)
            MODE=$OPTARG;;
        f)
            FUNCTION_MODE=$OPTARG;;
        s)
            SERVER=$OPTARG;;
        p)
            PORT=$OPTARG;;
        ?)
        echo "Unknown parameter"
        exit 1;;
    esac
done
...
nohup $JAVA -Dserver.port=${PORT} ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
```

执行方式 - `startup.sh - p 端口号`

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220825/image.l45pdndezvk.webp)

## Nacos集群配置(下)

[Nginx安装](https://frxcat.fun/middleware/Nginx/Nginx_install/#nginx环境安装)

1. **Nginx的配置，由它作为负载均衡器**

修改nginx的配置文件 - nginx.conf

```sh
[root@master conf]# pwd
/usr/local/nginx/conf
[root@master conf]# ll
总用量 96
-rw-r--r-- 1 root root 1637 8月   7 16:35 \
-rw-r--r-- 1 root root  740 7月  30 15:58 ！
-rw-r--r-- 1 root root 1077 7月  27 20:31 fastcgi.conf
-rw-r--r-- 1 root root 1077 7月  27 20:33 fastcgi.conf.default
-rw-r--r-- 1 root root 1007 7月  27 20:31 fastcgi_params
-rw-r--r-- 1 root root 1007 7月  27 20:33 fastcgi_params.default
-rw-r--r-- 1 root root   43 8月   6 21:43 htpasswd
-rw-r--r-- 1 root root 2837 7月  27 20:33 koi-utf
-rw-r--r-- 1 root root 2223 7月  27 20:33 koi-win
-rw-r--r-- 1 root root 5349 7月  27 20:31 mime.types
-rw-r--r-- 1 root root 5349 7月  27 20:33 mime.types.default
-rw-r--r-- 1 root root 1525 8月   7 16:41 nginx.conf
-rw-r--r-- 1 root root 1476 7月  30 13:52 nginx.conf.backup
-rw-r--r-- 1 root root 3018 8月   6 20:58 nginx.conf.static.bak
-rw-r--r-- 1 root root  168 7月  31 16:20 nginx_gzip.conf
-rw-r--r-- 1 root root  790 7月  30 16:08 nginx_server.conf
-rw-r--r-- 1 root root  749 7月  31 20:04 nginxTestServer.conf
-rw-r--r-- 1 root root  636 7月  27 20:31 scgi_params
-rw-r--r-- 1 root root  636 7月  27 20:33 scgi_params.default
-rw-r--r-- 1 root root  664 7月  27 20:31 uwsgi_params
-rw-r--r-- 1 root root  664 7月  27 20:33 uwsgi_params.default
-rw-r--r-- 1 root root 3610 7月  27 20:33 win-utf
[root@master conf]# vim nginx.conf
```

修改内容

```nginx
    upstream cluster{
        server 127.0.0.1:3333;
        server 127.0.0.1:4444;
        server 127.0.0.1:5555;
    }

    server {
        listen 1111;
        server_name localhost;

        location / {
            proxy_pass http://cluster;
        }
    }
```

按照指定启动

```sh
[root@master conf]# nginx -c /usr/local/nginx/conf/nginx.conf
[root@master conf]# nginx -s reload
```

1. **截止到此处，1个Nginx+3个nacos注册中心+1个mysql**

**测试**

- 启动3个nacos注册中心

  - `./startup.sh - p 3333`
  - `./startup.sh - p 4444`
  - `./startup.sh - p 5555`

  ![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220826/image.yezln9eghrk.webp)

  - 查看nacos进程启动数`ps -ef | grep nacos | grep -v grep | wc -l`



```sh
[root@master bin]# ps -ef | grep nacos | grep -v grep | wc -l
3
```

- 启动nginx
  - `./nginx -c /usr/local/nginx/conf/nginx.conf`
  - 查看nginx进程`ps -ef| grep nginx`



```sh
[root@master bin]# nginx -c /usr/local/nginx/conf/nginx.conf
[root@master bin]# ps -ef|grep nginx
root       2812      1  0 21:47 ?        00:00:00 nginx: master process nginx -c /usr/local/nginx/conf/nginx.conf
nobody     2813   2812  0 21:47 ?        00:00:00 nginx: worker process
root       2824   1977  0 21:47 pts/0    00:00:00 grep --color=auto nginx
```

- 测试通过nginx，访问nacos - [http://192.168.91.200:1111/nacos/#/login(opens new window)](http://192.168.91.200:1111/nacos/#/login)

![1661484342465](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220826/1661484342465.2x6n9yc3nvg0.webp)

- 新建一个配置测试

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220826/image.5u823eqyx3c0.webp)

- 新建后，可在linux服务器的mysql新插入一条记录

```sh
mysql> select * from config_info;
+----+-------------------+---------------+--------------------------------------------+----------------------------------+---------------------+---------------------+----------+-----------+----------+-----------+--------+-------+--------+------+----------+
| id | data_id           | group_id      | content                                    | md5                              | gmt_create          | gmt_modified        | src_user | src_ip    | app_name | tenant_id | c_desc | c_use | effect | type | c_schema |
+----+-------------------+---------------+--------------------------------------------+----------------------------------+---------------------+---------------------+----------+-----------+----------+-----------+--------+-------+--------+------+----------+
|  1 | frx01-config.yaml | DEFAULT_GROUP | config:
    info: frx01-config.yaml2022.8 | 17c83f9c04dd207ffc6c9d87c456a316 | 2022-08-26 11:30:24 | 2022-08-26 11:30:24 | NULL     | 127.0.0.1 |          |           | NULL   | NULL  | NULL   | yaml | NULL     |
+----+-------------------+---------------+--------------------------------------------+----------------------------------+---------------------+---------------------+----------+-----------+----------+-----------+--------+-------+--------+------+----------+
```

- 让微服务cloudalibaba-provider-payment9002启动注册进nacos集群 - 修改配置文件

```yaml
server:
  port: 9002

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        #配置Nacos地址
        #server-addr: Localhost:8848
        #换成nginx的1111端口，做集群
        server-addr: 192.168.91.200:1111

management:
  endpoints:
    web:
      exposure:
        inc1ude: '*'
```

- 启动微服务cloudalibaba-provider-payment9002
- 访问nacos，查看注册结果

![1661485514866](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220826/1661485514866.5gpg606b9d80.webp)

- 注册成功

**高可用小总结**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220826/image.5d23j4rupes0.webp)

# Sentinel 实现熔断与限流

## Sentinel是什么

[官方Github(opens new window)](https://github.com/alibaba/Sentinel)

[官方文档(opens new window)](https://sentinelguard.io/zh-cn/docs/introduction.html)

**Sentinel 是什么？**

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

Sentinel 的主要特性：

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220826/image.6o17tpjuxww0.webp)

[link(opens new window)](https://github.com/alibaba/Sentinel/wiki/介绍#sentinel-是什么)

—句话解释，之前我们讲解过的Hystrix。

Hystrix与Sentinel比较：

- Hystrix
  1. 需要我们程序员自己手工搭建监控平台
  2. 没有一套web界面可以给我们进行更加细粒度化得配置流控、速率控制、服务熔断、服务降级
- Sentinel
  1. 单独一个组件，可以独立出来。
  2. 直接界面化的细粒度统一配置。

约定 > 配置 > 编码

都可以写在代码里面，但是我们本次还是大规模的学习使用配置和注解的方式，尽量少写代码

- Sentinel生态

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220826/image.1b0m1x3sals0.webp)

> sentinel 英 [ˈsentɪnl] 美 [ˈsentɪnl] n. 哨兵

## Sentinel下载安装运行

[官方文档(opens new window)](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_sentinel)

服务使用中的各种问题：

- 服务雪崩
- 服务降级
- 服务熔断
- 服务限流

Sentinel 分为两个部分：

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

安装步骤：

- 下载

  - https://github.com/alibaba/Sentinel/releases
  - 下载到本地sentinel-dashboard-1.8.2.jar

- 运行命令

  - 前提

    - Java 8 环境
    - 8080端口不能被占用

    

    ```sh
    netstat -ano | findstr 8080
    taskkill -pid pidnumber -f
    ```

    1
    2

  - 命令

    - `java -jar sentinel-dashboard-1.8.2.jar`

- 访问Sentinel管理界面

  - [http://localhost:8080(opens new window)](http://localhost:8080/)
  - 登录账号密码均为sentinel

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220826/image.6y6m807az4s0.webp)

## Sentinel初始化监控

**启动Nacos8848成功**

**新建工程 - cloudalibaba-sentinel-service8401**

POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-sentinel-service8401</artifactId>

    <dependencies>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.frx01.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件+actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>4.6.3</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

YML

```yaml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080   #配置Sentinel dashboard地址
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描，直至找到未被占用的端口
        port: 8719

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持
```

主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
public class MainApp8401 {
    public static void main(String[] args) {
        SpringApplication.run(MainApp8401.class,args);
    }
}
```

业务类FlowLimitController

```java
@RestController
@Slf4j
public class FlowLimitController {

    @GetMapping("/testA")
    public String testA(){
        return "----------------testA";
    }

    @GetMapping("/testB")
    public String testB(){
        log.info(Thread.currentThread().getName()+"\t"+".....testB");
        return "----------------testB";
    }
}
```

启动Sentinel8080 - `java -jar sentinel-dashboard-1.7.0.jar`

**启动微服务8401**

**启动8401微服务后查看sentienl控制台**

- 刚启动，空空如也，啥都没有

Sentinel采用的懒加载说明

- 执行一次访问即可
  - [http://localhost:8401/testA(opens new window)](http://localhost:8401/testA)
  - [http://localhost:8401/testB(opens new window)](http://localhost:8401/testB)
- 效果 - sentinel8080正在监控微服务8401

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.1x94hxj5ov4w.webp)

## Sentinel流控规则简介

基本介绍

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.3kvfyj3ozgo0.webp)

进一步解释说明：

- 资源名：唯一名称，默认请求路径。
- 针对来源：Sentinel可以针对调用者进行限流，填写微服务名，默认default（不区分来源）。
- 阈值类型/单机阈值：
  - QPS(每秒钟的请求数量)︰当调用该API的QPS达到阈值的时候，进行限流。
  - 线程数：当调用该API的线程数达到阈值的时候，进行限流。
- 是否集群：不需要集群。
- 流控模式：
- 直接：API达到限流条件时，直接限流。
  - 关联：当关联的资源达到阈值时，就限流自己。
  - 链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流)【API级别的针对来源】。
- 流控效果：
  - 快速失败：直接失败，抛异常。
  - Warm up：根据Code Factor（冷加载因子，默认3）的值，从阈值/codeFactor，经过预热时长，才达到设置的QPS阈值。
  - 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效。

## Sentinel流控-QPS直接失败

**直接 -> 快速失败（系统默认）**

**配置及说明**

表示1秒钟内查询1次就是OK，若超过次数1，就直接->快速失败，报默认错误

![1661597646696](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/1661597646696.2p5k0wl2bii0.webp)

**测试**

快速多次点击访问[http://localhost:8401/testA(opens new window)](http://localhost:8401/testA)

**结果**

返回页面 Blocked by Sentinel (flow limiting)

**源码**

com.alibaba.csp.sentinel.slots.block.flow.controller.DefaultController

**思考**

直接调用默认报错信息，技术方面OK，但是，是否应该有我们自己的后续处理？类似有个fallback的兜底方法?

## Sentinel流控-线程数直接失败

线程数：当调用该API的线程数达到阈值的时候，进行限流。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.56askfjb2x00.webp)

只能有一个线程访问

## Sentinel流控-关联

**是什么？**

- 当自己关联的资源达到阈值时，就限流自己
- 当与A关联的资源B达到阀值后，就限流A自己（B惹事，A挂了）

**设置testA**

当关联资源/testB的QPS阀值超过1时，就限流/testA的Rest访问地址，**当关联资源到阈值后限制配置好的资源名**。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.6pj6bmtz4c80.webp)

**Postman模拟并发密集访问testB**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.77mpmp6l1v40.webp)

访问testB成功

- 浏览器访问A，发现A挂了

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.1dhix5vbmuv4.webp)

## Sentinel流控-预热

> **Warm Up**
>
> Warm Up（`RuleConstant.CONTROL_BEHAVIOR_WARM_UP`）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。详细文档可以参考 [流量控制 - Warm Up 文档 (opens new window)](https://github.com/alibaba/Sentinel/wiki/限流---冷启动)，具体的例子可以参见 WarmUpFlowDemo。
>
> 通常冷启动的过程系统允许通过的 QPS 曲线如下图所示：
>
> ![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.7jvfq5bcsp40.webp)
>
> [link(opens new window)](https://github.com/alibaba/Sentinel/wiki/流量控制#warm-up)

> 默认coldFactor为3，即请求QPS 从 threshold / 3开始，经预热时长逐渐升至设定的QPS阈值。[link(opens new window)](https://github.com/alibaba/Sentinel/wiki/流量控制#warm-up)

**源码** - com.alibaba.csp.sentinel.slots.block.flow.controller.WarmUpController

**WarmUp配置**

案例，阀值为10+预热时长设置5秒。

系统初始化的阀值为10/ 3约等于3,即阀值刚开始为3;然后过了5秒后阀值才慢慢升高恢复到10

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.318rm89qk0w0.webp)

**测试**

多次快速点击http://localhost:8401/testB - 刚开始不行，后续慢慢OK

**应用场景**

如：秒杀系统在开启的瞬间，会有很多流量上来，很有可能把系统打死，预热方式就是把为了保护系统，可慢慢的把流量放进来,慢慢的把阀值增长到设置的阀值。

## Sentinel流控-排队等待

速排队，让请求以均匀的速度通过，阀值类型必须设成QPS，否则无效。

设置：/testA每秒1次请求，超过的话就排队等待，等待的超时时间为20000毫秒。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.r2ezbq3xheo.webp)

> 匀速排队
>
> 匀速排队（RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。详细文档可以参考 [流量控制 - 匀速器模式 (opens new window)](https://github.com/alibaba/Sentinel/wiki/流量控制-匀速排队模式)，具体的例子可以参见 [PaceFlowDemo (opens new window)](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/PaceFlowDemo.java)。
>
> 该方式的作用如下图所示：
>
> ![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.3v2m5sm14y00.webp)
>
> 这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。
>
> > 注意：匀速排队模式暂时不支持 QPS > 1000 的场景。
>
> [link(opens new window)](https://github.com/alibaba/Sentinel/wiki/流量控制#匀速排队)

源码 - com.alibaba.csp.sentinel.slots.block.flow.controller.RateLimiterController

**测试**

- 添加日志记录代码到FlowLimitController的testA方法

```java
@RestController
@Slf4j
public class FlowLimitController {
    @GetMapping("/testA")
    public String testA()
    {
        log.info(Thread.currentThread().getName()+"\t"+"...testA");//<----
        return "------testA";
    }

    ...
}
```

- Postman模拟并发密集访问testA。具体操作参考[Sentinel流控-关联](https://frxcat.fun/Spring/SpringCloud/Sentinel_/#sentinel流控-关联)
- 后台结果

```java
2022-08-27 20:45:22.214  INFO 17540 --- [nio-8401-exec-2] c.f.s.controller.FlowLimitController     : http-nio-8401-exec-2	.....testB
2022-08-27 20:45:23.329  INFO 17540 --- [nio-8401-exec-3] c.f.s.controller.FlowLimitController     : http-nio-8401-exec-3	.....testB
2022-08-27 20:45:24.434  INFO 17540 --- [nio-8401-exec-5] c.f.s.controller.FlowLimitController     : http-nio-8401-exec-5	.....testB
2022-08-27 20:45:25.523  INFO 17540 --- [nio-8401-exec-6] c.f.s.controller.FlowLimitController     : http-nio-8401-exec-6	.....testB
2022-08-27 20:45:26.623  INFO 17540 --- [nio-8401-exec-7] c.f.s.controller.FlowLimitController     : http-nio-8401-exec-7	.....testB
2022-08-27 20:45:27.731  INFO 17540 --- [nio-8401-exec-8] c.f.s.controller.FlowLimitController     : http-nio-8401-exec-8	.....testB
2022-08-27 20:45:28.844  INFO 17540 --- [nio-8401-exec-9] c.f.s.controller.FlowLimitController     : http-nio-8401-exec-9	.....testB
2022-08-27 20:45:29.933  INFO 17540 --- [io-8401-exec-10] c.f.s.controller.FlowLimitController     : http-nio-8401-exec-10	.....testB
2022-08-27 20:45:31.035  INFO 17540 --- [nio-8401-exec-1] c.f.s.controller.FlowLimitController     : http-nio-8401-exec-1	.....testB
2022-08-27 20:45:32.153  INFO 17540 --- [nio-8401-exec-2] c.f.s.controller.FlowLimitController     : h
```

## Sentinel降级简介

[官方文档(opens new window)](https://github.com/alibaba/Sentinel/wiki/熔断降级)

> **熔断降级概述**
>
> 除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高可用的重要措施之一。一个服务常常会调用别的模块，可能是另外的一个远程服务、数据库，或者第三方 API 等。例如，支付的时候，可能需要远程调用银联提供的 API；查询某个商品的价格，可能需要进行数据库查询。然而，这个被依赖服务的稳定性是不能保证的。如果依赖的服务出现了不稳定的情况，请求的响应时间变长，那么调用服务的方法的响应时间也会变长，线程会产生堆积，最终可能耗尽业务自身的线程池，服务本身也变得不可用。
>
> 现代微服务架构都是分布式的，由非常多的服务组成。不同服务之间相互调用，组成复杂的调用链路。以上的问题在链路调用中会产生放大的效果。复杂链路上的某一环不稳定，就可能会层层级联，最终导致整个链路都不可用。因此我们需要对不稳定的**弱依赖服务调用**进行熔断降级，暂时切断不稳定调用，避免局部不稳定因素导致整体的雪崩。熔断降级作为保护自身的手段，通常在客户端（调用端）进行配置。
>
> [link(opens new window)](https://github.com/alibaba/Sentinel/wiki/熔断降级#概述)

- RT（平均响应时间，秒级）
  - 平均响应时间 超出阈值 且 在时间窗口内通过的请求>=5，两个条件同时满足后触发降级。
  - 窗口期过后关闭断路器。
  - RT最大4900（更大的需要通过-Dcsp.sentinel.statistic.max.rt=XXXX才能生效）。
- 异常比列（秒级）
- QPS >= 5且异常比例（秒级统计）超过阈值时，触发降级;时间窗口结束后，关闭降级 。
- 异常数(分钟级)
  - 异常数(分钟统计）超过阈值时，触发降级;时间窗口结束后，关闭降级

Sentinel熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高)，对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。

当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）。

Sentinei的断路器是没有类似Hystrix半开状态的。(Sentinei 1.8.0 已有半开状态)

半开的状态系统自动去检测是否请求有异常，没有异常就关闭断路器恢复使用，有异常则继续打开断路器不可用。

具体可以参考[Hystrix的服务降级熔断限流概念初讲](https://frxcat.fun/Spring/SpringCloud/Hystrix_/#hystrix的服务降级熔断限流概念初讲)。

## Sentinel降级-RT

是什么？

> 平均响应时间(`DEGRADE_GRADE_RT`)：当1s内持续进入5个请求，对应时刻的平均响应时间（**秒级**）均超过阈值（ `count`，以ms为单位），那么在接下的时间窗口（`DegradeRule`中的`timeWindow`，以s为单位）之内，对这个方法的调用都会自动地熔断(抛出`DegradeException` )。注意Sentinel 默认统计的RT上限是4900 ms，超出此阈值的都会算作4900ms，若需要变更此上限可以通过启动配置项`-Dcsp.sentinel.statistic.max.rt=xxx`来配置。

**注意**：Sentinel 1.7.0才有**平均响应时间**（`DEGRADE_GRADE_RT`），Sentinel 1.8.0的没有这项，取而代之的是**慢调用比例** (`SLOW_REQUEST_RATIO`)。

> 慢调用比例 (SLOW_REQUEST_RATIO)：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。
>
> [link(opens new window)](https://github.com/alibaba/Sentinel/wiki/熔断降级#熔断策略)

接下来讲解Sentinel 1.7.0的。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.1nmc1swgco68.webp)

**测试**

代码

```java
@RestController
@Slf4j
public class FlowLimitController {
    
	@GetMapping("/testD")
    public String testD(){
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.info("test 测试RT");
        return "---------------testD";
    }
}
```

配置

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.6pd9n2n3c4g0.webp)

jmeter压测

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.187hn49vgg80.webp)

结论

按照上述配置，永远一秒钟打进来10个线程（大于5个了）调用testD，我们希望200毫秒处理完本次任务，如果超过200毫秒还没处理完，在未来1秒钟的时间窗口内，断路器打开（保险丝跳闸）微服务不可用，保险丝跳闸断电了后续我停止jmeter，没有这么大的访问量了，断路器关闭（保险丝恢复），微服务恢复OK。

## Sentinel降级-异常比例

**是什么？**

> 异常比例(`DEGRADE_GRADE_EXCEPTION_RATIO`)：当资源的每秒请求量 >= 5，并且每秒异常总数占通过量的比值超过阈值（ `DegradeRule`中的 `count`）之后，资源进入降级状态，即在接下的时间窗口( `DegradeRule`中的`timeWindow`，以s为单位）之内，对这个方法的调用都会自动地返回。异常比率的阈值范围是`[0.0, 1.0]`，代表0% -100%。

**注意**，与Sentinel 1.8.0相比，有些不同（Sentinel 1.8.0才有的半开状态），Sentinel 1.8.0的如下：

> 异常比例 (`ERROR_RATIO`)：当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。[link(opens new window)](https://github.com/alibaba/Sentinel/wiki/熔断降级#熔断策略)

接下来讲解Sentinel 1.7.0的。

**测试**

代码

```java
@RestController
@Slf4j
public class FlowLimitController {

    ...

    @GetMapping("/testD")
    public String testD() {
        log.info("testD 异常比例");
        int age = 10/0;
        return "------testD";
    }
}
```

配置

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.2fqu8qr6nim8.webp)

jmeter

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220827/image.1m84918wopts.webp)

结论

按照上述配置，单独访问一次，必然来一次报错一次(int age = 10/0)，调一次错一次。

开启jmeter后，直接高并发发送请求，多次调用达到我们的配置条件了。断路器开启(保险丝跳闸)，微服务不可用了，不再报错error而是服务降级了。

## Sentinel热点key(上)

**基本介绍**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/image.48v9ncqzdz00.webp)

**官网**

[官方文档(opens new window)](https://github.com/alibaba/Sentinel/wiki/热点参数限流)

> 何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：
>
> - 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
> - 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制
>
> 热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。
>
> ![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/image.jnvgtfj7xww.webp)
>
> Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式。
>
> [link(opens new window)](https://github.com/alibaba/Sentinel/wiki/热点参数限流#overview)

承上启下复习start

兜底方法，分为系统默认和客户自定义，两种

之前的case，限流出问题后，都是用sentinel系统默认的提示: Blocked by Sentinel (flow limiting)

我们能不能自定？类似hystrix，某个方法出问题了，就找对应的兜底降级方法?

结论 - 从**@HystrixCommand到@SentinelResource**

**代码**

com.alibaba.csp.sentinel.slots.block.BlockException

```java
@RestController
@Slf4j
public class FlowLimitController{

    ...
   
	@GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey",blockHandler/*兜底方法*/ = "deal_testHotKey")
    public String testHostKey(@RequestParam(value = "p1",required = false)String p1,
                              @RequestParam(value = "p2",required = false)String p2){
        return "-------------testHotKey";
    }

    @GetMapping
    public String deal_testHotKey(String p1, String p2, BlockException exception){
        return "-------------deal_testHotKey,o(╥﹏╥)o";//sentinel系统默认的提示：Blocked by Sentinel (flow limiting)
    }
}
```

**配置**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/image.1q4q36pkaeao.webp)

一

- `@SentinelResource(value = "testHotKey")`
- 异常打到了前台用户界面看到，不友好

二

- `@SentinelResource(value = "testHotKey", blockHandler = "dealHandler_testHotKey")`
- 方法testHotKey里面第一个参数只要QPS超过每秒1次，马上降级处理
- 异常用了我们自己定义的兜底方法

**测试**

- error
  - [http://localhost:8401/testHotKey?p1=abc(opens new window)](http://localhost:8401/testHotKey?p1=abc)
  - [http://localhost:8401/testHotKey?p1=abc&p2=33(opens new window)](http://localhost:8401/testHotKey?p1=abc&p2=33)

- right
  - http://localhost:8401/testHotKey?p2=abc

## Sentinel热点key(下)

上述案例演示了第一个参数p1，当QPS超过1秒1次点击后马上被限流。

**参数例外项**

- 普通 - 超过1秒钟一个后，达到阈值1后马上被限流
- **我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样**
- 特例 - 假如当p1的值等于5时，它的阈值可以达到200

**配置**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/image.65m5scosrpw0.webp)

**测试**

- right - [http://localhost:8401/testHotKey?p1=5(opens new window)](http://localhost:8401/testHotKey?p1=5)

![QQ22918914922917714320220828143803](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/QQ22918914922917714320220828143803.3ae8743igh60.gif)

- error - [http://localhost:8401/testHotKey?p1=3(opens new window)](http://localhost:8401/testHotKey?p1=3)

![QQ22918914922917714320220828143535](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/QQ22918914922917714320220828143535.4sp17j0poq20.gif)

- 当p1等于5的时候，阈值变为200
- 当p1不等于5的时候，阈值就是平常的1

**前提条件** - 热点参数的注意点，参数必须是基本类型或者String

**其它**

在方法体抛异常

```java
@RestController
@Slf4j
public class FlowLimitController{

    ...
   
	@GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey",blockHandler/*兜底方法*/ = "deal_testHotKey")
    public String testHostKey(@RequestParam(value = "p1",required = false)String p1,
                              @RequestParam(value = "p2",required = false)String p2){
        int a = 10/0;
        return "-------------testHotKey";
    }

    @GetMapping
    public String deal_testHotKey(String p1, String p2, BlockException exception){
        return "-------------deal_testHotKey,o(╥﹏╥)o";//sentinel系统默认的提示：Blocked by Sentinel (flow limiting)
    }
}
```

将会抛出Spring Boot 2的默认异常页面，而不是兜底方法。

- @SentinelResource - 处理的是sentinel控制台配置的违规情况，有blockHandler方法配置的兜底处理;
- RuntimeException `int age = 10/0`，这个是java运行时报出的运行时异常RunTimeException，@SentinelResource不管

总结 - @SentinelResource主管配置出错，运行出错该走异常走异常

## Sentinel系统规则

[官方文档(opens new window)](https://github.com/alibaba/Sentinel/wiki/系统自适应限流)

> Sentinel 系统自适应限流从**整体维度**对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。[link(opens new window)](https://github.com/alibaba/Sentinel/wiki/系统自适应限流)

> **系统规则**
>
> 系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。
>
> 系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。
>
> 系统规则支持以下的模式:
>
> - Load 自适应（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
> - **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
> - **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
> - **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
> - **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。
>
> [link](https://github.com/alibaba/Sentinel/wiki/系统自适应限流#系统规则)

## SentinelResource配置(上)

*按资源名称限流 + 后续处理*

**启动Nacos成功**

**启动Sentinel成功**

**Module - cloudalibaba-sentinel-service8401**



```java
@RestController
public class RateLimitController {

    @GetMapping("/byResource")
    @SentinelResource(value = "byResource",blockHandler = "handleException")
    public CommonResult byResource(){
        return new CommonResult(200,"按资源名称限流测试OK",new Payment(2020L,"serial001"));
    }

    public CommonResult handleException(BlockException exception){
        return new CommonResult(444,exception.getClass().getCanonicalName()+"\t服务不可用");
    }
}
```

1
2
3
4
5
6
7
8
9
10
11
12
13

**配置流控规则**

配置步骤

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/image.7g67qp5jzig0.webp)

图形配置和代码关系

表示1秒钟内查询次数大于1，就跑到我们自定义的处流，限流

**测试**:[http://localhost:8401/byResource(opens new window)](http://localhost:8401/byResource)

1秒钟点击1下，OK

超过上述，疯狂点击，返回了自己定义的限流处理信息，限流发生



```json
{"code":444, "message":"com.alibaba.csp.sentinel.slots.block.flow.FlowException\t 服务不可用", "data":null}
```

1

**额外问题**

此时关闭问服务8401 -> Sentinel控制台，流控规则消失了，[如何解决](https://frxcat.fun/Spring/SpringCloud/Sentinel_/#sentinel持久化规则)

------

*按照Url地址限流 + 后续处理*

**通过访问的URL来限流，会返回Sentinel自带默认的限流处理信息**

**业务类RateLimitController**

```java
@RestController
public class RateLimitController {
    ...
        
	@GetMapping("/rateLimit/byUrl")
    @SentinelResource(value = "byUrl")
    public CommonResult byUrl(){
        return new CommonResult(200,"按url限流测试OK",new Payment(2020L,"serial002"));
    }
}
```

**Sentinel控制台配置**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/image.6cb3ofi255o0.webp)

**测试**

- 快速点击[http://localhost:8401/rateLimit/byUrl(opens new window)](http://localhost:8401/rateLimit/byUrl)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/image.6rbdgxikqxo0.webp)

- 结果 - 会返回Sentinel自带的限流处理结果 Blocked by Sentinel (flow limiting)

**上面兜底方案面临的问题**

1. 系统默认的，没有体现我们自己的业务要求。
2. 依照现有条件，我们自定义的处理方法又和业务代码耦合在一块，不直观。
3. 每个业务方法都添加—个兜底的，那代码膨胀加剧。
4. 全局统—的处理方法没有体现。

## SentinelResource配置(中)

客户自定义限流处理逻辑

自定义限流处理类 - 创建CustomerBlockHandler类用于自定义限流处理逻辑

```java
public class CustomerBlockHandler {

    public static CommonResult handlerException1(BlockException exception){
        return new CommonResult(4444,"按客户自定义,global handlerException----1");
    }

    public static CommonResult handlerException2(BlockException exception){
        return new CommonResult(4444,"按客户自定义,global handlerException----1");
    }
}
```

RateLimitController

```java
@RestController
public class RateLimitController {
    
    ...
	@GetMapping("/rateLimit/customerBlockHandler")
    @SentinelResource(value = "customerBlockHandler",
        blockHandlerClass = CustomerBlockHandler.class,
        blockHandler = "handlerException2")
    public CommonResult customerBlockHandler(){
        return new CommonResult(200,"按客户自定义",new Payment(2020L,"serial003"));
    }
}
```

Sentinel控制台配置

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/image.2fetseirolog.webp)

启动微服务后先调用一次 - [http://localhost:8401/rateLimit/customerBlockHandler (opens new window)](http://localhost:8401/rateLimit/customerBlockHandler)。

然后，多次快速刷新[http://localhost:8401/rateLimit/customerBlockHandler (opens new window)](http://localhost:8401/rateLimit/customerBlockHandler)。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220828/image.6ierww0guuc0.webp)

刷新后，我们自定义兜底方法的字符串信息就返回到前端。

## SentinelResource配置(下)

> **@SentinelResource 注解**
>
> > 注意：注解方式埋点不支持 private 方法。
>
> `@SentinelResource` 用于定义资源，并提供可选的异常处理和 fallback 配置项。 `@SentinelResource` 注解包含以下属性：
>
> - `value`：资源名称，必需项（不能为空）
>
> - `entryType`：entry 类型，可选项（默认为 `EntryType.OUT`）
>
> - `blockHandler` / `blockHandlerClass`: `blockHandler` 对应处理 `BlockException` 的函数名称，可选项。`blockHandler` 函数访问范围需要是 `public`，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 BlockException。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `blockHandlerClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。
>
> - ```
>   fallback
>   ```
>
>    
>
>   /
>
>   ```
>   fallbackClass
>   ```
>
>   ：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了
>
>   ```
>   exceptionsToIgnore
>   ```
>
>   里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：
>
>   - 返回值类型必须与原函数返回值类型一致；
>   - 方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
>   - fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。
>
> - ```
>   defaultFallback
>   ```
>
>   （since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了
>
>   ```
>   exceptionsToIgnore
>   ```
>
>   里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。defaultFallback 函数签名要求
>
>   - 返回值类型必须与原函数返回值类型一致；
>   - 方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
>   - defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。
>
> - `exceptionsToIgnore`（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。
>
> [link](https://github.com/alibaba/Sentinel/wiki/注解支持#sentinelresource-注解)

Sentinel主要有三个核心Api：

1. SphU定义资源
2. Tracer定义统计
3. ContextUtil定义了上下文

## Sentinel服务熔断Ribbon环境预说

sentinel整合ribbon+openFeign+fallback

Ribbon系列

- 启动nacos和sentinel
- 提供者9003/9004
- 消费者84

**提供者9003/9004**

新建cloudalibaba-provider-payment9003/9004，两个一样的做法

POM

```yaml
server:
  port: 9003

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
        include: '*'
```

**记得修改不同的端口号**

主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9003 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9003.class,args);
    }
}
```

业务类

```java
@RestController
public class PaymentController {

    @Value("{server.port}")
    private String serverPort;

    //模拟数据库
    public static HashMap<Long, Payment> hashMap = new HashMap<>();

    static {
        hashMap.put(1L,new Payment(1L,"28a8c1e3bc2742d8848569891fb42181"));
        hashMap.put(2L,new Payment(2L,"bba8c1e3bc2742d8848569891ac32182"));
        hashMap.put(3L,new Payment(3L,"6ua8c1e3bc2742d8848569891xt92183"));
    }

    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id){
        Payment payment = hashMap.get(id);
        CommonResult<Payment> result = new CommonResult<>(200, "from mysql,serverPort:  " + serverPort, payment);
        return result;
    }
}
```

测试地址 - [http://localhost:9003/paymentSQL/1(opens new window)](http://localhost:9003/paymentSQL/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.25fj5vyffykg.webp)

**消费者84**

新建cloudalibaba-consumer-nacos-order84

POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-consumer-nacos-order84</artifactId>
    <dependencies>
        <!--SpringCloud openfeign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.frx01.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

YML

```yaml
server:
  port: 84

spring:
  application:
    name: nacos-order-consumer
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

#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider

# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: false
```

主启动

```java
@EnableDiscoveryClient
@SpringBootApplication
//@EnableFeignClients
public class OrderNacosMain84 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain84.class,args);
    }
}
```

ApplicationContextConfig

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

CircleBreakerController

```java
@RestController
@Slf4j
public class CircleBreakerController {

    public static final String SERVICE_URL = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/common/fallback/{id}")
    @SentinelResource(value = "fallback")//没有配置
    public CommonResult<Payment> fallback(@PathVariable Long id){
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class);
        if(id==4){
            throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
        }else if(result.getData() == null){
            throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }
}
```

修改后请重启微服务

- 热部署对java代码级生效及时
- 对@SentinelResource注解内属性，有时效果不好

目的

- fallback管运行异常
- blockHandler管配置违规

测试地址 - http://localhost:84/consumer/fallback/1

## Sentinel服务熔断无配置

没有任何配置 - **如果ID为4，或大于5给用户error页面，不友好**

```java
@RestController
@Slf4j
public class CircleBreakerController {

    public static final String SERVICE_URL = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback")//没有配置
    public CommonResult<Payment> fallback(@PathVariable Long id){
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class);
        if(id==4){
            throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
        }else if(result.getData() == null){
            throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }
}
```

## Sentinel服务熔断只配置fallback

fallback只负责业务异常

```java
@RestController
@Slf4j
public class CircleBreakerController {

    public static final String SERVICE_URL = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    //@SentinelResource(value = "fallback")//没有配置
    @SentinelResource(value = "fallback",fallback = "handlerFallback")//fallback只负责业务异常
    public CommonResult<Payment> fallback(@PathVariable Long id){
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class);
        if(id==4){
            throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
        }else if(result.getData() == null){
            throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }

    //本例是fallback
    public CommonResult handlerFallback(@PathVariable Long id, Throwable e){
        Payment payment = new Payment(id, "null");
        return new CommonResult(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
    }
}
```

测试地址 - [http://localhost:84/consumer/fallback/4(opens new window)](http://localhost:84/consumer/fallback/4)

页面返回结果：

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.14s7o7pzawik.webp)

## Sentinel服务熔断只配置blockHandler

blockHandler只负责**sentinel控制台配置违规**

```java
@RestController
@Slf4j
public class CircleBreakerController {

    public static final String SERVICE_URL = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    //@SentinelResource(value = "fallback")//没有配置
    //@SentinelResource(value = "fallback",fallback = "handlerFallback")//fallback只负责业务异常
    @SentinelResource(value = "fallback",blockHandler = "blockHandler")//blockHandler只负责sentinel控制台配置违规
    public CommonResult<Payment> fallback(@PathVariable Long id){
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class);
        if(id==4){
            throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
        }else if(result.getData() == null){
            throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }

    //本例是fallback
    /*public CommonResult handlerFallback(@PathVariable Long id, Throwable e){
        Payment payment = new Payment(id, "null");
        return new CommonResult(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
    }*/

    //本例是blockHandler
    public CommonResult blockHandler(@PathVariable Long id, BlockException blockException){
        Payment payment = new Payment(id, "null");
        return new CommonResult(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
    }
}
```

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.r7qe8ck91f4.webp)

测试地址 - [http://localhost:84/consumer/fallback/4 (opens new window)](http://localhost:84/consumer/fallback/4),第一次访问页面报错，但是一秒内快速访问两次，结果:

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.4lktw021rss0.webp)

## Sentinel服务熔断fallback和blockHandler都配置

若blockHandler和fallback 都进行了配置，则被限流降级而抛出BlockException时只会进入blockHandler处理逻辑。

```java
@RestController
@Slf4j
public class CircleBreakerController {

    public static final String SERVICE_URL = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    //@SentinelResource(value = "fallback")//没有配置
    //@SentinelResource(value = "fallback",fallback = "handlerFallback")//fallback只负责业务异常
    //@SentinelResource(value = "fallback",blockHandler = "blockHandler")//blockHandler只负责sentinel控制台配置违规
    @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler")
    public CommonResult<Payment> fallback(@PathVariable Long id){
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class);
        if(id==4){
            throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
        }else if(result.getData() == null){
            throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }

    //本例是fallback
    public CommonResult handlerFallback(@PathVariable Long id, Throwable e){
        Payment payment = new Payment(id, "null");
        return new CommonResult(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
    }

    //本例是blockHandler
    public CommonResult blockHandler(@PathVariable Long id, BlockException blockException){
        Payment payment = new Payment(id, "null");
        return new CommonResult(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
    }
}
```

## Sentinel服务熔断exceptionsToIgnore

exceptionsToIgnore，忽略指定异常，即这些异常不用兜底方法处理。

```java
@RestController
@Slf4j
public class CircleBreakerController {

    ...

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler",
        exceptionsToIgnore = {IllegalArgumentException.class}) //<-------------
    public CommonResult<Payment> fallback(@PathVariable Long id){
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class);
        if(id==4){
            //exceptionsToIgnore属性有IllegalArgumentException.class，
            //所以IllegalArgumentException不会跳入指定的兜底程序。
            throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
        }else if(result.getData() == null){
            throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
        }
        return result;
    }
	...
}
```

## Sentinel服务熔断OpenFeign

**修改84模块**

- 84消费者调用提供者9003
- Feign组件一般是消费侧

POM

```xml
<!--SpringCloud openfeign -->

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

YML

```yaml
# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: true
```

业务类

带@Feignclient注解的业务接口，fallback = PaymentFallbackService.class

```java
@FeignClient(value = "nacos-payment-provider",fallback = PaymentFallbackService.class)
public interface PaymentService {

    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);
}
```

```java
@Component
public class PaymentFallbackService implements PaymentService {
    @Override
    public CommonResult<Payment> paymentSQL(Long id) {
        return new CommonResult<>(44444,"服务降级返回,---PaymentFallbackService",new Payment(id,"errorSerial"));
    }
}
```

Controller

```java
@RestController
@Slf4j
public class CircleBreakerController {

    ...
    
	//==================OpenFeign    
	@Resource
    private PaymentService paymentService;

    @GetMapping(value = "/consumer/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id){
        return paymentService.paymentSQL(id);
    }
}
```

主启动

```java
@EnableDiscoveryClient
@SpringBootApplication
@EnableFeignClients//<-------
public class OrderNacosMain84 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain84.class,args);
    }
}
```

测试 - [http://localhost:84/consumer/paymentSQL/1(opens new window)](http://localhost:84/consumer/paymentSQL/1)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.4ptxwjfudfk0.webp)

测试84调用9003，此时故意关闭9003微服务提供者，**84消费侧自动降级**，不会被耗死。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.1drrkjbpqig0.webp)

一直是端口9004

**熔断框架比较**

|       -        |                          Sentinel                          |        Hystrix         |           resilience4j           |
| :------------: | :--------------------------------------------------------: | :--------------------: | :------------------------------: |
|    隔离策略    |                信号量隔离（并发线程数限流）                | 线程池隔商/信号量隔离  |            信号量隔离            |
|  熔断降级策略  |               基于响应时间、异常比率、异常数               |      基于异常比率      |      基于异常比率、响应时间      |
|  实时统计实现  |                   滑动窗口（LeapArray）                    | 滑动窗口（基于RxJava） |         Ring Bit Buffer          |
|  动态规则配置  |                       支持多种数据源                       |     支持多种数据源     |             有限支持             |
|     扩展性     |                         多个扩展点                         |       插件的形式       |            接口的形式            |
| 基于注解的支持 |                            支持                            |          支持          |               支持               |
|      限流      |              基于QPS，支持基于调用关系的限流               |       有限的支持       |           Rate Limiter           |
|    流量整形    |            支持预热模式匀速器模式、预热排队模式            |         不支持         |      简单的Rate Limiter模式      |
| 系统自适应保护 |                            支持                            |         不支持         |              不支持              |
|     控制台     | 提供开箱即用的控制台，可配置规则、查看秒级监控，机器发观等 |     简单的监控查看     | 不提供控制台，可对接其它监控系统 |

## Sentinel持久化规则

**是什么**

一旦我们重启应用，sentinel规则将消失，生产环境需要将配置规则进行持久化。

**怎么玩**

将限流配置规则持久化进Nacos保存，只要刷新8401某个rest地址，sentinel控制台的流控规则就能看到，只要Nacos里面的配置不删除，针对8401上sentinel上的流控规则持续有效。

**步骤**

修改cloudalibaba-sentinel-service8401

POM

```xml
<!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

YML

```yaml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719
      datasource: #<---------------------------关注点，添加Nacos数据源配置
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持
```

添加Nacos业务规则配置

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.4awbthpovfw0.webp)



```json
[{
    "resource": "/rateLimit/byUrl",
    "IimitApp": "default",
    "grade": 1,
    "count": 1, 
    "strategy": 0,
    "controlBehavior": 0,
    "clusterMode": false
}]
```

- resource：资源名称；
- limitApp：来源应用；
- grade：阈值类型，0表示线程数, 1表示QPS；
- count：单机阈值；
- strategy：流控模式，0表示直接，1表示关联，2表示链路；
- controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待；
- clusterMode：是否集群。

启动8401后刷新sentinel发现业务规则有了

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.4kh8x1jx9k80.webp)

快速访问测试接口 - http://localhost:8401/rateLimit/byUrl - 页面返回`Blocked by Sentinel (flow limiting)`

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.j0jdehf5s5s.webp)

停止8401再看sentinel - 停机后发现流控规则没有了

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220829/image.5gwcanu3evs0.webp)

重新启动8401再看sentinel

- 乍一看还是没有，稍等一会儿
- 多次调用 - [http://localhost:8401/rateLimit/byUrl(opens new window)](http://localhost:8401/rateLimit/byUrl)
- 重新配置出现了，持久化验证通过

# Seata 分布式事务

## 分布式事务问题由来

分布式前

- 单机单库没这个问题
- 从1:1 -> 1:N -> N:N

单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用,分别使用三个独立的数据源，业务操作需要调用三三 个服务来完成。此时**每个服务内部的数据一致性由本地事务来保证， 但是全局的数据一致性问题没法保证**。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220831/image.4allg2qmxww0.webp)

一句话：**一次业务操作需要跨多个数据源或需要跨多个系统进行远程调用，就会产生分布式事务问题**。

## Seata术语

**是什么**

Seata是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。

[官方网址(opens new window)](http://seata.io/zh-cn/)

**能干嘛**

一个典型的分布式事务过程

分布式事务处理过程的一ID+三组件模型：

- Transaction ID XID 全局唯一的事务ID
- 三组件概念
  - TC (Transaction Coordinator) - 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。
  - TM (Transaction Manager) - 事务管理器：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
  - RM (Resource Manager) - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

处理过程：

1. TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID；
2. XID在微服务调用链路的上下文中传播；
3. RM向TC注册分支事务，将其纳入XID对应全局事务的管辖；
4. TM向TC发起针对XID的全局提交或回滚决议；
5. TC调度XID下管辖的全部分支事务完成提交或回滚请求。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220831/image.4jnpi8ows1o0.webp)

## Seata-Server安装

**去哪下**

发布说明: [https://github.com/seata/seata/releases(opens new window)](https://github.com/seata/seata/releases)

**怎么玩**

本地@Transactional

全局@GlobalTransactional

**SEATA 的分布式交易解决方案**

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220831/image.79kbqiibjmw0.webp)

我们只需要使用一个 `@GlobalTransactional` 注解在业务方法上:

**Seata-Server安装**

- [1.4版本的使用(opens new window)](https://www.yuque.com/mrlinxi/pxvr4g/nyye5k)

官网地址 - [http://seata.io/zh-cn/(opens new window)](http://seata.io/zh-cn/)

下载版本 - 0.9.0

seata-server-0.9.0.zip解压到指定目录并修改conf目录下的file.conf配置文件

先备份原始file.conf文件

主要修改:自定义事务组名称+事务日志存储模式为db +数据库连接信息

file.conf

service模块

```nginx
service {
    ##fsp_tx_group是自定义的
    vgroup_mapping.my.test.tx_group="fsp_tx_group" 
    default.grouplist = "127.0.0.1:8091"
    enableDegrade = false
    disable = false
    max.commitretry.timeout= "-1"
    max.ollbackretry.timeout= "-1"
}
```

store模

```nginx
## transaction log store
store {
	## store mode: file, db
	## 改成db
	mode = "db"
	
	## file store
	file {
		dir = "sessionStore"
		
		# branch session size, if exceeded first try compress lockkey, still exceeded throws exceptions
		max-branch-session-size = 16384
		# globe session size, if exceeded throws exceptions
		max-global-session-size = 512
		# file buffer size, if exceeded allocate new buffer
		file-write-buffer-cache-size = 16384
		# when recover batch read size
		session.reload.read_size= 100
		# async, sync
		flush-disk-mode = async
	}

	# database store
	db {
		## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
		datasource = "dbcp"
		## mysql/oracle/h2/oceanbase etc.
		## 配置数据源
		db-type = "mysql"
		driver-class-name = "com.mysql.jdbc.Driver"
		url = "jdbc:mysql://127.0.0.1:3306/seata"
		user = "root"
		password = "你自己密码"
		min-conn= 1
		max-conn = 3
		global.table = "global_table"
		branch.table = "branch_table"
		lock-table = "lock_table"
		query-limit = 100
	}
}
```

mysql5.7数据库新建库seata，在seata库里建表

建表db_store.sql在\seata-server-0.9.0\seata\conf目录里面

```sql
--- seata  分布式事务
-- the table to store GlobalSession data
CREATE DATABASE seata;
USE seata;
DROP TABLE IF EXISTS `global_table`;
CREATE TABLE `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY                         `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY                         `idx_transaction_id` (`transaction_id`)
);

-- the table to store BranchSession data
DROP TABLE IF EXISTS `branch_table`;
CREATE TABLE `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `lock_key`          VARCHAR(128),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME,
    `gmt_modified`      DATETIME,
    PRIMARY KEY (`branch_id`),
    KEY                 `idx_xid` (`xid`)
);

-- the table to store lock data
DROP TABLE IF EXISTS `lock_table`;
CREATE TABLE `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(96),
    `transaction_id` LONG,
    `branch_id`      LONG,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`)
);
```

修改seata-server-0.9.0\seata\conf目录下的registry.conf配置文件

```nginx
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  # 改用为nacos
  type = "nacos"

  nacos {
  	## 加端口号
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
  }
  ...
}
```

- 启动Nacos,位置nacos\bin\startup.cmd

```sh
startup.cmd -m standalone
```

- 启动Seata,位置\seata\bin\seata-server.bat

```sh
seata-server.bat
```

- 查看Nacos服务列表

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.7cs0ri0kbnk0.webp)

## Seata业务数据库准备

以下演示都需要先启动Nacos后启动Seata,保证两个都OK。

分布式事务业务说明

这里我们会创建三个服务，一个订单服务，一个库存服务，一个账户服务。

当用户下单时,会在订单服务中创建一个订单, 然后通过远程调用库存服务来扣减下单商品的库存，再通过远程调用账户服务来扣减用户账户里面的余额，最后在订单服务中修改订单状态为已完成。

该操作跨越三个数据库，有两次远程调用，很明显会有分布式事务问题。

**一言蔽之**，下订单—>扣库存—>减账户(余额)。

创建业务数据库

- seata_ order：存储订单的数据库;
- seata_ storage：存储库存的数据库;
- seata_ account：存储账户信息的数据库。

建库SQL

```sql
CREATE DATABASE seata_order;
CREATE DATABASE seata_storage;
CREATE DATABASE seata_account;
```

按照上述3库分别建对应业务表

- seata_order库下建t_order表

```sql
use seata_order;
CREATE TABLE t_order (
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
    `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
    `count` INT(11) DEFAULT NULL COMMENT '数量',
    `money` DECIMAL(11,0) DEFAULT NULL COMMENT '金额',
    `status` INT(1) DEFAULT NULL COMMENT '订单状态: 0:创建中; 1:已完结'
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

SELECT * FROM t_order;
```

- seata_storage库下建t_storage表

```sql
use seata_storage;
CREATE TABLE t_storage (
`id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
`product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
`total` INT(11) DEFAULT NULL COMMENT '总库存',
`used` INT(11) DEFAULT NULL COMMENT '已用库存',
`residue` INT(11) DEFAULT NULL COMMENT '剩余库存'
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

INSERT INTO seata_storage.t_storage(`id`, `product_id`, `total`, `used`, `residue`)
VALUES ('1', '1', '100', '0','100');

SELECT * FROM t_storage;
```

- seata_account库下建t_account表

```sql
USE seata_account;
CREATE TABLE t_account(
	`id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY COMMENT 'id',
	`user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
	`total` DECIMAL(10,0) DEFAULT NULL COMMENT '总额度',
	`used` DECIMAL(10,0) DEFAULT NULL COMMENT '已用余额',
	`residue` DECIMAL(10,0) DEFAULT '0' COMMENT '剩余可用额度'
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

INSERT INTO seata_account.t_account(`id`, `user_id`, `total`, `used`, `residue`)
VALUES ('1', '1', '1000', '0', '1000');

SELECT * FROM t_account;
```

按照上述3库分别建对应的回滚日志表

- 订单-库存-账户3个库下**都需要建各自的回滚日志表**
- \seata-server-0.9.0\seata\conf目录下的db_ undo_ log.sql
- 建表SQL

```sql
-- the table to store seata xid data
-- 0.7.0+ add context
-- you must to init this sql for you business databese. the seata server not need it.
-- 此脚本必须初始化在你当前的业务数据库中，用于AT 模式XID记录。与server端无关（注：业务数据库）
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
drop table `undo_log`;
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

## Seata之Order-Module配置搭建

下订单 -> 减库存 -> 扣余额 -> 改（订单）状态

seata-order-service2001

POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>seata-order-service2001</artifactId>
    <dependencies>
        <!--nacos-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--seata-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>seata-all</artifactId>
                    <groupId>io.seata</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.seata</groupId>
            <artifactId>seata-all</artifactId>
            <version>1.4.2</version>
        </dependency>
        <!--feign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--web-actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--mysql-druid-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.37</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

配置文件

YML

```yaml
server:
  port: 2001

spring:
  application:
    name: seata-order-service
  cloud:
    alibaba:
      seata:
        #自定义事务组名称需要与seata-server中的对应
        tx-service-group: fsp_tx_group
    nacos:
      discovery:
        server-addr: localhost:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata_order
    username: root
    password: hsp

feign:
  hystrix:
    enabled: false

logging:
  level:
    io:
      seata: info

mybatis:
  mapperLocations: classpath:mapper/*.xml
```

domain

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T>{
    private Integer code;
    private String message;
    private T data;
    public CommonResult(Integer code,String message){
        this(code,message,null);
    }
}
```

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {
    private Long id;

    private Long userId;

    private Long productId;

    private Integer count;

    private BigDecimal money;

    private Integer status;//订单状态：0：创建中；1：已完结

}
```

file.conf

```nginx
transport {
  # tcp udt unix-domain-socket
  type = "TCP"
  #NIO NATIVE
  server = "NIO"
  #enable heartbeat
  heartbeat = true
  #thread factory for netty
  thread-factory {
    boss-thread-prefix = "NettyBoss"
    worker-thread-prefix = "NettyServerNIOWorker"
    server-executor-thread-prefix = "NettyServerBizHandler"
    share-boss-worker = false
    client-selector-thread-prefix = "NettyClientSelector"
    client-selector-thread-size = 1
    client-worker-thread-prefix = "NettyClientWorkerThread"
    # netty boss thread size,will not be used for UDT
    boss-thread-size = 1
    #auto default pin or 8
    worker-thread-size = 8
  }
  shutdown {
    # when destroy server, wait seconds
    wait = 3
  }
  serialization = "seata"
  compressor = "none"
}

service {

  vgroup_mapping.fsp_tx_group = "default" #修改自定义事务组名称

  default.grouplist = "127.0.0.1:8091"
  enableDegrade = false
  disable = false
  max.commit.retry.timeout = "-1"
  max.rollback.retry.timeout = "-1"
  disableGlobalTransaction = false
}


client {
  async.commit.buffer.limit = 10000
  lock {
    retry.internal = 10
    retry.times = 30
  }
  report.retry.count = 5
  tm.commit.retry.count = 1
  tm.rollback.retry.count = 1
}

## transaction log store
store {
  ## store mode: file、db
  mode = "db"

  ## file store
  file {
    dir = "sessionStore"

    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    max-branch-session-size = 16384
    # globe session size , if exceeded throws exceptions
    max-global-session-size = 512
    # file buffer size , if exceeded allocate new buffer
    file-write-buffer-cache-size = 16384
    # when recover batch read size
    session.reload.read_size = 100
    # async, sync
    flush-disk-mode = async
  }

  ## database store
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "root"
    password = "hsp"
    min-conn = 1
    max-conn = 3
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
  }
}
lock {
  ## the lock store mode: local、remote
  mode = "remote"

  local {
    ## store locks in user's database
  }

  remote {
    ## store locks in the seata's server
  }
}
recovery {
  #schedule committing retry period in milliseconds
  committing-retry-period = 1000
  #schedule asyn committing retry period in milliseconds
  asyn-committing-retry-period = 1000
  #schedule rollbacking retry period in milliseconds
  rollbacking-retry-period = 1000
  #schedule timeout retry period in milliseconds
  timeout-retry-period = 1000
}

transaction {
  undo.data.validation = true
  undo.log.serialization = "jackson"
  undo.log.save.days = 7
  #schedule delete expired undo_log in milliseconds
  undo.log.delete.period = 86400000
  undo.log.table = "undo_log"
}

## metrics settings
metrics {
  enabled = false
  registry-type = "compact"
  # multi exporters use comma divided
  exporter-list = "prometheus"
  exporter-prometheus-port = 9898
}

support {
  ## spring
  spring {
    # auto proxy the DataSource bean
    datasource.autoproxy = false
  }
}
```

registry.conf

```nginx
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
  }
  eureka {
    serviceUrl = "http://localhost:8761/eureka"
    application = "default"
    weight = "1"
  }
  redis {
    serverAddr = "localhost:6379"
    db = "0"
  }
  zk {
    cluster = "default"
    serverAddr = "127.0.0.1:2181"
    session.timeout = 6000
    connect.timeout = 2000
  }
  consul {
    cluster = "default"
    serverAddr = "127.0.0.1:8500"
  }
  etcd3 {
    cluster = "default"
    serverAddr = "http://localhost:2379"
  }
  sofa {
    serverAddr = "127.0.0.1:9603"
    application = "default"
    region = "DEFAULT_ZONE"
    datacenter = "DefaultDataCenter"
    cluster = "default"
    group = "SEATA_GROUP"
    addressWaitTime = "3000"
  }
  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "file"

  nacos {
    serverAddr = "localhost"
    namespace = ""
  }
  consul {
    serverAddr = "127.0.0.1:8500"
  }
  apollo {
    app.id = "seata-server"
    apollo.meta = "http://192.168.1.204:8801"
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    session.timeout = 6000
    connect.timeout = 2000
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  file {
    name = "file.conf"
  }
}
```

## Seata之Order-Module撸码(上)

Dao接口及实现

```java
@Mapper
public interface OrderDao {

    //1.新建订单
    void create(Order order);

    //2.修改订单状态，从0改为1
    void update(@Param("userId") Long userId,@Param("status") Integer status);
}
```

```xml
?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.frx01.springcloud.dao.OrderDao">
    
    <resultMap id="BaseResultMap" type="com.frx01.springcloud.domain.Order">
        <id column="id" property="id" jdbcType="BIGINT"/>
        <result column="user_id" property="userId" jdbcType="BIGINT"/>
        <result column="product_id" property="productId" jdbcType="BIGINT"/>
        <result column="count" property="count" jdbcType="INTEGER"/>
        <result column="money" property="money" jdbcType="DECIMAL"/>
        <result column="status" property="status" jdbcType="INTEGER"/>
    </resultMap>
    <insert id="create">
        insert into t_order (id,user_id,product_id,count,money,status)
        values(null,#{userId},#{productId},#{count},#{money},0);
    </insert>

     <update id="update">
         update T_order set status = 1
         where user_id=#{userId} and status=#{status};
     </update>
</mapper>
```

Service接口及实现

- OrderService
  - OrderServiceImpl
- StorageService
- AccountService

```java
public interface OrderService {
    void create(Order order);
}
```

```java
@FeignClient(value = "seata-storage-service")
public interface StorageService {

    @PostMapping(value = "/storage/decrease")
    CommonResult decrease(@RequestParam("productId") Long productId, @RequestParam("count") Integer count);

}
```

```java
@FeignClient(value = "seata-account-service")
public interface AccountService {

    @PostMapping(value = "/account/decrease")
    CommonResult decrease(@RequestParam("userId")BigInteger userId, @RequestParam("money") BigDecimal money);
}
```

```java
@Service
@Slf4j
public class OrderServiceImpl implements OrderService {

    @Resource
    private OrderDao orderDao;

    @Resource
    private StorageService storageService;

    @Resource
    private AccountService accountService;
	/**
     * 创建订单->调用库存服务扣减库存->调用账户服务扣减账户余额->修改订单状态
     * 简单说：下订单->扣库存->减余额->改状态
     */
    
    @Override
    public void create(Order order) {

        log.info("----------->开始新建订单");
        //1.新建订单
        orderDao.create(order);

        //2.扣减库存
        log.info("----------->订单微服务开始调用库存,做扣减Count");
        storageService.decrease(order.getProductId(),order.getCount());
        log.info("----------->订单微服务开始调用库存,做扣减end");

        //3.扣减账户
        log.info("----------->订单微服务开始调用账户，做扣减Money");
        accountService.decrease(order.getUserId(),order.getMoney());
        log.info("----------->订单微服务开始调用账户，做扣减end");

        //4.修改订单状态，从零到1，1代表已经完成
        log.info("----------->修改订单状态开始");
        orderDao.update(order.getUserId(),0);
        log.info("----------->修改订单状态end");

        log.info("----------->下订单结束了，O(∩_∩)O哈哈~");
    }
}
```

## Seata之Order-Module撸码(下)

Controller

```java
@RestController
public class OrderController {

    @Resource
    private OrderService orderService;

    @GetMapping("/order/create")
    public CommonResult create(Order order){
        orderService.create(order);
        return new CommonResult(200,"订单创建成功");
    }
}
```

Config配置

- MyBatisConfig
- DataSourceProxyConfig

```java
@Configuration
@MapperScan("com.frx01.springcloud.dao")
public class MyBatisConfig {
}
```

```java
/**
 * @author frx
 * @version 1.0
 * @date 2022/9/2  0:36
 * desc:说用Seata对数据源进行代理
 */
@Configuration
public class DataSourceProxyConfig {

    @Value("${mybatis.mapperLocations}")
    private String mapperLocations;
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource){

        return new DataSourceProxy(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSourceProxy dataSourceProxy) throws Exception{
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }
    
}
```

主启动

```java
//取消数据源的自动创建，而是使用自己定义的
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableDiscoveryClient
@EnableFeignClients
public class SeataOrderMainApp2001 {
    public static void main(String[] args) {
        SpringApplication.run(SeataOrderMainApp2001.class,args);
    }
}
```

## Seata之Storage-Module说明

与seata-order-service2001模块大致相同

seata- storage - service2002

POM（与seata-order-service2001模块大致相同）

YML

```yaml
server:
  port: 2002

spring:
  application:
    name: seata-storage-service
  cloud:
    alibaba:
      seata:
        tx-service-group: fsp_tx_group
    nacos:
      discovery:
        server-addr: localhost:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata_storage
    username: root
    password: 123456

logging:
  level:
    io:
      seata: info

mybatis:
  mapperLocations: classpath:mapper/*.xml
```

file.conf（与seata-order-service2001模块大致相同）

registry.conf（与seata-order-service2001模块大致相同）

domain

```java
@Data
public class Storage {

    private Long id;

    //产品id
    private Long productId;

    //总库存
    private Integer total;

    //已用库存
    private Integer used;

    //剩余库存
    private Integer residue;
}
```

CommonResult（与seata-order-service2001模块大致相同）

Dao接口及实现

```java
@Mapper
public interface StorageDao {

    //扣减库存
    void decrease(@Param("productId") Long productId,@Param("count") Integer count);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.frx01.springcloud.dao.StorageDao">

    <resultMap id="BaseResultMap" type="com.frx01.springcloud.domain.Storage">
        <id column="id" property="id" jdbcType="BIGINT"/>
        <result column="product_id" property="productId" jdbcType="BIGINT"/>
        <result column="total" property="total" jdbcType="INTEGER"/>
        <result column="used" property="used" jdbcType="INTEGER"/>
        <result column="residue" property="residue" jdbcType="INTEGER"/>
    </resultMap>
    <update id="decrease">
        UPDATE t_storage SET used = used + #{count},residue = residue - #{count}
                where product_id = #{productId}
    </update>

</mapper>
```

Service接口及实现

```java
public interface StorageService {

    //扣减库存
    void decrease(Long productId,Integer count);
}
```

```java
@Service
public class StorageServiceImpl implements StorageService {

    public static final Logger LOGGER = (Logger) LoggerFactory.getLogger(StorageServiceImpl.class);

    @Resource
    private StorageDao storageDao;

    //扣减库存
    @Override
    public void decrease(Long productId, Integer count) {
        LOGGER.info("----------->storage-service中扣减库存开始");
        storageDao.decrease(productId,count);
        LOGGER.info("----------->storage-service中扣减库存结束");
    }
}
```

Controller

```java
@RestController
public class StorageController {

    //扣减库存
    @Resource
    private StorageService storageService;
    @RequestMapping("/storage/decrease")
    public CommonResult decrease(Long productId,Integer count){
        storageService.decrease(productId,count);
        return new CommonResult(200,"扣减库存成功!");
    }
}
```

Config配置（与seata-order-service2001模块大致相同）

主启动（与seata-order-service2001模块大致相同）

## Seata之Account-Module说明

与seata-order-service2001模块大致相同

seata- account- service2003

POM（与seata-order-service2001模块大致相同）

YML

```yaml
server:
  port: 2003

spring:
  application:
    name: seata-account-service
  cloud:
    alibaba:
      seata:
        tx-service-group: fsp_tx_group
    nacos:
      discovery:
        server-addr: localhost:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata_account
    username: root
    password: hsp

feign:
  hystrix:
    enabled: false

logging:
  level:
    io:
      seata: info

mybatis:
  mapperLocations: classpath:mapper/*.xml
```

file.conf（与seata-order-service2001模块大致相同）

registry.conf（与seata-order-service2001模块大致相同）

domain

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Account {

    private Long id;
    
    //用户id
    private Long userId;

    //总额度
    private BigDecimal total;

    //已用额度
    private BigDecimal used;

    //剩余额度 
    private BigDecimal residue;
}
```

CommonResult（与seata-order-service2001模块大致相同）

Dao接口及实现

```java
@Mapper
public interface AccountDao {

    //扣减账户余额
    void decrease(@Param("userId") Long userId, @Param("money")BigDecimal money);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.frx01.springcloud.dao.AccountDao">

    <resultMap id="BaseResultMap" type="com.frx01.springcloud.domain.Account">
        <id column="id" property="id" jdbcType="BIGINT"/>
        <result column="user_id" property="userId" jdbcType="DECIMAL"/>
        <result column="total" property="total" jdbcType="DECIMAL"/>
        <result column="used" property="used" jdbcType="DECIMAL"/>
        <result column="residue" property="residue" jdbcType="DECIMAL"/>
    </resultMap>
    <update id="decrease">
        UPDATE t_account SET residue = residue - #{money},used = used + #{money}
            WHERE userId = #{userId};
    </update>
</mapper>
```

Service接口及实现

```java
public interface AccountService {

    /**
     * 扣减账户余额
     * @param userId 用户id
     * @param money 金额
     */
    void decrease(@RequestParam("userId") Long userId, @RequestParam("money") BigDecimal money);
}
```

```java
@Service
public class AccountServiceImpl implements AccountService {

    public static final Logger LOGGER = LoggerFactory.getLogger(AccountServiceImpl.class);

    @Resource
    private AccountDao accountDao;

    //扣减账户金额
    @Override
    public void decrease(Long userId, BigDecimal money) {
        LOGGER.info("----------->account-service中扣减账户余额开始");
        accountDao.decrease(userId, money);
        LOGGER.info("----------->account-service中扣减账户余额结束");
    }
}
```

Controller

```java
@RestController
public class AccountController {

    @Resource
    private AccountService accountService;

    //扣减账户余额

    @RequestMapping("/account/decrease")
    public CommonResult decrease(@RequestParam("userId") Long userId,@RequestParam("money") BigDecimal money){
        accountService.decrease(userId,money);
        return new CommonResult(200,"扣减账户余额成功!");
    }
}
```

Config配置（与seata-order-service2001模块大致相同）

主启动（与seata-order-service2001模块大致相同）

下订单 -> 减库存 -> 扣余额 -> 改（订单）状态

数据库初始情况：



```sql
SELECT * FROM t_order;
```

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.2b5f51i7f5zw.webp)



```sql
SELECT * FROM t_storage;
```

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20221012/image.5aeajach9ls0.webp)



```sql
SELECT * FROM t_account;
```

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.4w72fi0pycg0.webp)

正常下单 - [http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100(opens new window)](http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.3n1fd5325y00.webp)

数据库正常下单后状况：



```sql
SELECT * FROM t_order;
```

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.41l96vouw1c0.webp)



```sql
SELECT * FROM t_storage;
```

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.5vqwhtoqdnk0.webp)



```sql
SELECT * FROM t_account;
```

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.1kn0bm8iik5c.webp)

**超时异常，没加@GlobalTransactional**

模拟AccountServiceImpl添加超时



```java
@Service
public class AccountServiceImpl implements AccountService {

    public static final Logger LOGGER = LoggerFactory.getLogger(AccountServiceImpl.class);

    @Resource
    private AccountDao accountDao;

    //扣减账户金额
    @Override
    public void decrease(Long userId, BigDecimal money) {
        LOGGER.info("----------->account-service中扣减账户余额开始");
        //模拟超时异常，全局事务回滚
        //暂停几秒钟线程
        try {
            TimeUnit.SECONDS.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        accountDao.decrease(userId, money);
        LOGGER.info("----------->account-service中扣减账户余额结束");
    }
}
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23

另外，OpenFeign的调用默认时间是1s以内，所以最后会抛异常。

下单 - [http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100(opens new window)](http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100)

数据库情况



```sql
SELECT * FROM t_order;
```

1

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.2l5vnjjun1e.webp)



```sql
SELECT * FROM t_storage;
```

1

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.qpf2wazr99c.webp)



```sql
SELECT * FROM t_account;
```

1

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220901/image.706shnyil7g0.webp)

**故障情况**

- 当库存和账户金额扣减后，订单状态并没有设置为已经完成，没有从零改为1
- 而且由于feign的重试机制，账户余额还有可能被多次扣减

为了更清楚的看到，我把数据库数据清空。

超时异常，加了@GlobalTransactional**

用@GlobalTransactional标注OrderServiceImpl的create()方法。





















 











```java
@Service
@Slf4j
public class OrderServiceImpl implements OrderService {

    @Resource
    private OrderDao orderDao;

    @Resource
    private StorageService storageService;

    @Resource
    private AccountService accountService;

    /**
     * 创建订单->调用库存服务扣减库存->调用账户服务扣减账户余额->修改订单状态
     * 简单说：下订单->扣库存->减余额->改状态
     */
    //rollbackFor = Exception.class表示对任意异常都进行回滚
    @GlobalTransactional(name = "fsp-tx-order",rollbackFor = Exception.class)
    @Override
    public void create(Order order) {
		...
    }
}
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24

下单 - [http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100(opens new window)](http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100)

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220902/image.5ruzgxtsx9k0.webp)

数据库情况



```sql
SELECT * FROM t_order;
```

1

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220902/image.1qtc9j1zh5og.webp)



```sql
SELECT * FROM t_storage;
```

1

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220902/image.39kalvkrafu0.webp)



```sql
SELECT * FROM t_account;
```

1

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220902/image.5jemsfuxnww0.webp)

还是模拟AccountServiceImpl添加超时，下单后数据库数据并没有任何改变，记录都添加不进来，**达到出异常，数据库回滚的效果**。

## Seata之原理简介

2019年1月份蚂蚁金服和阿里巴巴共同开源的分布式事务解决方案。

Simple Extensible Autonomous Transaction Architecture，简单可扩展自治事务框架。

2020起始，用1.0以后的版本。Alina Gingertail

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220902/image.4kucd3b8uhk0.webp)

分布式事务的执行流程

- TM开启分布式事务(TM向TC注册全局事务记录) ;
- 按业务场景，编排数据库、服务等事务内资源(RM向TC汇报资源准备状态) ;
- TM结束分布式事务，事务一阶段结束(TM通知TC提交/回滚分布式事务) ;
- TC汇总事务信息，决定分布式事务是提交还是回滚；
- TC通知所有RM提交/回滚资源，事务二阶段结束。

**AT模式如何做到对业务的无侵入**

- 是什么

> **前提**
>
> - 基于支持本地 ACID 事务的关系型数据库。
> - Java 应用，通过 JDBC 访问数据库。
>
> **整体机制**
>
> 两阶段提交协议的演变：
>
> - 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源。
> - 二阶段：
>   - 提交异步化，非常快速地完成。
>   - 回滚通过一阶段的回滚日志进行反向补偿。
>
> [link(opens new window)](https://seata.io/zh-cn/docs/overview/what-is-seata.html)

- 一阶段加载

在一阶段，Seata会拦截“业务SQL”

1. 解析SQL语义，找到“业务SQL" 要更新的业务数据，在业务数据被更新前，将其保存成"before image”
2. 执行“业务SQL" 更新业务数据，在业务数据更新之后,
3. 其保存成"after image”，最后生成行锁。

以上操作全部在一个数据库事务内完成, 这样保证了一阶段操作的原子性。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220902/image.3aepx5ztene0.webp)

- 二阶段提交

二阶段如果顺利提交的话，因为"业务SQL"在一阶段已经提交至数据库，所以Seata框架只需将一阶段保存的快照数据和行锁删掉，完成数据清理即可。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220902/image.17voa04kq7ek.webp)

- 二阶段回滚

二阶段如果是回滚的话，Seata 就需要回滚一阶段已经执行的 “业务SQL"，还原业务数据。

回滚方式便是用"before image"还原业务数据；但在还原前要首先要校验脏写，对比“数据库当前业务数据”和"after image"。

如果两份数据完全一致就说明没有脏写， 可以还原业务数据，如果不一致就说明有脏写, 出现脏写就需要转人工处理。

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220902/image.5faqn74q58k0.webp)

补充

![image](https://cdn.jsdelivr.net/gh/xustudyxu/image-hosting1@master/20220902/image.4kxovt5nfhq0.webp)