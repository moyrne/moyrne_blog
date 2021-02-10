---
title: "Kubernetes入门-基础"
date: 2021-02-06T10:13:46+08:00
draft: false
---

- CPU
  ~~~text
  CGroup 设置cpu可使用时间
  ~~~
- 文件系统隔离
  ~~~text
  Mount NameSpace + mount("none","/tmp","tmpfs",0,"")
  ~~~

- 为 结点 打上污点 禁止其它Pod在结点上启动
  ~~~shell
  kubectl taint nodes node1 foo=bar:NoSchedule
  ~~~

- rook 安装 按顺序执行以下命令
  ~~~shell
  $ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/common.yaml
  $ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml
  
  # 在 apply cluster.yaml 之前, 需要先apply crds.yaml, 否则会报错 no matches for kind "CephCluster" in version "ceph.rook.io/v1"
  $ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/crds.yaml
  
  $ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml
  ~~~

- [20210211] docker 多阶段构建,减小体积（示例）
  ~~~dockerfile
  FROM golang:1.15
  
  ENV GOPROXY=https://goproxy.cn,direct \
      GO111MODULE=on
  
  WORKDIR /app
  
  COPY . .
  
  RUN go build
  
  FROM ubuntu:20.04
  
  ENV PORT=8080
  
  WORKDIR /app
  
  # --from=0 表示从第一阶段中获取 文件
  COPY --from=0 /app .
  
  EXPOSE 8080
  
  ENTRYPOINT ["./beego"]
  ~~~
  
- 设置 volume
  ~~~yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: beego-demo
  spec:
    selector:
      matchLabels:
        app: beego-demo
    replicas: 2
    template:
      metadata:
        labels:
          app: beego-demo
      spec:
        containers:
        - name: beego
          image: master:5000/beego
          ports:
            - containerPort: 8080
          volumeMounts:
            - mountPath: "/app/views"
              name: beego-index
        volumes:
          - name: beego-index
            hostPath:
              path: "/var/data/beego/views"
  ~~~

  还在学习的路上 。。。。。。