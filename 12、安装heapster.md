# 安装heapster插件

## 准备镜像
需要的镜像，大家都去去阿里云搜索：`https://dev.aliyun.com/search.html`
- 172.20.0.119/k8s/heapster-amd64:v1.4.3
- 172.20.0.119/k8s/heapster-influxdb-amd64:v1.3.3
- 172.20.0.119/k8s/heapster-grafana-amd64:v4.4.3

## 准备YAML文件

到 `https://github.com/kubernetes/heapster/tree/release-1.5/deploy/kube-config`下载最新版本的 kubernetes yaml。

``` 
grafana.yaml
heapster.yaml
heapster-rbac.yaml
influxdb.yaml
```

主要修改heapster.yaml配置文件
```
 command:
        - /heapster
        - --source=kubernetes:http://172.20.0.113:8080?inClusterConfig=false
        - --sink=influxdb:http://monitoring-influxdb.kube-system.svc:8086
```
+ `--source` 修改成你所部署的kube-apiserver地址


## 执行所有定义文件

``` bash
$ kubectl create -f  .
deployment "monitoring-grafana" created
service "monitoring-grafana" created
deployment "heapster" created
serviceaccount "heapster" created
clusterrolebinding "heapster" created
service "heapster" created
configmap "influxdb-config" created
deployment "monitoring-influxdb" created
service "monitoring-influxdb" created
```

## 访问 grafana

1. 通过 kube-apiserver 访问：

获取 monitoring-grafana 服务 URL

``` bash
[root@localhost ~]# kubectl cluster-info
Kubernetes master is running at https://172.20.0.113:6443
Heapster is running at https://172.20.0.113:6443/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://172.20.0.113:6443/api/v1/namespaces/kube-system/services/kube-dns/proxy
kubernetes-dashboard is running at https://172.20.0.113:6443/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy
monitoring-grafana is running at https://172.20.0.113:6443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
monitoring-influxdb is running at https://172.20.0.113:6443/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```

浏览器访问 URL： `http://172.20.0.113:8080/api/v1/proxy/namespaces/kube-system/services/monitoring-grafana`

2. 通过 kubectl proxy 访问：

创建代理

 ``` bash
$ kubectl proxy --address='172.20.0.113' --port=8086 --accept-hosts='^*$'
Starting to serve on 172.20.0.113:8086
```

浏览器访问 URL：`http://172.20.0.113:8086/api/v1/proxy/namespaces/kube-system/services/monitoring-grafana`


## 访问 influxdb admin UI

获取 influxdb http 8086 映射的 NodePort

```
$ kubectl get svc -n kube-system|grep influxdb
monitoring-influxdb    10.254.22.46    <nodes>       8086:32299/TCP,8083:30269/TCP   9m
```
