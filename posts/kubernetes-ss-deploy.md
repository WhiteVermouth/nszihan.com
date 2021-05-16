---
title: "在Kubernetes中搭建SS、Snell等代理服务"
date: "2020-05-18 01:14:35"
---

## 什么是 Kubernetes

我们都知道`Docker`容器，但是在生产环境中，仅仅有`Docker`还不够。当遇到应用自动化部署、容器伸缩和管理等问题时，我们需要一个容器编排系统。`Kubernetes`就是一个生产级的容器编排系统。

## Kubernetes 基础

`Pod`：最小运行单元，由一个或多个容器组成
`Deployment`：常用的控制器
`Service`：用于服务注册、服务发现，作用域主要为 K8S 集群内部
`Ingress`：提供外部访问入口

## 从哪里获取 Kubernetes 集群

`Kubernetes` 集群可以自己搭建，也可以直接从云服务商购买。自行搭建较为复杂，且运维工作也会很复杂，使用云服务商的`Kubernetes`集群较为方便。

比较有名的`Kubernetes`提供商有：`Azure`、`AWS`、`GCP`、阿里云

上面这几家都是云计算大厂，稳定性更好，但是价格也会很高。

相对便宜的云服务可以选择：`DigitalOcean`、`Linode`

## 编写 Snell 部署所需的资源配置文件

1. 编写 `ConfigMap` 用于存储配置文件

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: snell-server-config
  namespace: stage
  labels:
    app: surge-snell
data:
  snell-server.conf: |-
    [snell-server]
    listen = 0.0.0.0:10000
    psk = password
    obfs = http
```

2. 编写 `Deployment` 和 `Service` 资源配置

```yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: snell-server
  namespace: stage
  labels:
    app: surge-snell
spec:
  replicas: 2
  revisionHistoryLimit: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: snell-server
  template:
    metadata:
      labels:
        app: snell-server
    spec:
      containers:
        - name: snell-server
          image: vermouthx/surge-snell:2.0.1
          command: ["snell-server", "-c", "/etc/snell/snell-server.conf"]
          volumeMounts:
            - name: snell-config
              mountPath: /etc/snell
          resources:
            limits:
              memory: "128Mi"
              cpu: "100m"
          ports:
            - containerPort: 10000
      volumes:
        - name: snell-config
          configMap:
            name: snell-server-config
      restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: snell-service
  namespace: stage
  labels:
    app: surge-snell
spec:
  selector:
    app: snell-server
  ports:
    - protocol: TCP
      port: 10000
      targetPort: 10000
```

3. 使用 `kubectl` 创建上述资源

```shell
kubectl apply -f /path/to/resource
```
