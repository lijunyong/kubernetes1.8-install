# 安装etcd集群


需要为 etcd 集群创建加密通信的 TLS 证书，这里复用以前创建的 kubernetes 证书

``` bash
cp ca.pem kubernetes-key.pem kubernetes.pem /etc/kubernetes/ssl
```

+ kubernetes 证书的 `hosts` 字段列表中包含上面三台机器的 IP，否则后续证书校验会失败；

## 下载二进制文件

到 `https://github.com/coreos/etcd/releases` 页面下载最新版本的二进制文件

``` bash
wget https://github.com/coreos/etcd/releases/download/v3.2.8/etcd-v3.2.8-linux-amd64.tar.gz
tar -xvf etcd-v3.2.8-linux-amd64.tar.gz
mv etcd-v3.2.8-linux-amd64/etcd* /usr/local/bin
```

## 创建 etcd 的 systemd unit 文件
```
[root@localhost ~]# vi /usr/lib/systemd/system/etcd.service
```
创建etcd工作目录：mkdir /var/lib/etcd/
```
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/local/bin/etcd 
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
+ 创建ectd配置文件

```
[root@localhost ~]# vi /etc/etcd/etcd.conf

ETCD_NAME=etcd1
ETCD_DATA_DIR="/var/lib/etcd/"
ETCD_LISTEN_PEER_URLS="https://172.20.0.113:2380"
ETCD_LISTEN_CLIENT_URLS="https://172.20.0.113:2379,https://172.20.0.113:4001"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://172.20.0.113:2380"
ETCD_INITIAL_CLUSTER="etcd1=https://172.20.0.113:2380,etcd2=https://172.20.0.114:2380,etcd3=https://172.20.0.115:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="k8s-etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://172.20.0.113:2379,https://172.20.0.113:4001"

#[security]
ETCD_CERT_FILE="/etc/kubernetes/ssl/kubernetes.pem"
ETCD_KEY_FILE="/etc/kubernetes/ssl/kubernetes-key.pem"
ETCD_TRUSTED_CA_FILE="/etc/kubernetes/ssl/ca.pem"
#ETCD_CLIENT_CERT_AUTH="true"
ETCD_PEER_CERT_FILE="/etc/kubernetes/ssl/kubernetes.pem"
ETCD_PEER_KEY_FILE="/etc/kubernetes/ssl/kubernetes-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/etc/kubernetes/ssl/ca.pem"
#ETCD_PEER_CLIENT_CERT_AUTH="true"

```
注意：配置文件只是其中一台机器的，部署到另一台机器时，记得更改ip地址

## 启动 etcd 服务

``` bash
systemctl daemon-reload
systemctl enable etcd
systemctl start etcd
systemctl status etcd
```

## 验证服务

在任一 kubernetes master 机器上执行如下命令：

``` bash
[root@localhost ~]# etcdctl --endpoints=https://172.20.0.113:2379 \
   --ca-file=/etc/kubernetes/ssl/ca.pem \
   --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
   --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
   cluster-health
member 58a6194023ba1810 is healthy: got healthy result from https://172.20.0.115:2379
member 7a5dcb96fe62cea2 is healthy: got healthy result from https://172.20.0.113:2379
member b918c49d7706fa2a is healthy: got healthy result from https://172.20.0.114:2379
cluster is healthy
[root@localhost ~]# 

```
