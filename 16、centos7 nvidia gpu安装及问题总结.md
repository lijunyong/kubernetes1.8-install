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

#刚开始安装会进入more模式，一直按`空格`即可

#Do you accept the previously read EULA?
#是否接受协议
accept

#Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 387.26?
#是否安装Nvida驱动，需要
y

#Do you want to install the OpenGL libraries?
#是否需要安装OpenGL，不需要
n

#Do you want to run nvidia-xconfig?
回车

#Install the CUDA 9.1 Toolkit?
y

#Enter Toolkit Location
#输入Toolkit的安装目录
#一般默认即可
回车

#Do you want to install a symbolic link at /usr/local/cuda?
#创建一个软连接，我选择是
y

#Install the CUDA 9.1 Samples?
#安装CUDA官方示例包
y

#Enter CUDA Samples Location
#输入示例包的安装目录，我选的默认路径，可以根据实际情况选择输入
回车


#如果版本不同，提示也可能不同，根据情况输入或选择
```

配置环境变量
如果有注意的话，cuda安装完之后有个提示，要配置环境变量
```
#修改系统配置文件
sudo vim /etc/profile

#在最后面加入以下内容
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

#保存并退出
:wq

#使配置文件生效
sudo source /etc/profile
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

