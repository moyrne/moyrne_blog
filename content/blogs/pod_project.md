---
title: "Pod 对象"
date: 2021-02-12T15:07:14+08:00
tags: ["kubernetes"]
draft: false
---

## Volume
- Projected Volume 将预先设置好的数据投射到容器中，
    - Secret
~~~yaml
volumes: 
- name: mysql-cred 
projected: 
  sources: 
  - secret: 
    name: user 
  - secret: 
    name: pass
~~~
    - ConfigMap
~~~
~~~
    - DownwardAPI
~~~
~~~
    - ServiceAccountToken
~~~
~~~
  
## restartPolicy livenessProbe
1. 只要 Pod 的 restartPolicy 指定的策略允许重启异常的容器（比如：Always），那么这个 Pod 就会保持 Running 状态，并进行容器重启。否则，Pod 就会进入 Failed 状态 。 
2. 对于包含多个容器的 Pod，只有它里面所有的容器都进入异常状态后，Pod 才会进入 Failed 状态。在此之前，Pod 都是 Running 状态。此时，Pod 的 READY 字段会显示正常容器的个数，比如：
   ~~~shell
   $ kubectl get pod test-liveness-exec
   NAME          READY STATUS  RESTARTS AGE
   liveness-exec 0/1   Running 1        1m
   ~~~

3. livenessProbe HTTP Or TCP, 多用于Web服务健康检查, Web服务中暴露出健康检查URL
~~~yaml
...
livenessProbe:
     httpGet:
       path: /healthz
       port: 8080
       httpHeaders:
       - name: X-Custom-Header
         value: Awesome
       initialDelaySeconds: 3
       periodSeconds: 3
~~~
~~~yaml
    ...
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
~~~

## PodPreset
PodPreset 里定义的内容，只会在 Pod API 对象被创建之前追加在这个对象本身上，而不会影响任何 Pod 的控制器的定义。

比如，我们现在提交的是一个 nginx-deployment，那么这个 Deployment 对象本身是永远不会被 PodPreset 改变的， 被修改的只是这个 Deployment 创建出来的所有 Pod。