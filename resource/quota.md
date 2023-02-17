https://kubernetes.io/zh-cn/docs/concepts/policy/resource-quotas/

# 资源配额

当多个用户或团队共享具有固定节点数目的集群时，人们会担心有人使用超过其基于公平原则所分配到的资源量。

资源配额是帮助管理员解决这一问题的工具。

资源配额，通过 `ResourceQuota` 对象来定义，对每个命名空间的资源消耗总量提供限制。 它可以限制命名空间中某种类型的对象的总数目上限，也可以限制命名空间中的 Pod 可以使用的计算资源的总上限。

## 计算资源配额

| `limits.cpu`       | 所有非终止状态的 Pod，其 CPU 限额总量不能超过该值。          |
| ------------------ | ------------------------------------------------------------ |
| `limits.memory`    | 所有非终止状态的 Pod，其内存限额总量不能超过该值。           |
| `requests.cpu`     | 所有非终止状态的 Pod，其 CPU 需求总量不能超过该值。          |
| `requests.memory`  | 所有非终止状态的 Pod，其内存需求总量不能超过该值。           |
| `hugepages-<size>` | 对于所有非终止状态的 Pod，针对指定尺寸的巨页请求总数不能超过此值。 |
| `cpu`              | 与 `requests.cpu` 相同。                                     |
| `memory`           | 与 `requests.memory` 相同。                                  |





- #### 项目资源配额

```shell

kubectl apply -f quota.yaml


#将部署的pod 数量增大到限额以上，看是否可以实现
kubectl scale deployment/example --replicas=5 -n test1

#实验完成后，删除配额
kubectl delete -f quota.yaml

# kubectl get resourcequota -n test1
NAME      AGE   REQUEST     LIMIT
example   29s   pods: 1/4

```

- quota.yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: example
spec:
  hard:
    pods: '4'
    #requests.cpu: '1'
    #requests.memory: 1Gi
    #limits.cpu: '2'
    #limits.memory: 2Gi
```





## 存储资源配额

用户可以对给定命名空间下的[存储资源](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/) 总量进行限制。

此外，还可以根据相关的存储类（Storage Class）来限制存储资源的消耗。

| 资源名称                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `requests.storage`                                           | 所有 PVC，存储资源的需求总量不能超过该值。                   |
| `persistentvolumeclaims`                                     | 在该命名空间中所允许的 [PVC](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) 总量。 |
| `<storage-class-name>.storageclass.storage.k8s.io/requests.storage` | 在所有与 `<storage-class-name>` 相关的持久卷申领中，存储请求的总和不能超过该值。 |
| `<storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims` | 在与 storage-class-name 相关的所有持久卷申领中，命名空间中可以存在的[持久卷申领](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)总数。 |



### 与limitrange的区别

limitrage在pod, container级别

qutota在ns级别
