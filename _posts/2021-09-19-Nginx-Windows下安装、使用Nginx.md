---
layout: post
title: Windows下安装、使用Nginx
tags: Nginx
categories: Nginx
---

### 一、windows下Nginx快速入门

1、下载地址

- http://nginx.org/en/download.html 

2、解压目录结构

![](https://muyerj.github.io/images/posts/2021/09/19/nginx-windows.jpg)

3、运行
- 方式一
```
双击 nginx.exe
```

- 方式二

```
cmd到安装目录
输入 nginx.exe 或 start nginx
```

最后再浏览器输入 ： http://localhost:80  == > 即可校验安装成功

4、停止

- 方式一：cmd命令
```
nginx -s stop # 快速停止
nginx -s quit  # 正常停止
nginx -s reload # 重新加载；可以重启其它文件启动的情况
```

- 方式二：taskkill /f /t /im nginx.exe
```
注意！！！
在我自己修改完配置文件后，想重新启动 使用 nginx -s reload 命令后，就一直报以下错误：
nginx: [error] CreateFile() "D: ginx-1.14.2/logs/nginx.pid" failed 

后面继续查，查出命令 ： taskkill /f /t /im nginx.exe 可以直接杀死Nginx进程
```


### 二、配置本地反向代理和解决跨域

自己在本地启动公司前端项目和后端项目，发现前端项目调用后端项目时，老是跨域，所以才有这次windows下Nginx实践，以下是我的配置

**conf/nginx.conf**

```

#user  nobody;
worker_processes  1;



events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    
    server {
        listen       80;
        server_name  localhost;
        location / {  
            proxy_pass http://localhost:9876;
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
            add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
         
            if ($request_method = 'OPTIONS') {
                return 204;
            }
        }
}


}

```

