# 部署node节点

Kubernetes node节点包含如下组件：

+ Flanneld：参考jimmysong的文章[Kubernetes基于Flannel的网络配置](https://jimmysong.io/posts/kubernetes-network-config/)，之前没有配置TLS，现在需要在service配置文件中增加TLS配置，安装过程请参考[安装flannel网络插件](7、安装配置flannel.md)。
+ Docker：docker的安装很简单，这里也不说了，但是需要注意docker的配置。
+ kubelet：直接用二进制文件安装
+ kube-proxy：直接用二进制文件安装

**注意**：每台 node 上都需要安装 flannel，master 节点上可以不安装。

**步骤简介**

1. 确认在上一步中我们安装配置的网络插件flannel已启动且运行正常
2. 安装配置docker后启动
3. 安装配置kubelet、kube-proxy后启动
4. 验证

## 安装和配置kubelet

Kubernetes 1.8开始要求关闭系统的Swap，如果不关闭，默认配置下kubelet将无法启动。可以通过kubelet的启动参数--fail-swap-on=false更改这个限制。 我们这里关闭系统的Swap:
```
swapoff -a
```
修改 /etc/fstab 文件，注释掉 SWAP 的自动挂载，使用free -m确认swap已经关闭。

swappiness参数调整，修改/etc/sysctl.d/k8s.conf添加下面一行：
```
vm.swappiness=0
```
执行sysctl -p /etc/sysctl.d/k8s.conf使修改生效。

### 创建kubelet的service配置文件

创建kubelet存储文件

``` bash
[root@localhost bin]# mkdir /var/lib/kubelet
```

文件位置`vi /usr/lib/systemd/system/kubelet.service`。

``` bash
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/usr/local/bin/kubelet \
  --cgroup-driver=cgroupfs \
  --address=172.20.0.113 \
  --hostname-override=172.20.0.113 \
  --pod-infra-container-image=172.20.0.119/k8s/rhel7/pod-infrastructure:latest \
  --experimental-bootstrap-kubeconfig=/etc/kubernetes/bootstrap.kubeconfig \
  --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \
  --cert-dir=/etc/kubernetes/ssl \
  --cluster-dns=10.254.0.2 \
  --cluster-domain=cluster.local. \
  --hairpin-mode promiscuous-bridge \
  --allow-privileged=true \
  --fail-swap-on=false \
  --serialize-image-pulls=false \
  --logtostderr=true \
  --max-pods=512 \
  --v=2
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
```
### 启动kublet

``` bash
systemctl daemon-reload
systemctl enable kubelet
systemctl start kubelet
systemctl status kubelet
```

### 通过 kublet 的 TLS 证书请求

kubelet 首次启动时向 kube-apiserver 发送证书签名请求，必须通过后 kubernetes 系统才会将该 Node 加入到集群。

查看未授权的 CSR 请求

``` bash
$ kubectl get csr
NAME        AGE       REQUESTOR           CONDITION
csr-2b308   4m        kubelet-bootstrap   Pending
$ kubectl get nodes
No resources found.
```

通过 CSR 请求

``` bash
$ kubectl certificate approve csr-2b308
certificatesigningrequest "csr-2b308" approved
$ kubectl get nodes
NAME        STATUS    AGE       VERSION
10.64.3.7   Ready     49m       v1.6.1
```

自动生成了 kubelet kubeconfig 文件和公私钥

``` bash
$ ls -l /etc/kubernetes/kubelet.kubeconfig
-rw------- 1 root root 2284 Apr  7 02:07 /etc/kubernetes/kubelet.kubeconfig
$ ls -l /etc/kubernetes/ssl/kubelet*
-rw-r--r-- 1 root root 1046 Apr  7 02:07 /etc/kubernetes/ssl/kubelet-client.crt
-rw------- 1 root root  227 Apr  7 02:04 /etc/kubernetes/ssl/kubelet-client.key
-rw-r--r-- 1 root root 1103 Apr  7 02:07 /etc/kubernetes/ssl/kubelet.crt
-rw------- 1 root root 1675 Apr  7 02:07 /etc/kubernetes/ssl/kubelet.key
```
## 配置 kube-proxy

**安装conntrack**

```bash
yum install -y conntrack-tools
```

**创建 kube-proxy 的service配置文件**

文件路径`vi /usr/lib/systemd/system/kube-proxy.service`。
```
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
WorkingDirectory=/var/lib/kube-proxy
ExecStart=/usr/local/bin/kube-proxy \
  --bind-address=172.20.0.113 \
  --hostname-override=172.20.0.113 \
  --cluster-cidr=10.254.0.0/16 \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig \
  --logtostderr=true \
  --v=2
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

### 启动 kube-proxy

``` bash
systemctl daemon-reload
systemctl enable kube-proxy
systemctl start kube-proxy
systemctl status kube-proxy
```


