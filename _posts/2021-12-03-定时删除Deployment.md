---
layout: post
title: 定时删除Deployment
tags: k8s
categories: k8s
---

- 目的: 测试环境阿里云k8s服务在晚上凌晨期间是无人使用的，可以释放掉解约钱

## 创建定时删除deploymentJob步骤

### 1、安装crontab
### 2、创建sh文件，并且赋予权限  

https://www.cnblogs.com/zhuyeshen/p/12073618.html

chmod 755 deleteJob.sh

chmod 755 createJob.sh

---deleteJob.sh
```

#!/bin/bash

time=`date +%Y-%m-%d-%H-%M`
echo $time$"*****开始执行*****  k8s删除deployment"
list=$(/usr/local/bin/kubectl delete deployments --all -n test20211101)
echo "$list"
result=$(/usr/local/bin/kubectl delete deployments --all -n test20211101)
echo "$result"
echo "*****执行结束*****"
```

---createJob.sh

```
#!/bin/bash
time=`date +%Y-%m-%d-%H-%M`
echo $time$"*****开始执行*****  创建xxl-job"
/usr/local/bin/kubectl apply -f /data/k8s/xxl-job.yaml
```

### 3、配置定时任务（千万别动其他配置）

vim /etc/crontab

30 21 * * * root /data/k8s/deleteJob.sh >> /root/log.txt

30 8 * * * root /data/k8s/createJob.sh  >> /root/log.txt
