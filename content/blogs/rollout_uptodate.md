---
title: "Deployment 水平扩容和滚动更新"
date: 2021-02-13T22:38:54+08:00
tags: ["kubernetes"]
draft: false
---

#### 概念

- Deployment 所管理的 Pod 的 ownerReference 就是 ReplicaSet
- Deployment 会拥有多个ReplicaSet, 不同的 ReplicaSet 代表不同的版本
- 关系图 ![avatar](/images/deployment_replicaset_pod.png)

- Deployment 状态字段
    1. DESIRED：用户期望的 Pod 副本个数（spec.replicas 的值）；
    2. CURRENT：当前处于 Running 状态的 Pod 的个数；
    3. UP-TO-DATE：当前处于最新版本的 Pod 的个数，所谓最新版本指的是 Pod 的 Spec 部分与 Deployment 里 Pod 模板里定义的完全一致；
    4. AVAILABLE：当前已经可用的 Pod 的个数，即：既是 Running 状态，又是最新版本，并且已经处于 Ready（健康检查正确）状态的 Pod 的个数。

- ReplicaSet 状态字段 DESIRED、CURRENT 和 READY，与 Deployment 一致，Deployment 仅多出 UP-TO-DATE 状态

- 滚动更新 RollingUpdateStrategy 在“滚动更新”的过程中永远都会确保至少有 2 个 Pod 处于可用状态，至多只有 4 个 Pod 同时存在于集群中
  旧版本RS的Pod数量递减，新版本RS的Pod数量递增
  ~~~yaml
  apiVersion: 
    apps/v1kind: 
      Deploymentmetadata: 
        name: nginx-deployment 
        labels: 
          app: nginx
  spec:
  ... 
    strategy: 
      type: RollingUpdate 
      rollingUpdate: 
        maxSurge: 1 
        maxUnavailable: 1
  ~~~
  
- 金丝雀发布和蓝绿发布，示例： https://github.com/ContainerSolutions/k8s-deployment-strategies/tree/master/canary

#### 命令

- 扩容

  ~~~shell
  $ kubectl scale deployment nginx-deployment --replicas=4
  ~~~

- 滚动更新

  ~~~shell
  # --record 参数会记录下每次操作执行的命令
  $ kubectl create -f nginx-deployment.yaml --record
  ~~~

- 实时查看 Deployment 状态变化

  ~~~shell
  $ kubectl rollout status deployment/nginx-deployment
  ~~~

- 查看 ReplicaSet

  ~~~shell
  $ kubectl get rs 
  ~~~

- 回滚 (有 --to-version=? 参数可选，回滚到指定版本)
  ~~~shell
  $ kubectl rollout undo deployment/nginx-deployment
  ~~~

- 查看 Deployment 每次的变更
  ~~~shell
  $ kubectl rollout history deployment/nginx-deployment
  ~~~