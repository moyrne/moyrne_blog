---
title: "Kubernetes入门-基础"
date: 2021-02-06T10:13:46+08:00
tags: ["kubernetes"]
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
  
  COPY kubernetes .
  
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
  
- nodeSelector 与 nodeAffinity
  - nodeAffinity 支持更丰富的语义，如：operator
  ~~~yaml
  nodeSelector: 
    name: <Node名字>
  ~~~
  ~~~yaml
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: metadata.name
            operator: In
            values:
            - node-geektime
  ~~~


- xtrabackup 无法下载
  ~~~shell
  docker pull ist0ne/xtrabackup
  docker tag ist0ne/xtrabackup:latest gcr.io/google-samples/xtrabackup:1.0
  ~~~

- DaemonSet
  - DaemonSet具有Toleration字段,用于忽略节点上的某些污点
  ~~~yaml
  # k8s项目中，当一个节点的网络插件尚未安装时，这个节点就会被自动加上名为node.kubernetes.io/network-unavailable的“污点”
  # 例如master节点上也会有 node-role.kubernetes.io/master 这样的污点
  template:
    metadata:
      labels:
        name: network-plugin-agent
    spec:
      tolerations:
      - key: node.kubernetes.io/network-unavailable
        operator: Exists
        effect: NoSchedule
  ~~~
  - DaemonSet 与 DeploymentSet 一样拥有滚动更新的能力，但与 DeploymentSet 通过 ReplicaSet 管理不同版本的区别是， DaemonSet 使用的是 ControllerRevision 对象（StatefulSet 同样如此）。

- Job 与 CronJob
  - 像 Deployment ，线上业务时才会使用，但实际上还有离线业务，此时就可以使用 Job。
  - 定时任务则是 CronJob，定时任务可能存在上一次任务未完成，下一次任务已经开始执行的情况，可以通过 spec.concurrencyPolicy 进行配置。
    1. concurrencyPolicy=Allow，这也是默认情况，这意味着这些 Job 可以同时存在； 
    2. concurrencyPolicy=Forbid，这意味着不会创建新的 Pod，该创建周期被跳过；
    3. concurrencyPolicy=Replace，这意味着新产生的 Job 会替换旧的、没有执行完的 Job。
  ~~~yaml
  # job
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: pi
  spec:
    parallelism: 2
    completions: 4
    template:
      spec:
        containers:
        - name: pi
          image: resouer/ubuntu-bc
          command: ["sh", "-c", "echo 'scale=5000; 4*a(1)' | bc -l "]
        restartPolicy: Never
    backoffLimit: 4
  ~~~
  ~~~yaml
  # cronjob
  apiVersion: batch/v1beta1
  kind: CronJob
  metadata:
    name: hello
  spec:
    schedule: "*/1 * * * *"
    jobTemplate:
      spec:
        template:
          spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
  ~~~
  


  还在学习的路上 。。。。。。