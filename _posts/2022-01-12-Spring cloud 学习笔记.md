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

SpringBoot2.X和SpringCloudH版本

2.2.x   Hoxton
2.1.x   Greenwich

SpringCloud Alibaba

```