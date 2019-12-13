## 环境准备
**准备两台宿主机，在宿主机上安装docker-ce**

|  主机ip   | docker0网络  |
|  ----  | ----  |
| 172.20.0.118  | 10.0.128.0/24 |
| 172.20.0.119  | 10.0.129.0/24 |

**配置docker ip段地址**  
1、在172.20.0.118上配置如下
```
[root@localhost ~]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://0b33z6az.mirror.aliyuncs.com"],
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"], 
  "insecure-registries": ["172.20.0.119"],
  "bip": "10.0.128.1/24"
}
```
2、在172.20.0.119上配置如下
```
[root@localhost ~]# cat /etc/docker/daemon.json
{
  "registry-mirrors": ["https://0b33z6az.mirror.aliyuncs.com"],
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"], 
  "insecure-registries": ["172.20.0.119"],
  "bip": "10.0.129.1/24"
}
```
## 配置路由规则
