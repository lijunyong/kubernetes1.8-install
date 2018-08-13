# etcd备份、恢复及数据迁移
etcd在使用中难免遇到故障情况，这就涉及数据备份、恢复、迁移的问题。 参考
+ https://blog.csdn.net/kozazyh/article/details/79587141
+ https://www.maideliang.com/index.php/archives/25/

## 保存快照
由于我使用的书tsl部署etcd集群，所有需要证书：
```
ETCDCTL_API=3 etcdctl \
--endpoints=https://172.20.0.113:2379 \  
--cacert=/etc/kubernetes/ssl/ca.pem  \
--cert=/etc/kubernetes/ssl/kubernetes.pem  \
--key=/etc/kubernetes/ssl/kubernetes-key.pem \
snapshot save /opt/snapshot.db
```
**注意** kubernetes访问etcd是通过ETCDCTL_API=3，ectd访问kubernetest数据，如下
```
ETCDCTL_API=3 etcdctl \
--endpoints=https://172.20.0.113:2379 \
--cacert=/etc/kubernetes/ssl/ca.pem \
--cert=/etc/kubernetes/ssl/kubernetes.pem \
--key=/etc/kubernetes/ssl/kubernetes-key.pem \
get /registry/namespaces --prefix -w=json
{"header":{"cluster_id":14187615942257587067,"member_id":8817427494834720418,"revision":1022623,"raft_term":18670},"kvs":[{"key":"L3JlZ2lzdHJ5L25hbWVzcGFjZXMvZGVmYXVsdA==","create_revision":4,"mod_revision":712781,"version":6,"value":"azhzAAoPCgJ2MRIJTmFtZXNwYWNlEnsKYQoHZGVmYXVsdBIAGgAiACokNmJkNDdiNjgtMjY2OS0xMWU4LTgxZDEtMDAwYzI5YTZlMmM2MgA4AEIICK35nNUFEABaGgoPaXN0aW8taW5qZWN0aW9uEgdlbmFibGVkegASDAoKa3ViZXJuZXRlcxoICgZBY3RpdmUaACIA"},{"key":"L3JlZ2lzdHJ5L25hbWVzcGFjZXMvaXN0aW8tc3lzdGVt","create_revision":725693,"mod_revision":725693,"version":1,"value":"azhzAAoPCgJ2MRIJTmFtZXNwYWNlEmQKSgoMaXN0aW8tc3lzdGVtEgAaACIAKiRiYTM5ODI2Mi05MTcyLTExZTgtOGQ2NC0wMDBjMjlhNmUyYzYyADgAQggI8aDr2gUQAHoAEgwKCmt1YmVybmV0ZXMaCAoGQWN0aXZlGgAiAA=="},{"key":"L3JlZ2lzdHJ5L25hbWVzcGFjZXMva3ViZS1wdWJsaWM=","create_revision":9,"mod_revision":9,"version":1,"value":"azhzAAoPCgJ2MRIJTmFtZXNwYWNlEmMKSQoLa3ViZS1wdWJsaWMSABoAIgAqJDZiZGFiZTNhLTI2NjktMTFlOC04MWQxLTAwMGMyOWE2ZTJjNjIAOABCCAit+ZzVBRAAegASDAoKa3ViZXJuZXRlcxoICgZBY3RpdmUaACIA"},{"key":"L3JlZ2lzdHJ5L25hbWVzcGFjZXMva3ViZS1zeXN0ZW0=","create_revision":8,"mod_revision":8,"version":1,"value":"azhzAAoPCgJ2MRIJTmFtZXNwYWNlEmMKSQoLa3ViZS1zeXN0ZW0SABoAIgAqJDZiZDkwZjYyLTI2NjktMTFlOC04MWQxLTAwMGMyOWE2ZTJjNjIAOABCCAit+ZzVBRAAegASDAoKa3ViZXJuZXRlcxoICgZBY3RpdmUaACIA"}],"count":4}
```
## 数据恢复、迁移
把生成的snapshot.db拷贝到需要恢复的etcd集群，在三台etcd机器上分别执行如下命令：
```
# etcd1
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db  \     
--endpoints=http://172.16.56.226:2379 \
--name=etcd1     \   
--initial-advertise-peer-urls=http://172.16.56.226:2380      \
--initial-cluster-token=k8s-etcd-cluster    \
--initial-cluster=etcd1=http://172.16.56.226:2380,etcd2=http://172.16.56.225:2380,etcd3=http://172.16.56.227:2380    \
--data-dir=/var/lib/etcd

# etcd2
ETCDCTL_API=3  etcdctl snapshot restore snapshot.db   \
--endpoints=http://172.16.56.225:2379 --name=etcd2   \
--initial-advertise-peer-urls=http://172.16.56.225:2380   \
--initial-cluster-token=k8s-etcd-cluster    \
--initial-cluster=etcd1=http://172.16.56.226:2380,etcd2=http://172.16.56.225:2380,etcd3=http://172.16.56.227:2380    \
--data-dir=/var/lib/etcd

# etcd3
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db  \
--endpoints=http://172.16.56.227:2379 --name=etcd3   \
--initial-advertise-peer-urls=http://172.16.56.227:2380  \
--initial-cluster-token=k8s-etcd-cluster  \
--initial-cluster=etcd1=http://172.16.56.226:2380,etcd2=http://172.16.56.225:2380,etcd3=http://172.16.56.227:2380  \
--data-dir=/var/lib/etcd

```
**注意** 特别注意数据恢复etcd集群千万不需要启动，恢复后在启动

然后启动etcd集群
```
systemctl restart etcd
```

检测数据是否恢复成功
```
[root@iZbp15vsdlkxqh8dzmse80Z opt]# ETCDCTL_API=3 etcdctl --endpoints=http://172.16.56.226:2379 get /registry/namespaces/ --prefix -w=json
{"header":{"cluster_id":3209749552177254380,"member_id":17045765930214344638,"revision":1008481,"raft_term":3},"kvs":[{"key":"L3JlZ2lzdHJ5L25hbWVzcGFjZXMvZGVmYXVsdA==","create_revision":4,"mod_revision":712781,"version":6,"value":"azhzAAoPCgJ2MRIJTmFtZXNwYWNlEnsKYQoHZGVmYXVsdBIAGgAiACokNmJkNDdiNjgtMjY2OS0xMWU4LTgxZDEtMDAwYzI5YTZlMmM2MgA4AEIICK35nNUFEABaGgoPaXN0aW8taW5qZWN0aW9uEgdlbmFibGVkegASDAoKa3ViZXJuZXRlcxoICgZBY3RpdmUaACIA"},{"key":"L3JlZ2lzdHJ5L25hbWVzcGFjZXMvaXN0aW8tc3lzdGVt","create_revision":725693,"mod_revision":725693,"version":1,"value":"azhzAAoPCgJ2MRIJTmFtZXNwYWNlEmQKSgoMaXN0aW8tc3lzdGVtEgAaACIAKiRiYTM5ODI2Mi05MTcyLTExZTgtOGQ2NC0wMDBjMjlhNmUyYzYyADgAQggI8aDr2gUQAHoAEgwKCmt1YmVybmV0ZXMaCAoGQWN0aXZlGgAiAA=="},{"key":"L3JlZ2lzdHJ5L25hbWVzcGFjZXMva3ViZS1wdWJsaWM=","create_revision":9,"mod_revision":9,"version":1,"value":"azhzAAoPCgJ2MRIJTmFtZXNwYWNlEmMKSQoLa3ViZS1wdWJsaWMSABoAIgAqJDZiZGFiZTNhLTI2NjktMTFlOC04MWQxLTAwMGMyOWE2ZTJjNjIAOABCCAit+ZzVBRAAegASDAoKa3ViZXJuZXRlcxoICgZBY3RpdmUaACIA"},{"key":"L3JlZ2lzdHJ5L25hbWVzcGFjZXMva3ViZS1zeXN0ZW0=","create_revision":8,"mod_revision":8,"version":1,"value":"azhzAAoPCgJ2MRIJTmFtZXNwYWNlEmMKSQoLa3ViZS1zeXN0ZW0SABoAIgAqJDZiZDkwZjYyLTI2NjktMTFlOC04MWQxLTAwMGMyOWE2ZTJjNjIAOABCCAit+ZzVBRAAegASDAoKa3ViZXJuZXRlcxoICgZBY3RpdmUaACIA"}],"count":4}
[root@iZbp15vsdlkxqh8dzmse80Z opt]# 

```
