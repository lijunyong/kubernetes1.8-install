# 安装准备阶段
+ 关闭centos防火墙
+ 关闭SELINUX
+ 创建TLS证书和秘钥

## 关闭centos防火墙
```
#停止firewalld服务
systemctl stop firewalld

#禁用firewalld服务
systemctl mask firewalld
```
