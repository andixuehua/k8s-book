https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/

### 回收（Reclaiming）

当用户不再使用其存储卷时，他们可以从 API 中将 PVC 对象删除， 从而允许该资源被回收再利用。PersistentVolume 对象的回收策略告诉集群， 当其被从申领中释放时如何处理该数据卷。 目前，数据卷可以被 Retained（保留）、Recycled（回收）或 Deleted（删除）。

#### 保留（Retain）

回收策略 `Retain` 使得用户可以手动回收资源。当 PersistentVolumeClaim 对象被删除时，PersistentVolume 卷仍然存在，对应的数据卷被视为"已释放（released）"。 由于卷上仍然存在这前一申领人的数据，该卷还不能用于其他申领。 管理员可以通过下面的步骤来手动回收该卷：

1. 删除 PersistentVolume 对象。与之相关的、位于外部基础设施中的存储资产 （例如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）在 PV 删除之后仍然存在。
2. 根据情况，手动清除所关联的存储资产上的数据。
3. 手动删除所关联的存储资产。

如果你希望重用该存储资产，可以基于存储资产的定义创建新的 PersistentVolume 卷对象。

#### 删除（Delete）

对于支持 `Delete` 回收策略的卷插件，删除动作会将 PersistentVolume 对象从 Kubernetes 中移除，同时也会从外部基础设施（如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）中移除所关联的存储资产。 动态制备的卷会继承[其 StorageClass 中设置的回收策略](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#reclaim-policy)， 该策略默认为 `Delete`。管理员需要根据用户的期望来配置 StorageClass； 否则 PV 卷被创建之后必须要被编辑或者修补。 参阅[更改 PV 卷的回收策略](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/change-pv-reclaim-policy/)。





- #### 改变卷的reclaimpolicy，保留数据,复 用原有数据

- reclaimpolicy最初从sc里面设定，建立的pv自动继承sc的配置，但是在pv配置内可以自行修改reclaimpolicy

- 前提，[先完成此pv/pvc配置](../pv/rwo.md)

```


#删除所有pod,删除 pvc，检查发现pv也会自动删除，

＃检查storageclass的，为以下配置
kubectl get sc
reclaimPolicy: Delete

#检查pv的配置，也为以下的配置：
kubectl get pv xxx -o yaml 
  persistentVolumeReclaimPolicy: Delete
    
```

- 如何保留原有卷的数据

```shell
＃可将pv中的配置修改为：
kubectl edit pv xxx
persistentVolumeReclaimPolicy: Retain

#然后将pod删除，再删除pvc,检查pv,状态会变为release,但不会被删除掉

＃新建一个相同名字，相同容量的pvc,启用 pod,发现pvc并不会bound到原来的pv上去，而是新建一个pv出来

因为只有当pv的状态是availabel状态时，才能被其他pvc使用
```

- 释放pv,复用pv

```shell
 #检查pv的配置，删除类似以下所有内容，检查pv的状态，会变为Available
 kubectl edit pv xxx

 claimRef:
    kind: PersistentVolumeClaim
    namespace: test-vol
    name: test-vol
    uid: 46276e0f-8026-40f5-a61f-eaa3978046de
    apiVersion: v1
    resourceVersion: '518384'

＃新建一个相同名字，相同容量的pvc,启用 pod,此时的pvc会重新bound到现有的pv上去了。
```



**当pv　persistentVolumeReclaimPolicy是Ｔetain时, 使用kubectl delete pv xxx,后端的目录一样会保留**
