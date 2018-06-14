# 安装nvidia gpu
## 安装gpu驱动
从nvidia官网`https://www.nvidia.com/Download/index.aspx?lang=en-us`下载对应版本驱动，我用的是centos7

```
得到文件：NVIDIA-Linux-x86_64-396.26.run
```

安装依赖库
```
yum install gcc gcc-c++
yum install kernel-headers
```

+ **kernel-devel特别注意，需要安装系统对应内核版本，否则后续安装NVIDIA-Linux-x86_64-396.26.run会报错**

```
rpm -ivh kernel-devel-3.10.0-514.26.2.el7.x86_64.rpm
```

安装dkms
```
yum install epel-release
yum install --enablerepo=epel dkms
```

安装gpu驱动，需要指定kernel所在位置
```
sh NVIDIA-Linux-x86_64-396.26.run --kernel-source-path=/usr/src/kernels/3.10.0-514.26.2.el7.x86_64/
```

进入驱动安装界面

## 安装cuda

访问nvidia官网`https://developer.nvidia.com/cuda-downloads`，下载对应版本cuda
```
sh cuda_9.2.88_396.26_linux
```


## 安装中遇到的问题

### 1、NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running

安装cuda之前需要安装gpu驱动，下载`NVIDIA-Linux-x86_64-396.26.run`，进行安装

### 2、ERROR: Unable to load the kernel module 'nvidia.ko'.

```
可以通过以下方式查看内核版本和源码包版本：
[root@iZuf694gmi68hzf8sfgl9bZ opt]# ls /boot | grep vmlinuz
vmlinuz-0-rescue-963c2c41b08343f7b063dddac6b2e486
vmlinuz-3.10.0-514.26.2.el7.x86_64
vmlinuz-3.10.0-514.el7.x86_64

通过`ftp://ftp.riken.jp/Linux/cern/centos/7/updates/x86_64/repoview/kernel-devel.html`，下载对应版本kernel-devel进行安装
```

### 3、Unable to load the 'nvidia-drm' kernel module
```
安装dkms
yum install epel-release
yum install --enablerepo=epel dkms
```

### 4、安装gpu驱动

```
sh NVIDIA-Linux-x86_64-396.26.run  --kernel-source-path=/usr/src/kernels/3.10.0-514.26.2.el7.x86_64/
```

