# 搭建kubernetes master高可用集群
搭建高可用集群需要用到的软件：
+ keepalive：通过keepalived vrrp协议生成vip，实现vip在多台master之间漂移，实现高可用
+ haproxy：通过haproxy实现负载均衡，所有访问都通过vip经过haproxy实现负载均衡
+ kubernetes master：三台master实现高可用

## 安装keepalived
所有master节点上都需要安装keepalived，建议直接使用yum安装。
```
yum install keepalived
```
### 配置keepalived
进入/etc/keepalived，修改keepalived.conf配置文件，可参考http://blog.51cto.com/liuzhengwei521/1929589 ，http://lonf.me/2017/02/15/high-availability-Kubernetes-Master/#keepalived-%E9%AA%8C%E8%AF%81<br/>
**模拟MASTER产生故障**
+ 当检测到/etc/keepalived目录下有down文件时，priority减少20，变为80；低于BACKUP的priority；
+ 此时MASTER变成BACKUP，同时执行notify_backup的脚本文件（关闭haproxy）；
+ 同时BACKUP变成MASTER，同时执行notify_master的脚本文件（启动haproxy）；

**模拟MASTER故障恢复**
+ 当删除/etc/keepalived目录下的down文件时，原MASTER的优先级又变为100，高于原BACKUP的priority;
+ 此时原MASTER由BACKUP又抢占成了MASTER,同时执行notify_master的脚本文件（启动haproxy）；
+ 同时原BACKUP由MASTER又变了BACKUP,同时执行notify_backup的脚本文件（关闭haproxy）；

master配置文件
```
! Configuration File for keepalived

global_defs {
   notification_email {
   }
   router_id LVS_DEVEL01
}

vrrp_script check_k8s {
    script "/etc/keepalived/chk_k8s_master.sh"   #当检测到kube-apiserver 6443端口无监听，priority减少20，变为80；低于BACKUP的priority；
    interval 3
    weight -20
}

vrrp_instance VI_1 {
    state MASTER
    interface team1
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
       172.20.0.224/24
    }
    track_script {
        check_k8s
    }
    
    notify_master "/etc/keepalived/k8s_master.sh"  #notify_master:当当前节点成为master时，通知脚本执行任务(一般用于启动某服务，比如nginx,haproxy等)
    notify_backup "/etc/keepalived/k8s_backup.sh"  #notify_backup:当当前节点成为backup时，通知脚本执行任务(一般用于关闭某服务，比如nginx,haproxy等)
}
```
配置chk_k8s_master.sh shell脚本
``` bash
#!/bin/bash

count=`ss -tnl | grep 6443 | wc -l`

if [ $count = 0 ]; then
  exit 1
else
  exit 0
fi
```
配置k8s_master.sh shell脚本
``` bash
#!/bin/bash
systemctl start haproxy
```
配置k8s_backup.sh shell脚本
``` bash
#!/bin/bash
systemctl stop haproxy
```
## 安装haproxy
所有master节点上都需要安装haproxy，建议直接使用yum安装。
```
yum install haproxy
```
进入/etc/haproxy，修改haproxy.cfg文件，可参考http://xstarcd.github.io/wiki/sysadmin/haproxy_confs.html
```
global
    daemon
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    stats socket /var/lib/haproxy/stats
    log 127.0.0.1 local0 err
    log 127.0.0.1 local1 warning

defaults
    mode http
    log global
    option tcplog
    retries 3               
    timeout connect 5000ms
    timeout client 10000ms
    timeout server 50000ms

listen k8s_server_6443
    bind 172.20.0.224:9443
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    server k8s-master-1 172.20.0.113:6443  check
    server k8s-master-2 172.20.0.114:6443  check
    server k8s-master-3 172.20.0.115:6443  check

listen k8s_server_8080
    bind 172.20.0.224:9080
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    server k8s-master-1 172.20.0.113:8080  check
    server k8s-master-2 172.20.0.114:8080  check
    server k8s-master-3 172.20.0.115:8080  check
    
listen stats
    bind *:8086
    stats refresh 30s
    stats uri /stats
    stats realm HAProxy\ Stats
    stats auth admin:admin
```

## 配置和启动 kube-apiserver

**创建 kube-apiserver的service配置文件**

service配置文件`vi /usr/lib/systemd/system/kube-apiserver.service`内容：

```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
User=root
ExecStart=/usr/local/bin/kube-apiserver \
  --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds,NodeRestriction \
  --apiserver-count=3 \
  --advertise-address=172.20.0.113 \
  --allow-privileged=true \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/var/lib/audit.log \
  --authorization-mode=Node,RBAC \
  --bind-address=0.0.0.0 \
  --client-ca-file=/etc/kubernetes/ssl/ca.pem \
  --enable-swagger-ui=true \
  --etcd-cafile=/etc/kubernetes/ssl/ca.pem \
  --etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem \
  --etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem \
  --etcd-servers=https://172.20.0.113:2379,https://172.20.0.114:2379,https://172.20.0.115:2379 \
  --event-ttl=1h \
  --kubelet-https=true \
  --insecure-bind-address=0.0.0.0 \
  --runtime-config=rbac.authorization.k8s.io/v1 \
  --service-account-key-file=/etc/kubernetes/ssl/ca-key.pem \
  --service-cluster-ip-range=10.254.0.0/16 \
  --service-node-port-range=30000-32000 \
  --tls-cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --tls-private-key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  --enable-bootstrap-token-auth \
  --token-auth-file=/etc/kubernetes/token.csv \
  --v=2
Restart=on-failure
RestartSec=5
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
+ `--bind-address` 和 `--insecure-bind-address` 必须设置为 `0.0.0.0`
+ `--apiserver-count` 默认为1，这配置三台master，所以设置为3

**启动kube-apiserver**

``` bash
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl start kube-apiserver
systemctl status kube-apiserver
```

## 配置和启动 kube-controller-manager

**创建 kube-controller-manager的serivce配置文件**

文件路径`vi /usr/lib/systemd/system/kube-controller-manager.service`

```
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
  --address=127.0.0.1 \
  --master=http://172.20.0.224:9080 \
  --allocate-node-cidrs=true \
  --service-cluster-ip-range=10.254.0.0/16 \
  --cluster-name=kubernetes \
  --cluster-cidr=10.254.0.0/16 \
  --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem \
  --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem \
  --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem \
  --root-ca-file=/etc/kubernetes/ssl/ca.pem \
  --leader-elect=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
+ `--service-cluster-ip-range` 参数指定 Cluster 中 Service 的CIDR范围，该网络在各 Node 间必须路由不可达，必须和 kube-apiserver 中的参数一致；
+ `--cluster-signing-*` 指定的证书和私钥文件用来签名为 TLS BootStrap 创建的证书和私钥；
+ `--root-ca-file` 用来对 kube-apiserver 证书进行校验，**指定该参数后，才会在Pod 容器的 ServiceAccount 中放置该 CA 证书文件**；
+ `--address` 值必须为 `127.0.0.1`，因为当前 kube-apiserver 期望 scheduler 和 controller-manager 在同一台机器
+ `--master` 值必须为 vip地址：`http://172.20.0.224:9080`

### 启动 kube-controller-manager

``` bash
systemctl daemon-reload
systemctl enable kube-controller-manager
systemctl start kube-controller-manager
```

## 配置和启动 kube-scheduler

**创建 kube-scheduler的serivce配置文件**

文件路径`vi /usr/lib/systemd/system/kube-scheduler.service`。

```
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --address=127.0.0.1 \
  --master=http://172.20.0.224:9080 \
  --leader-elect=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
+ `--address` 值必须为 `127.0.0.1`，因为当前 kube-apiserver 期望 scheduler 和 controller-manager 在同一台机器；
+ `--master` 值必须为 vip地址：`http://172.20.0.224:9080`
### 启动 kube-scheduler

``` bash
systemctl daemon-reload
systemctl enable kube-scheduler
systemctl start kube-scheduler
```

## 验证 master 节点功能

``` bash
$ kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
etcd-1               Healthy   {"health": "true"}   
etcd-2               Healthy   {"health": "true"}   
```











