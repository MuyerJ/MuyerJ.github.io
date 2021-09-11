---
layout: post
title: kubernetes权威指南学习笔记
tags: k8s
categories: k8s
---

# 1、k8s入门

### 1.1、k8s是什么

### 1.2、从一个简单例子开始

### 1.3、k8s的基本概念和术语

Replication Controller

```
	1、定义了一个期望的场景，即声明某种Pod的副本数量在任意时刻都符合某个预期值
	2、组成：◎ Pod期待的副本数量。◎ 用于筛选目标Pod的Label Selector。◎ 当Pod的副本数量小于预期数量时，用于创建新Pod的Pod模板（template）
	3、运行过程：
		定义了一个RC并将其提交到Kubernetes集群中后，Master上的ControllerManager组件就得到通知，定期巡检系统中当前存活的目标Pod，
		并确保目标Pod实例的数量刚好等于此RC的期望值，如果有过多的Pod副本在运行，系统就会停掉一些Pod，否则系统会再自动创建一些Pod
	4、通过修改RC的副本数量，来实现Pod的动态缩放（Scaling）
	5、RC的删除：
		删除RC并不会影响通过该RC已创建好的Pod。为了删除所有Pod，可以设置replicas的值为0，然后更新该RC。
		另外，kubectl提供了stop和delete命令来一次性删除RC和RC控制的全部Pod。
	6、滚动升级（Rolling Update）
		通常会使用一个新的容器镜像版本替代旧版本
		在整个升级过程中此消彼长
```

Replica Set 

```
	Kubernetes 1.2中，官方解释其为“下一代的RC”
	Replica Set与RC当前的唯一区别是，Replica Sets支持基于集合的Label selector（Set-based selector），而RC只支持基于等式的Label Selector
	当前很少单独使用Replica Set，它主要被Deployment这个更高层的资源对象所使用，从而形成一整套Pod创建、删除、更新的编排机制。
		我们在使用Deployment时，无须关心它是如何创建和维护Replica Set的，这一切都是自动发生的。
	总结RC:
		◎ 在大多数情况下，我们通过定义一个RC实现Pod的创建及副本数量的自动控制。
		◎ 在RC里包括完整的Pod定义模板。
		◎ RC通过Label Selector机制实现对Pod副本的自动控制。
		◎ 通过改变RC里的Pod副本数量，可以实现Pod的扩容或缩容。
		◎ 通过改变RC里Pod模板中的镜像版本，可以实现Pod的滚动升级。
```

Deployment 

```
	1、简介：
		Kubernetes在1.2版本中引入的新概念，用于更好地解决Pod的编排问题,在内部使用了Replica Set来实现目的从Deployment的作用与目的、YAML定义，还是从它的具体命令行操作来看，我们都可以把它看作RC的一次升级，两者的相似度超过90%。
	2、Deployment的典型使用场景
	
```	
	
HPA (Horizontal Pod Autoscaler，Pod横向自动扩容) :

```
	简介：通过手工执行kubectl scale命令，我们可以实现Pod扩容或缩容,分布式系统要能够根据当前负载的变化自动触发水平扩容或缩容,就需要HPA了
	原理：通过追踪分析指定RC控制的所有目标Pod的负载变化情况，来确定是否需要有针对性地调整目标Pod的副本数量，这是HPA的实现原理
	指标：
		 CPUUtilizationPercentage：目标Pod所有副本自身的CPU利用率的平均值
		 应用程序自定义的度量指标，比如服务在每秒内的相应请求数（TPS或QPS）		 
```

StatefulSet：

```
	Pod的管理对象RC、Deployment、DaemonSet和Job都面向无状态的服务
	但是很多服务是有状态的，特别是一些复杂的中间件集群，例如MySQL集群、MongoDB集群、Akka集群、ZooKeeper集群等：
		每个节点都有固定的身份ID，通过这个ID，集群中的成员可以相互发现并通信。
	StatefulSet除了要与PV卷捆绑使用以存储Pod的状态数据，还要与HeadlessService配合使用，Headless Service与普通Service的关键区别在于，它没有Cluster IP
```

service

```
	一、概述：
	1、每个Service其实就是我们经常提起的微服务架构中的一个微服务
	2、Pod、RC与Service的逻辑关系：
	每个Pod都提供了一个独立的Endpoint（Pod IP+ContainerPort）以被客户端访问
	部署一个负载均衡器（软件或硬件），为这组Pod开启一个对外的服务端口如8000端口，并且将这些Pod的Endpoint列表加入8000端口的转发列表，客户端就可以通过负载均衡器的对外IP地址+服务端口来访问此服务
	3、service yaml的定义
		metadata.name
		spec.ports:-port/-targetPort
		service支持多个endpoint
	二、k8s服务发现：
		大部分分布式系统：提供特定的API接口来实现服务发现功能
		k8s Service都有唯一的Cluster IP及唯一的名称 ，通过Service的名称找到对应的Cluster IP。
		如何实现的？Add-On增值包引入了DNS系统，把服务名作为DNS域名，这样程序就可以直接使用服务名来建立通信连接了
		
	三、外部系统访问Service的问题
		Kubernetes里的3种IP：NodeIp、PodIp、ClusterIp
		如何服务符合暴露给外界？
		
```

job
	
volume
```
            1.pod里的container共享文件目录
            2.和docker的volume不等价
            3.用法：
                template.spec.values
                emptyDir
                hostPath
``` 

# 2、安装配置指南

# 3、深入掌握Pod

### 3.1、Pod定义详解

```yaml
apiVersion: v1        　　          #必选，版本号，例如v1,版本号必须可以用 kubectl api-versions 查询到 .
kind: Pod       　　　　　　         #必选，Pod
metadata:       　　　　　　         #必选，元数据
  name: string        　　          #必选，Pod名称
  namespace: string     　　        #必选，Pod所属的命名空间,默认为"default"
  labels:       　　　　　　          #自定义标签
  - name: string      　          #自定义标签名字
  annotations:        　　                 #自定义注释列表
  - name: string
spec:         　　　　　　　            #必选，Pod中容器的详细定义
  containers:       　　　　            #必选，Pod中容器列表
  - name: string      　　                #必选，容器名称,需符合RFC 1035规范
    image: string     　　                #必选，容器的镜像名称
    imagePullPolicy: [ Always|Never|IfNotPresent ]  #获取镜像的策略 Alawys表示下载镜像 IfnotPresent表示优先使用本地镜像,否则下载镜像，Nerver表示仅使用本地镜像
    command: [string]     　　        #容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]      　　             #容器的启动命令参数列表
    workingDir: string                     #容器的工作目录
    volumeMounts:     　　　　        #挂载到容器内部的存储卷配置
    - name: string      　　　        #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string                 #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean                 #是否为只读模式
    ports:        　　　　　　        #需要暴露的端口库号列表
    - name: string      　　　        #端口的名称
      containerPort: int                #容器需要监听的端口号
      hostPort: int     　　             #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string                  #端口协议，支持TCP和UDP，默认TCP
    env:        　　　　　　            #容器运行前需设置的环境变量列表
    - name: string      　　            #环境变量名称
      value: string     　　            #环境变量的值
    resources:        　　                #资源限制和请求的设置
      limits:       　　　　            #资源限制的设置
        cpu: string     　　            #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string                  #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests:       　　                #资源请求的设置
        cpu: string     　　            #Cpu请求，容器启动的初始可用数量
        memory: string                    #内存请求,容器启动的初始可用数量
    livenessProbe:      　　            #对Pod内各容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
      exec:       　　　　　　        #对Pod容器内检查方式设置为exec方式
        command: [string]               #exec方式需要制定的命令或脚本
      httpGet:        　　　　        #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:      　　　　　　#对Pod内个容器健康检查方式设置为tcpSocket方式
        port: number
        initialDelaySeconds: 0       #容器启动完成后首次探测的时间，单位为秒
        timeoutSeconds: 0    　　    #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
        periodSeconds: 0     　　    #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
        successThreshold: 0
        failureThreshold: 0
        securityContext:
          privileged: false
    restartPolicy: [Always | Never | OnFailure] #Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
    nodeSelector: obeject   　　    #设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
    imagePullSecrets:     　　　　#Pull镜像时使用的secret名称，以key：secretkey格式指定
    - name: string
    hostNetwork: false      　　    #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
    volumes:        　　　　　　    #在该pod上定义共享存储卷列表
    - name: string     　　 　　    #共享存储卷名称 （volumes类型有很多种）
      emptyDir: {}      　　　　    #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
      hostPath: string      　　    #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
        path: string      　　        #Pod所在宿主机的目录，将被用于同期中mount的目录
      secret:       　　　　　　    #类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
        scretname: string
        items:
        - key: string
          path: string
      configMap:      　　　　            #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
        name: string
        items:
        - key: string
          path: string
```



```

version:
    String ,必填,版本号,例如V1

---
kind:
    String,必填,资源类型,例如pod

---
metadata:
    Object,必填,元数据
    
metadata.name:
    String,必填,资源的名称,命名规范符合RFC1035规范
    
metadata.namespace:
    String,必填,资源所在的命名空间,默认空间是default

metadata.labels[]
metadata.annotation[]
    List,自定义标签/注解列表

---
```