# etcd备份、恢复及数据迁移
现etcd跟kubernetes共用一套证书，而在证书的制作过程中hosts[]导致etcd新增节点无法加入现有集群中：
```
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "172.20.0.112",
      "172.20.0.113",
      "172.20.0.114",
      "172.20.0.115",
      "10.254.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
```
针对现有情况，为etcd重新制作一套证书。

## 创建 CA (Certificate Authority)
```
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF
```
**创建 CA 证书签名请求**

创建 `ca-csr.json`  文件，内容如下：

``` json
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```
**生成 CA 证书和私钥**

``` bash
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
$ ls ca*
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
```
**生成 CA 证书和私钥**

``` bash
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
$ ls ca*
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
```

## 创建 kubernetes 证书

创建 kubernetes 证书签名请求文件 `kubernetes-csr.json`：

``` json
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "172.20.0.112",
      "172.20.0.113",
      "172.20.0.114",
      "172.20.0.115",
      "0.0.0.0"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
```
+ 如果 hosts 不能为空，否则报错`cannot get the version of member 7a5dcb96fe62cea2 (Get https://172.20.0.113:2380/version: x509: cannot validate certificate for 172.20.0.113 because it doesn't contain any IP`

**生成 etcd 证书和私钥**

``` bash
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
$ ls kubernetes*
kubernetes.csr  kubernetes-csr.json  kubernetes-key.pem  kubernetes.pem
```
## 分发证书

将生成的证书和秘钥文件（后缀名为`.pem`）拷贝到所有机器的 `/etc/kubernetes/etcd` 目录下备用；

``` bash
mkdir -p /etc/kubernetes/etcd
cp *.pem /etc/kubernetes/etcd
```

## 替换etcd证书
```
[root@node2 etcd]# vi /etc/etcd/etcd.conf
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
改成
```
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
ETCD_CERT_FILE="/etc/kubernetes/etcd/kubernetes.pem"
ETCD_KEY_FILE="/etc/kubernetes/etcd/kubernetes-key.pem"
ETCD_TRUSTED_CA_FILE="/etc/kubernetes/etcd/ca.pem"
#ETCD_CLIENT_CERT_AUTH="true"
ETCD_PEER_CERT_FILE="/etc/kubernetes/etcd/kubernetes.pem"
ETCD_PEER_KEY_FILE="/etc/kubernetes/etcd/kubernetes-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/etc/kubernetes/etcd/ca.pem"
#ETCD_PEER_CLIENT_CERT_AUTH="true"
```
重启etcd服务
```
[root@node2 ~]# systemctl restart etcd
[root@node2 etcd]# etcdctl --endpoints=https://172.20.0.113:2379,https://172.20.0.114:2379,https://172.20.0.115:2379   --ca-file=/etc/kubernetes/etcd/ca.pem   --cert-file=/etc/kubernetes/etcd/kubernetes.pem   --key-file=/etc/kubernetes/etcd/kubernetes-key.pem   cluster-health
member 58a6194023ba1810 is healthy: got healthy result from https://172.20.0.115:2379
member 7a5dcb96fe62cea2 is healthy: got healthy result from https://172.20.0.113:2379
member b918c49d7706fa2a is healthy: got healthy result from https://172.20.0.114:2379
cluster is healthy

```

## 替换flanneld证书
```
[root@node2 ~]# vi /etc/sysconfig/flanneld
# Flanneld configuration options

# etcd url location.  Point this to the server where etcd runs
ETCD_ENDPOINTS="https://172.20.0.113:2379,https://172.20.0.114:2379,https://172.20.0.115:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
ETCD_PREFIX="/kube-centos/network"

# Any additional options that you want to pass
FLANNEL_OPTIONS="-etcd-cafile=/etc/kubernetes/ssl/ca.pem -etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem -etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem"
```
## 替换kubernetes master证书
```
[root@node2 etcd]# vi /usr/lib/systemd/system/kube-apiserver.service 
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
User=root
ExecStart=/usr/local/bin/kube-apiserver \
  --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook \
  --advertise-address=172.20.0.113 \
  --allow-privileged=true \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/var/lib/audit.log \
  --authorization-mode=Node,RBAC \
  --bind-address=172.20.0.113 \
  --client-ca-file=/etc/kubernetes/ssl/ca.pem \
  --enable-swagger-ui=true \
  --etcd-cafile=/etc/kubernetes/etcd/ca.pem \
  --etcd-certfile=/etc/kubernetes/etcd/kubernetes.pem \
  --etcd-keyfile=/etc/kubernetes/etcd/kubernetes-key.pem \
  --etcd-servers=https://172.20.0.113:2379,https://172.20.0.114:2379,https://172.20.0.115:2379 \
  --event-ttl=1h \
  --kubelet-https=true \
  --insecure-bind-address=172.20.0.113 \
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

```
### 重启修改后的服务
```
[root@node2 etcd]# systemctl daemon-reload
[root@node2 etcd]# systemctl restart flanneld
[root@node2 etcd]# systemctl restart kube-apiserver
[root@node2 etcd]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
etcd-2               Healthy   {"health": "true"}   
etcd-1               Healthy   {"health": "true"}   

```
## etcd集群新增节点
### 新增节点配置文件
```
[root@localhost ~]# mkdir /var/lib/etcd/
[root@localhost ~]# cat /etc/etcd/etcd.conf 
ETCD_NAME=etcd4
ETCD_DATA_DIR="/var/lib/etcd/"
ETCD_LISTEN_PEER_URLS="https://172.20.0.112:2380"
ETCD_LISTEN_CLIENT_URLS="https://172.20.0.112:2379,https://172.20.0.112:4001"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://172.20.0.112:2380"
ETCD_INITIAL_CLUSTER="etcd1=https://172.20.0.113:2380,etcd2=https://172.20.0.114:2380,etcd3=https://172.20.0.115:2380,etcd4=https://172.20.0.112:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
ETCD_INITIAL_CLUSTER_TOKEN="k8s-etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://172.20.0.112:2379,https://172.20.0.112:4001"

#[security]
ETCD_CERT_FILE="/etc/kubernetes/etcd/kubernetes.pem"
ETCD_KEY_FILE="/etc/kubernetes/etcd/kubernetes-key.pem"
ETCD_TRUSTED_CA_FILE="/etc/kubernetes/etcd/ca.pem"
#ETCD_CLIENT_CERT_AUTH="true"
ETCD_PEER_CERT_FILE="/etc/kubernetes/etcd/kubernetes.pem"
ETCD_PEER_KEY_FILE="/etc/kubernetes/etcd/kubernetes-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/etc/kubernetes/etcd/ca.pem"
#ETCD_PEER_CLIENT_CERT_AUTH="true"


[root@localhost ~]# systemctl daemon-reload
```
### 在集群节点执行添加命令
```
[root@node2 bin]# etcdctl --endpoints=https://172.20.0.113:2379,https://172.20.0.114:2379,https://172.20.0.115:2379   --ca-file=/etc/kubernetes/etcd/ca.pem   --cert-file=/etc/kubernetes/etcd/kubernetes.pem   --key-file=/etc/kubernetes/etcd/kubernetes-key.pem   member add etcd4 https://172.20.0.112:2380
Added member named etcd4 with ID c3ef7c1eae5b387a to cluster

ETCD_NAME="etcd4"
ETCD_INITIAL_CLUSTER="etcd3=https://172.20.0.115:2380,etcd1=https://172.20.0.113:2380,etcd2=https://172.20.0.114:2380,etcd4=https://172.20.0.112:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"

```
### 在新增节点启动etcd服务
```
[root@localhost etcd]# systemctl start etcd
```
### 查看集群健康状况

```
[root@node2 bin]# etcdctl --endpoints=https://172.20.0.113:2379,https://172.20.0.114:2379,https://172.20.0.115:2379   --ca-file=/etc/kubernetes/etcd/ca.pem   --cert-file=/etc/kubernetes/etcd/kubernetes.pem   --key-file=/etc/kubernetes/etcd/kubernetes-key.pem   cluster-health
member 58a6194023ba1810 is healthy: got healthy result from https://172.20.0.115:2379
member 7a5dcb96fe62cea2 is healthy: got healthy result from https://172.20.0.113:2379
member b918c49d7706fa2a is healthy: got healthy result from https://172.20.0.114:2379
member c3ef7c1eae5b387a is healthy: got healthy result from https://172.20.0.112:2379
cluster is healthy

```





