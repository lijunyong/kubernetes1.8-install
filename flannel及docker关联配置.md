# flannel及docker关联配置
我自己在这个配置过程中，遇到很多问题，特地单抽出一个章节进行详细配置

> 如果您使用yum的方式安装的flannel则不需要执行mk-docker-opts.sh文件这一步，参考Flannel官方文档中的[Docker Integration](https://github.com/coreos/flannel/blob/master/Documentation/running.md)。

如果你不是使用yum安装的flannel，那么需要下载flannel github release中的tar包，解压后会获得一个**mk-docker-opts.sh**文件，到[flannel release](https://github.com/coreos/flannel/releases)页面下载对应版本的安装包，该脚本见[mk-docker-opts.sh](https://github.com/rootsongjc/kubernetes-handbook/tree/master/tools/flannel/mk-docker-opts.sh)，因为我们使用yum安装所以不需要执行这一步。

这个文件是用来`Generate Docker daemon options based on flannel env file`。

使用`systemctl`命令启动flanneld后，会自动执行`./mk-docker-opts.sh -i`生成如下两个文件环境变量文件：

- /run/flannel/subnet.env

```ini
FLANNEL_NETWORK=172.30.0.0/16
FLANNEL_SUBNET=172.30.46.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=false
```

- /run/docker_opts.env

```ini
DOCKER_OPT_BIP="--bip=172.30.46.1/24"
DOCKER_OPT_IPMASQ="--ip-masq=true"
DOCKER_OPT_MTU="--mtu=1450"
```

Docker将会读取这两个环境变量文件作为容器启动参数。

**注意：**不论您用什么方式安装的flannel，下面这一步是必不可少的。

修改docker的配置文件`vi /usr/lib/systemd/system/docker.service`，增加环境变量配置：

```
[root@localhost ~]# cat /etc/sysconfig/flanneld
# Flanneld configuration options  

# etcd url location.  Point this to the server where etcd runs
ETCD_ENDPOINTS="https://172.20.0.113:2379,https://172.20.0.114:2379,https://172.20.0.115:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
ETCD_PREFIX="/kube-centos/network"

# Any additional options that you want to pass
FLANNEL_OPTIONS="-etcd-cafile=/etc/kubernetes/ssl/ca.pem -etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem -etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem"
[root@localhost ~]# cat /usr/lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
EnvironmentFile=-/run/flannel/docker
EnvironmentFile=-/run/flannel/subnet.env
ExecStart=/usr/bin/dockerd \
          $DOCKER_OPT_BIP \
          $DOCKER_OPT_IPMASQ \
          $DOCKER_OPT_MTU
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

### 重新启动docker
``` bash
systemctl daemon-reload
systemctl restart docker
```

