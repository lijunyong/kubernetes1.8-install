# 配置master节点

## 
kubelet 启动时向 kube-apiserver 发送 TLS bootstrapping 请求，需要先将 bootstrap token 文件中的 kubelet-bootstrap 用户赋予 system:node-bootstrapper 角色，然后 kubelet 才有权限创建认证请求(certificatesigningrequests)。

```
# 先创建认证请求
# user 为 master 中 token.csv 文件里配置的用户
# 只需创建一次就可以

kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap

```

## kubernetes master 节点包含的组件：

+ kube-apiserver
+ kube-scheduler
+ kube-controller-manager

目前这三个组件需要部署在同一台机器上。

+ `kube-scheduler`、`kube-controller-manager` 和 `kube-apiserver` 三者的功能紧密相关；
+ 同时只能有一个 `kube-scheduler`、`kube-controller-manager` 进程处于工作状态，如果运行多个，则需要通过选举产生一个 leader；

**注**：

- 暂时未实现master节点的高可用
- master节点上没有部署flannel网络插件，如果想要在master节点上也能访问ClusterIP，请参考下一节[部署node节点](node-installation.md)中的配置Flanneld部分。

## TLS 证书文件

以下`pem`证书文件我们在[创建TLS证书和秘钥](create-tls-and-secret-key.md)这一步中已经创建过了，`token.csv`文件在[创建kubeconfig文件](create-kubeconfig.md)的时候创建。我们再检查一下。

``` bash
$ ls /etc/kubernetes/ssl
admin-key.pem  admin.pem  ca-key.pem  ca.pem  kube-proxy-key.pem  kube-proxy.pem  kubernetes-key.pem  kubernetes.pem
```

# 下载最新版本的二进制文件

``` shell
wget https://dl.k8s.io/v1.8.0/kubernetes-server-linux-amd64.tar.gz

tar -xzvf kubernetes-server-linux-amd64.tar.gz

cd kubernetes

cp -r server/bin/{kube-apiserver,kube-controller-manager,kube-scheduler,kubectl,kube-proxy,kubelet} /usr/local/bin/
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
  --etcd-cafile=/etc/kubernetes/ssl/ca.pem \
  --etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem \
  --etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem \
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

[Install]
WantedBy=multi-user.target
```

**启动kube-apiserver**

``` bash
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl start kube-apiserver
systemctl status kube-apiserver
```
