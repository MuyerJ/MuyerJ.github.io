---
layout: post
title: kubernetes之Ingress
tags: k8s
categories: k8s
---

### 一、安装步骤

#### 1、安装Ingress Controller

```
kubectl apply -f https://raw.githubusercontent.com/AliyunContainerService/serverless-k8s-examples/master/ingress-nginx/nginx-ingress-controller.yaml

service/nginx-ingress-lb created
configmap/nginx-configuration unchanged
configmap/tcp-services unchanged
configmap/udp-services unchanged
serviceaccount/nginx-ingress-controller created
==> Warning: rbac.authorization.k8s.io/v1beta1 ClusterRole is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRole
clusterrole.rbac.authorization.k8s.io/nginx-ingress-controller created
==> Warning: rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-controller created
deployment.apps/nginx-ingress-controller created



kubectl -n kube-system get pod
NAME                                       READY   STATUS    RESTARTS   AGE
coredns-86b47bc4b5-6vgkg                   1/1     Running   0          22d
coredns-86b47bc4b5-tsgl6                   1/1     Running   0          22d
metrics-server-6668f66875-hbr99            1/1     Running   0          32h
nginx-ingress-controller-f47f45778-8xh8s   1/1     Running   0          42s
nginx-ingress-controller-f47f45778-pmxkb   0/1     Running   0          42s



kubectl -n kube-system get pod
NAME                                       READY   STATUS    RESTARTS   AGE
coredns-86b47bc4b5-6vgkg                   1/1     Running   0          22d
coredns-86b47bc4b5-tsgl6                   1/1     Running   0          22d
metrics-server-6668f66875-hbr99            1/1     Running   0          32h
nginx-ingress-controller-f47f45778-8xh8s   1/1     Running   0          45s
nginx-ingress-controller-f47f45778-pmxkb   1/1     Running   0          45s
kubectl -n kube-system get svc
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
heapster           ClusterIP      172.21.2.212    <none>           80/TCP                       32h
kube-dns           ClusterIP      172.21.10.218   <none>           53/UDP,53/TCP,9153/TCP       22d
metrics-server     ClusterIP      172.21.2.97     <none>           443/TCP                      32h
nginx-ingress-lb   LoadBalancer   172.21.10.202   182.92.242.133   80:31225/TCP,443:31447/TCP   52s

```

nginx-ingress-controller.yaml 清单
```nginx-ingress-controller.yaml
---
apiVersion: v1
kind: Service
metadata:
 name: nginx-ingress-lb
 namespace: kube-system
 labels:
   app: nginx-ingress-lb
 
spec:
 type: LoadBalancer
 externalTrafficPolicy: "Local"
 ports:
 - port: 80
   name: http
   targetPort: 80
 - port: 443
   name: https
   targetPort: 443
 selector:
   app: ingress-nginx
---
apiVersion: v1
kind: ConfigMap
metadata:
 name: nginx-configuration
 namespace: kube-system
 labels:
   app: ingress-nginx
data:
   log-format-upstream: '$remote_addr - [$remote_addr] - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $request_length $request_time [$proxy_upstream_name] $upstream_addr $upstream_response_length $upstream_response_time $upstream_status $req_id $host [$proxy_alternative_upstream_name]'
   proxy-body-size: 20m
   proxy-connect-timeout: "10"
   max-worker-connections: "65536"
   enable-underscores-in-headers: "true"
   reuse-port: "true"
   worker-cpu-affinity: "auto"
   server-tokens: "false"
   ssl-redirect: "false"
   allow-backend-server-header: "true"
   ignore-invalid-headers: "true"
   generate-request-id: "true"
   upstream-keepalive-timeout: "900"
   #forwarded-for-header: "X-Real-IP"
   #compute-full-forwarded-for: "true"
   #hsts: "false"
   #enable-vts-status: "true"
   #use-proxy-protocol: "true"
---
kind: ConfigMap
apiVersion: v1
metadata:
 name: tcp-services
 namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
 name: udp-services
 namespace: kube-system

---
apiVersion: v1
kind: ServiceAccount
metadata:
 name: nginx-ingress-controller
 namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
 name: nginx-ingress-controller
rules:
 - apiGroups:
     - ""
   resources:
     - configmaps
     - endpoints
     - nodes
     - pods
     - secrets
     - namespaces
     - services
   verbs:
     - get
     - list
     - watch
 - apiGroups:
     - "extensions"
     - "networking.k8s.io"
   resources:
     - ingresses
   verbs:
     - get
     - list
     - watch
 - apiGroups:
     - ""
   resources:
     - events
   verbs:
     - create
     - patch
 - apiGroups:
     - "extensions"
     - "networking.k8s.io"
   resources:
     - ingresses/status
   verbs:
     - update
 - apiGroups:
     - ""
   resources:
     - configmaps
   verbs:
     - create
 - apiGroups:
     - ""
   resources:
     - configmaps
   resourceNames:
     - "ingress-controller-leader-nginx"
   verbs:
     - get
     - update
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
 name: nginx-ingress-controller
roleRef:
 apiGroup: rbac.authorization.k8s.io
 kind: ClusterRole
 name: nginx-ingress-controller
subjects:
 - kind: ServiceAccount
   name: nginx-ingress-controller
   namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
 name: nginx-ingress-controller
 labels:
   app: ingress-nginx
 namespace: kube-system
 annotations:
   component.version: "0.30.0"
   component.revision: "2"
spec:
 replicas: 2
 selector:
   matchLabels:
     app: ingress-nginx
 template:
   metadata:
     labels:
       app: ingress-nginx
     annotations:
       prometheus.io/port: "10254"
       prometheus.io/scrape: "true"
   spec:
     securityContext:
       sysctls:
       - name: net.core.somaxconn
         value: "65535"
#       - name: net.ipv4.ip_local_port_range
#         value: "1024 65535"
#       - name: fs.file-max
#         value: "1048576"
#       - name: fs.inotify.max_user_instances
#         value: "16384"
#       - name: fs.inotify.max_queued_events
#         value: "16384"
#       - name: fs.inotify.max_user_watches
#         value: "524288"
#    #tolerations:
     #  - key: node-role.kubernetes.io/master
     #    effect: NoSchedule
     affinity:
       podAntiAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
         - labelSelector:
             matchExpressions:
             - key: app
               operator: In
               values:
                - ingress-nginx
           topologyKey: "kubernetes.io/hostname"
       nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
           - matchExpressions:
             - key: type
               operator: NotIn
               values:
               - virtual-kubelet
             - key: k8s.aliyun.com
               operator: NotIn
               values:
               - "true"
     serviceAccountName: nginx-ingress-controller
     priorityClassName: system-node-critical
     containers:
     - name: nginx-ingress-controller
       image: registry.cn-hangzhou.aliyuncs.com/acs/aliyun-ingress-controller:v0.30.0.2-9597b3685-aliyun
       args:
         - /nginx-ingress-controller
         - --configmap=$(POD_NAMESPACE)/nginx-configuration
         - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
         - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
         - --annotations-prefix=nginx.ingress.kubernetes.io
         - --publish-service=$(POD_NAMESPACE)/nginx-ingress-lb
         - --v=2
       env:
         - name: POD_NAME
           valueFrom:
             fieldRef:
               fieldPath: metadata.name
         - name: POD_NAMESPACE
           valueFrom:
             fieldRef:
               fieldPath: metadata.namespace
       ports:
       - name: http
         containerPort: 80
       - name: https
         containerPort: 443
       resources:
         requests:
           cpu: 100m
           memory: 70Mi
       livenessProbe:
         failureThreshold: 3
         httpGet:
           path: /healthz
           port: 10254
           scheme: HTTP
         initialDelaySeconds: 10
         periodSeconds: 10
         successThreshold: 1
         timeoutSeconds: 10
       readinessProbe:
         failureThreshold: 3
         httpGet:
           path: /healthz
           port: 10254
           scheme: HTTP
         periodSeconds: 10
         successThreshold: 1
         timeoutSeconds: 10
       securityContext:
         capabilities:
             drop:
             - ALL
             add:
             - NET_BIND_SERVICE
         runAsUser: 101
     nodeSelector:
       beta.kubernetes.io/os: linux
```



#### 2、创建服务

```
kubectl apply -f svc.yaml



```

svc.yaml配置如下
```
apiVersion:  apps/v1
kind: Deployment
metadata:
  name: coffee
spec:
  replicas: 2
  selector:
    matchLabels:
      app: coffee
  template:
    metadata:
      labels:
        app: coffee
    spec:
      containers:
      - name: coffee
        image: nginxdemos/hello:plain-text
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: coffee-svc
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: coffee
  clusterIP: None

---
apiVersion:  apps/v1
kind: Deployment
metadata:
  name: tea
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tea
  template:
    metadata:
      labels:
        app: tea
    spec:
      containers:
      - name: tea
        image: nginxdemos/hello:plain-text
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: tea-svc
  labels:
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: tea
  clusterIP: None


```

#### 3、创建Ingress ,配置路由



```
kubectl apply -f ingress.yaml



```


ingress.yaml配置如下
```
apiVersion:  extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: k8s.51sou.top
    http:
      paths:
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
```
