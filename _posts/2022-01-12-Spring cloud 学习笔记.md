---
layout: post
title: Spring Cloud 学习笔记 （尚硅谷 周阳）
tags: springcloud
categories: springcloud
---

### 一、微服务架构


1.什么是微服务架构

```
是一种架构模式，提倡将单一应用程序划分成一组小的服务，服务之间相互协调、相互配合。

每个服务都运行在独立的进程中，服务与服务间采用轻量级的通信机制互相协作（HTTP协议和RESTful API）

```

2.微服务维度的关键词

```
服务注册和发现  --> Eureka 

服务调用和负载均衡 --> Ribbon、Feign  

服务熔断和降级 --> Hystrix    

服务消息队列

配置中心管理 --> springcloud config   

服务网关  --> zuul

服务监控

全链路追踪

自动化构建部署

服务定时任务调度
```

3.SpringCloud简介

```
是分布式微服务架构的一站式解决方案，是多种微服务架构落地技术的集合体，俗称微服务全家桶
```


4.boot和cloud版本选型

```
boot用的是数字、cloud用的是字母（以英国伦敦地铁站命名）

过时：
1.5.9.RELEASE  Dalston.SR1

当前：
SpringBoot2.X和SpringCloudH版本
2.2.x   Hoxton
2.1.x   Greenwich

查看版本对应的json串：https://start.spring.io/actuator/info

SpringCloud Alibaba

---

最终选型：
cloud Hoxton.SR1
boot 2.2.2.RELEASE
cloud alibaba 2.1.0.RELEASE
java java8
Maven 3.5及以上
Mysql 5.7及以上

```

5.cloud各个组件的停更和替换

```
停更不停用

服务注册中心：Eureka -->  zookeeper 、 consul 、 Nacos
服务调用：
    Ribbon 、 LoadBalancer
    Feign  -->  OpenFeign
服务降级：
    Hystrix --> sentienl
服务网关:
    Zuul  --> gateway   
服务配置
    config --> Nacos
服务总线
    Bus  --> Nacos
    
参考文档：官方文档
```


### 二、微服务搭建

1.父工程搭建


步骤：

```
<!-- 统一管理jar包版本 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.16</druid.version>
        <druid.spring.boot.starter.version>1.1.10</druid.spring.boot.starter.version>
        <spring.boot.version>2.2.2.RELEASE</spring.boot.version>
        <spring.cloud.version>Hoxton.SR1</spring.cloud.version>
        <spring.cloud.alibaba.version>2.1.0.RELEASE</spring.cloud.alibaba.version>
        <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
        <mybatis-spring-boot-starter.version>2.1.1</mybatis-spring-boot-starter.version>
        <hutool-all.version>5.1.0</hutool-all.version>
    </properties>


    <!--依赖管理：子模块继承后，子模块不用写groupId和version(锁定版本)-->
    <dependencyManagement>
        <dependencies>
            <!--spring boot 2.2.2-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud Hoxton.SR1-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--Spring cloud alibaba 2.1.0.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid-spring-boot-starter</artifactId>
                <version>${druid.spring.boot.starter.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis-spring-boot-starter.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

总结：dependencyManagement的作用

```

2.订单服务

- 模块名字：cloud-provider-payment8001

- pom.xml


```

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
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
    <!--mysql-connector-java-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!--jdbc-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
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


```

- application.yml


```
server:
  port: 8001

spring:
  application:
    name: payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource        #数据源
    driver-class-name: com.mysql.jdbc.Driver            #mysql驱动
    url: jdbc:mysql://www.51sou.top:3306/cloud?serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: root
    password: 123456


mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.yejiang.cloud.entities    # 所有Entity别名类所在包
  
```

- 主启动


```

@SpringBootApplication

```

- 业务类

 
```

建表
Entity
dao：
    mybatis --> resources/mapper/*.xml
    dao/接口
service
controller

```

3.消费订单服务



### 三、Eureka服务注册与发现

1、什么是服务注册？

2、Eureka两个组件？


```

Eureka Server提供服务注册服务：
    各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。
EurekaClient通过注册中心进行访问：
    是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。
    如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）

```

3、单机Eureka

EurekaServer 

- pom.xml


```

<!--eureka-server-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

```

- application.yml


```

server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
    #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

```

- 主启动


```

@EnableEurekaServer

```

EurekaClient


- pom.xml


```

<!--eureka-server-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

```

- application.yml


```

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

```

- 主启动


```

@EnableEurekaClient

```



4、集群Eureka

EurekaClient


```

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版

```

EurekaServer


```

server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://eureka7002.com:7002/eureka/
      
      
---


server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://eureka7001.com:7001/eureka/

```

负载均衡

```
1）使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
2）通过在eureka上注册过的微服务名称调用
 
```

5、自我保护机制

```
某个时刻某个微服务不可用了，Eureka不会立即清理，依旧会对该微服务进行保存，属于CAP中的AP

默认开启自我保护的，可以禁止自我保护：
    eureka.server.enable-self-preservation = false
```


### 四、Zookeeper服务注册与发现

1、安装zookeeper

```
1) 下载zookeeper到Centos7中
链接：https://pan.baidu.com/s/1x_c561umrOPxzn7DkIoNQA 
提取码：9999

2) 解压
tar -zxvf zookeeper-3.4.13.tar.gz

3) 修改配置
cd zookeeper-3.4.13/conf
#必须是 zoo.cfg
cp  zoo_sample.cfg  zoo.cfg
vim zoo.cfg
    dataDir=/tmp/zookeeper/data
    dataLogDir=/tmp/zookeeper/log  

4) 创建目录
mkdir /tmp/zookeeper
mkdir /tmp/zookeeper/data
mkdir /tmp/zookeeper/log

5) 启动zk
```

2、创建服务提供者

- pom.xml 


```
<!--SpringBoot整合Zookeeper客户端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <exclusions>
        <!--先排除自带的zookeeper3.5.3-->
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--添加zookeeper3.4.14版本-->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.14</version>
    <exclusions>
        <exclusion>
            <artifactId>slf4j-log4j12</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

- main


```
#该注解用于向使用consul或者zookeeper作为注册中心时注册服务
@EnableDiscoveryClient 
```

- applicaition.yml

```
spring:
  application:
    name: cloud-provideZK-payment
  cloud:
    zookeeper:
      connect-string: www.51sou.top:2181
```

3、创建服务消费者

### 五、Consul服务注册与发现

1、Consul介绍

```
service discovery:DNS或者HTTP接口注册发现
health checking:
key/value storage:
multi-datacenter:无需复杂的配置,即可支持任意数量的区域。
```

2、Consul安装

```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install consul
```

3、Consul使用


