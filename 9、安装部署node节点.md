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
