# 安装准备阶段
+ 关闭centos防火墙
+ 关闭SELINUX


## 关闭centos防火墙
```
#停止firewalld服务
systemctl stop firewalld

#禁用firewalld服务
systemctl mask firewalld
```

## 关闭SELINUX
关闭SELinux：

+ 临时关闭（不用重启机器）：
```
##设置SELinux 成为permissive模式
##setenforce 1 设置SELinux 成为enforcing模式
setenforce 0         
```
+ 修改配置文件需要重启机器：<br/>
修改/etc/selinux/config 文件，将SELINUX=enforcing改为SELINUX=disabled，重启机器即可
