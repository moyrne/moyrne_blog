---
title: "Etcd 配置"
date: 2022-05-21T20:18:41+08:00
draft: false
toc: false
images:
tags: ["shell"]
---

1. 使用`apt`安装`etcd`
~~~
sudo apt install etcd
~~~
2. 生成证书（参考 https://www.jianshu.com/p/ba0964c41b50）
3. 配置config文件 /etc/etcd/etcd.conf
~~~
# [Member]
ETCD_NAME="worker1"
ETCD_DATA_DIR="/var/lib/etcd/worker"
ETCD_LISTEN_PEER_URLS="http://yourip-1:2380"
ETCD_LISTEN_CLIENT_URLS="http://yourip-1:2379,http://127.0.0.1:2379"

# [Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://yourip-1:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://yourip-1:2379"
ETCD_INITIAL_CLUSTER="worker1=http://yourip-1:2380,worker2=http://yourip-2:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"

# [Security]
ETCD_CERT_FILE="/etc/etcd/cert/server.crt"
ETCD_KEY_FILE="/etc/etcd/cert/server.key"
ETCD_TRUSTED_CA_FILE="/etc/etcd/cert/ca.crt"
ETCD_CLIENT_CERT_AUTH=true
ETCD_PEER_CERT_FILE="/etc/etcd/cert/server.crt"
ETCD_PEER_KEY_FILE="/etc/etcd/cert/server.key"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/cert/ca.crt"
ETCD_PEER_CLIENT_CERT_AUTH=true
~~~
4. 修改service文件 `/lib/systemd/system/etcd.service`
~~~
[Unit]
Description=etcd - highly-available key value store
Documentation=https://github.com/coreos/etcd
Documentation=man:etcd
After=network.target
Wants=network-online.target

[Service]
EnvironmentFile=/etc/etcd/etcd.conf
Type=notify
User=etcd
ExecStart=/usr/bin/etcd --name ${ETCD_NAME} \
  --data-dir=${ETCD_DATA_DIR} \
  --listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
  --listen-client-urls=${ETCD_LISTEN_CLIENT_URLS} \
  --advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
  --initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
  --initial-cluster=${ETCD_INITIAL_CLUSTER} \
  --initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
  --initial-cluster-state=${ETCD_INITIAL_CLUSTER_STATE} \
  --cert-file=${ETCD_CERT_FILE} \
  --key-file=${ETCD_KEY_FILE} \
  --peer-cert-file=${ETCD_PEER_CERT_FILE} \
  --peer-key-file=${ETCD_PEER_KEY_FILE} \
  --trusted-ca-file=${ETCD_TRUSTED_CA_FILE} \
  --client-cert-auth=${ETCD_CLIENT_CERT_AUTH} \
  --peer-client-cert-auth=${ETCD_PEER_CLIENT_CERT_AUTH} \
  --peer-trusted-ca-file=${ETCD_PEER_TRUSTED_CA_FILE}

Restart=on-abnormal
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
Alias=etcd2.service
~~~