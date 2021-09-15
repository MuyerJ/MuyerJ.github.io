---
layout: post
title: kubernetes 滚动升级
tags: k8s
categories: k8s
---

```

https://blog.csdn.net/winterfeng123/article/details/109075454

https://blog.csdn.net/yunxiao6/article/details/108445576

https://blog.csdn.net/lxy___/article/details/105815235

批量删除阿里云镜像?
    https://help.aliyun.com/document_detail/72385.html
    
滚动升级？
    1.每次升级都修改镜像tag，但是阿里云镜像仓库sizeMax=300，这个还有实时删除，不可取
    2.
        阿里云镜像仓库最多300个
        阿里云镜像仓库镜像版本 xx 个?
            目前看是没有限制
    3.两种流水线实现滚动升级
        1) yaml实现：需要每次都写版本,但是可能镜像版本会有很多(没有限制)
        2) 页面点击配置：不需要写版本,但是如果deployment管理的pod数量是1时,会导致发布失败,所以当pod数是1时,需要手动扩容


```