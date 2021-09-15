---
layout: post
title: kubernetes之ServiceAccount
tags: k8s
categories: k8s
---

https://www.cnblogs.com/panwenbin-logs/p/10029834.html

```
Service account是为了方便Pod里面的进程调用Kubernetes API或其他外部服务而设计的。它与User account不同
　　1.User account是为人设计的，而service account则是为Pod中的进程调用Kubernetes API而设计；
　　2.User account是跨namespace的，而service account则是仅局限它所在的namespace；
　　3.每个namespace都会自动创建一个default service account
　　4.Token controller检测service account的创建，并为它们创建secret
　　5.开启ServiceAccount Admission Controller后
       1.每个Pod在创建后都会自动设置spec.serviceAccount为default（除非指定了其他ServiceAccout）
　　　　2.验证Pod引用的service account已经存在，否则拒绝创建
　　　　3.如果Pod没有指定ImagePullSecrets，则把service account的ImagePullSecrets加到Pod中
　　　　4.每个container启动后都会挂载该service account的token和ca.crt到/var/run/secrets/kubernetes.io/serviceaccount/


重要的点：
   <1> 每个namespace都会自动创建一个default 的serviceAccount
   <2> 每个Pod在创建后都会自动设置spec.serviceAccount为default
```


测试创建serviceName

```
kubectl create namespace test0912

```

Kubernetes为所有默认的ServiceAccount授权
```
https://blog.csdn.net/ch717828/article/details/106223056
```