# 安装dashboard插件

## 生成dashboard1.8 证书

通过openssl生成自签名证书，`dashboard.crt`后续需要导入浏览器访问dashboard。参考：`https://github.com/kubernetes/dashboard/wiki/Certificate-management#public-trusted-certificate-authority`
```
$ openssl genrsa -des3 -passout pass:x -out dashboard.pass.key 2048
...
$ openssl rsa -passin pass:x -in dashboard.pass.key -out dashboard.key
# Writing RSA key
$ rm dashboard.pass.key
$ openssl req -new -key dashboard.key -out dashboard.csr
...
Country Name (2 letter code) [AU]: US
...
A challenge password []:
...

$ openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
```
## 创建dashboard secret
依赖生成的证书，创建dashboard需要的secret，参考：`https://github.com/kubernetes/dashboard/wiki/Installation`<br/>
This setup requires, that certificates are stored in a secret named kubernetes-dashboard-certs in kube-system namespace. 
Assuming that you have dashboard.crt and dashboard.key files stored under $HOME/certs directory, 
you should create secret with contents of these files:
```
kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kube-system
```

官方文件目录：`https://github.com/kubernetes/kubernetes/tree/release-1.8/cluster/addons/dashboard`

我们使用的文件如下：
```
dashboard-configmap.yaml
dashboard-controller.yaml
dashboard-rbac.yaml
dashboard-service.yaml
```
修改好的[yaml](https://github.com/lijunyong/kubernetes1.8-install/tree/master/cluster/dashboard)文件
## 执行所有定义文件

``` bash
$ kubectl create -f  .
```

## 检查执行结果

查看分配的 NodePort

``` bash
[root@localhost dashboard]# kubectl get services kubernetes-dashboard -n kube-system
NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.254.164.205   <none>        443:31717/TCP   14d
```
浏览器通过：`https://172.20.0.113:31717/`访问 <br/>
![kubernetes dashboard](/images/dashboard.png)

### 生成 token

需要创建一个admin用户并授予admin角色绑定，使用下面的yaml文件创建admin用户并赋予他管理员权限，然后可以通过token登陆dashbaord，文件[admin-role.yaml]。这种认证方式本质上是通过 Service Account 的身份认证加上 Bearer token 请求 API server 的方式实现，参考 [Kubernetes 中的认证](https://kubernetes.io/docs/admin/authentication/)。

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
```

然后执行下面的命令创建 serviceaccount 和角色绑定，对于其他命名空间的其他用户只要修改上述 yaml 中的 `name` 和 `namespace` 字段即可：

```bash
kubectl create -f admin-role.yaml
```

创建完成后获取secret和token的值。

```bash
# 获取admin-token的secret名字
[root@localhost dashboard]# kubectl -n kube-system get secret|grep admin-token
admin-token-xthwd                        kubernetes.io/service-account-token   3         17d

# 获取token的值

[root@localhost dashboard]# kubectl -n kube-system describe secret admin-token-xthwd
Name:         admin-token-xthwd
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin
              kubernetes.io/service-account.uid=2b9925d9-f9e7-11e7-acec-000c29b3f6ff

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1359 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi14dGh3ZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjJiOTkyNWQ5LWY5ZTctMTFlNy1hY2VjLTAwMGMyOWIzZjZmZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.T-hrF4Drj88VdEruUsRJDGJakxmRartj7lRzxA6RHrJs6liyA00nAUt1EK--QoW1lxBSSqCWUMSeTm904GLazbi7WHpoHfzOowb9qqvUtBqlNF-0mc1xTmdBzdfh0MlfzTkx7w5EZWc9ItNWWLKdJkI2oUuoEpA_aF_-fJGxFRAZkuUrl4c1IFqJCsnqyp5czu6C0SalZzss86ZoRlkfAnNzihSvp2uu59driYnipEdPBUHv3A7ceiEESbg6DqOL1hATVIvThP8jAQFCLztNapZWtDpUWoLQ-Sc3ZIPZkGqlR_aY3ts9mBb1NVXPyhsWKdwnt2vH589GdQDl8e5lXQ

```
![kubernetes dashboard mon](/images/dashboard-mon.png)


