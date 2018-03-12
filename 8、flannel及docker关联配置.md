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

- /run/flannel/docker

```ini
[root@localhost ~]# cat /run/flannel/docker 
DOCKER_OPT_BIP="--bip=172.30.21.1/24"
DOCKER_OPT_IPMASQ="--ip-masq=true"
DOCKER_OPT_MTU="--mtu=1450"
DOCKER_NETWORK_OPTIONS=" --bip=172.30.21.1/24 --ip-masq=true --mtu=1450"
```

Docker将会读取这两个环境变量文件作为容器启动参数。

**注意：**不论您用什么方式安装的flannel，下面这一步是必不可少的。

修改docker的配置文件`vi /usr/lib/systemd/system/docker.service`，增加环境变量配置：

```
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
**注意**  ifconfig 检查flannel、docker0网卡是否一致，不一致会导致pod之间网络不通，这也是导致heapster、dashoard等应用出现问题的主要原因之一。<br/>
**如果flannel.1重启后网卡地址无法跟docker0匹配，请执行命令：ip link delete flannel.1，然后重新启动flannel**
``` bash
[root@localhost ~]# ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 172.30.82.1  netmask 255.255.255.0  broadcast 172.30.82.255
        inet6 fe80::42:45ff:feff:42e1  prefixlen 64  scopeid 0x20<link>
        ether 02:42:45:ff:42:e1  txqueuelen 0  (Ethernet)
        RX packets 42558  bytes 9449904 (9.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 47405  bytes 270788340 (258.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.20.0.113  netmask 255.255.255.0  broadcast 172.20.0.255
        inet6 fe80::5ec5:cbd0:8e08:efaa  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:b3:f6:ff  txqueuelen 1000  (Ethernet)
        RX packets 2013762  bytes 621574860 (592.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1929515  bytes 684681865 (652.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 172.30.82.0  netmask 255.255.255.255  broadcast 0.0.0.0
        ether ee:02:08:99:95:09  txqueuelen 0  (Ethernet)
        RX packets 193712  bytes 276110507 (263.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10501  bytes 851790 (831.8 KiB)
        TX errors 0  dropped 23 overruns 0  carrier 0  collisions 0

```
