

https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/

# 卷volume

Container 中的文件在磁盘上是临时存放的，这给 Container 中运行的较重要的应用程序带来一些问题。 问题之一是当容器崩溃时文件丢失。 kubelet 会重新启动容器，但容器会以干净的状态重启。 第二个问题会在同一 `Pod` 中运行多个容器并共享文件时出现。 Kubernetes [卷（Volume）](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/) 这一抽象概念能够解决这两个问题。



https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/

# 持久卷

本文描述 Kubernetes 中的**持久卷（Persistent Volume）** 。 建议先熟悉[卷（Volume）](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/)的概念。

## 介绍

存储的管理是一个与计算实例的管理完全不同的问题。 PersistentVolume 子系统为用户和管理员提供了一组 API， 将存储如何制备的细节从其如何被使用中抽象出来。 为了实现这点，我们引入了两个新的 API 资源：PersistentVolume 和 PersistentVolumeClaim。

**持久卷（PersistentVolume，PV）** 是集群中的一块存储，可以由管理员事先制备， 或者使用[存储类（Storage Class）](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/)来动态制备。 持久卷是集群资源，就像节点也是集群资源一样。PV 持久卷和普通的 Volume 一样， 也是使用卷插件来实现的，只是它们拥有独立于任何使用 PV 的 Pod 的生命周期。 此 API 对象中记述了存储的实现细节，无论其背后是 NFS、iSCSI 还是特定于云平台的存储系统。

**持久卷申领（PersistentVolumeClaim，PVC）** 表达的是用户对存储的请求。概念上与 Pod 类似。 Pod 会耗用节点资源，而 PVC 申领会耗用 PV 资源。Pod 可以请求特定数量的资源（CPU 和内存）；同样 PVC 申领也可以请求特定的大小和访问模式 （例如，可以要求 PV 卷能够以 ReadWriteOnce、ReadOnlyMany 或 ReadWriteMany 模式之一来挂载，参见[访问模式](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#access-modes)）。

尽管 PersistentVolumeClaim 允许用户消耗抽象的存储资源， 常见的情况是针对不同的问题用户需要的是具有不同属性（如，性能）的 PersistentVolume 卷。 集群管理员需要能够提供不同性质的 PersistentVolume， 并且这些 PV 卷之间的差别不仅限于卷大小和访问模式，同时又不能将卷是如何实现的这些细节暴露给用户。 为了满足这类需求，就有了**存储类（StorageClass）**资源。



## 卷和申领的生命周期[ ](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#lifecycle-of-a-volume-and-claim)

PV 卷是集群中的资源。PVC 申领是对这些资源的请求，也被用来执行对资源的申领检查。 PV 卷和 PVC 申领之间的互动遵循如下生命周期：

### 制备

PV 卷的制备有两种方式：静态制备或动态制备。

#### 静态制备

集群管理员创建若干 PV 卷。这些卷对象带有真实存储的细节信息， 并且对集群用户可用（可见）。PV 卷对象存在于 Kubernetes API 中，可供用户消费（使用）。

#### 动态制备

如果管理员所创建的所有静态 PV 卷都无法与用户的 PersistentVolumeClaim 匹配， 集群可以尝试为该 PVC 申领动态制备一个存储卷。 这一制备操作是基于 StorageClass 来实现的：PVC 申领必须请求某个 [存储类](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/)， 同时集群管理员必须已经创建并配置了该类，这样动态制备卷的动作才会发生。 如果 PVC 申领指定存储类为 `""`，则相当于为自身禁止使用动态制备的卷。

为了基于存储类完成动态的存储制备，集群管理员需要在 API 服务器上启用 `DefaultStorageClass` [准入控制器](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)。 举例而言，可以通过保证 `DefaultStorageClass` 出现在 API 服务器组件的 `--enable-admission-plugins` 标志值中实现这点；该标志的值可以是逗号分隔的有序列表。 关于 API 服务器标志的更多信息，可以参考 [kube-apiserver](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-apiserver/) 文档。



https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/

# 存储类

本文描述了 Kubernetes 中 StorageClass 的概念。 建议先熟悉[卷](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/)和[持久卷](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes)的概念。

## 介绍

StorageClass 为管理员提供了描述存储 "类" 的方法。 不同的类型可能会映射到不同的服务质量等级或备份策略，或是由集群管理员制定的任意策略。 Kubernetes 本身并不清楚各种类代表的什么。这个类的概念在其他存储系统中有时被称为 "配置文件"。

## StorageClass 资源

每个 StorageClass 都包含 `provisioner`、`parameters` 和 `reclaimPolicy` 字段， 这些字段会在 StorageClass 需要动态分配 PersistentVolume 时会使用到。

StorageClass 对象的命名很重要，用户使用这个命名来请求生成一个特定的类。 当创建 StorageClass 对象时，管理员设置 StorageClass 对象的命名和其他参数，一旦创建了对象就不能再对其更新。

管理员可以为没有申请绑定到特定 StorageClass 的 PVC 指定一个默认的存储类： 更多详情请参阅 [PersistentVolumeClaim 章节](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)。



https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

```
# kubectl get sc 
NAME                   PROVISIONER   RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client (default)   nfs           Delete          Immediate           true                   228d

# kubectl get sc  -o yaml nfs-client
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"nfs-client"},"parameters":{"archiveOnDelete":"false"},"provisioner":"nfs"}
    storageclass.kubernetes.io/is-default-class: "true"
  name: nfs-client
parameters:
  archiveOnDelete: "false"
provisioner: nfs
reclaimPolicy: Delete
volumeBindingMode: Immediate


 kubectl get deployments.apps -n nfs nfs-client-provisioner  -o yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nfs-client-provisioner","namespace":"nfs"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"nfs-client-provisioner"}},"strategy":{"type":"Recreate"},"template":{"metadata":{"labels":{"app":"nfs-client-provisioner"}},"spec":{"containers":[{"env":[{"name":"PROVISIONER_NAME","value":"k8s-sigs.io/nfs-subdir-external-provisioner"},{"name":"NFS_SERVER","value":"nfs.taikang1.local"},{"name":"NFS_PATH","value":"/mnt"}],"image":"hub.taikang1.local/k8s/nfs-subdir-external-provisioner:v4.0.2","name":"nfs-client-provisioner","volumeMounts":[{"mountPath":"/persistentvolumes","name":"nfs-client-root"}]}],"serviceAccountName":"nfs-client-provisioner","volumes":[{"name":"nfs-client-root","nfs":{"path":"/mnt","server":"nfs.taikang1.local"}}]}}}}
  creationTimestamp: "2022-11-06T15:26:26Z"
  generation: 1
  name: nfs-client-provisioner
  namespace: nfs
  resourceVersion: "26216686"
  uid: deb54f37-144b-4490-b148-3229d7cf0adb
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nfs-client-provisioner
  strategy:
    type: Recreate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nfs-client-provisioner
    spec:
      containers:
      - env:
        - name: PROVISIONER_NAME
          value: k8s-sigs.io/nfs-subdir-external-provisioner
        - name: NFS_SERVER
          value: nfs.taikang1.local
        - name: NFS_PATH
          value: /mnt
        image: hub.taikang1.local/k8s/nfs-subdir-external-provisioner:v4.0.2
        imagePullPolicy: IfNotPresent
        name: nfs-client-provisioner
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /persistentvolumes
          name: nfs-client-root
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: nfs-client-provisioner
      serviceAccountName: nfs-client-provisioner
      terminationGracePeriodSeconds: 30
      volumes:
      - name: nfs-client-root
        nfs:
          path: /mnt
          server: nfs.taikang1.local

```



### 回收策略[ ](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/#reclaim-policy)

由 StorageClass 动态创建的 PersistentVolume 会在类的 `reclaimPolicy` 字段中指定回收策略，可以是 `Delete` 或者 `Retain`。如果 StorageClass 对象被创建时没有指定 `reclaimPolicy`，它将默认为 `Delete`。

通过 StorageClass 手动创建并管理的 PersistentVolume 会使用它们被创建时指定的回收策略



- #### 创建由storageclass 自动供应的pvc

```shell
检查default sc

# kubectl get sc
NAME                   PROVISIONER   RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client (default)   nfs           Delete          Immediate           false                  18d

#设置default class
kubectl  patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'


kubectl apply -f pvc.yaml

#检查pvc/pv状态，是否处于bound状态
kubectl get pvc

kubectl get pv
```

- pvc.yaml

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-vol
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
  #storageClassName: gp2
```

建立一个deployment使用pv

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: test-vol
          persistentVolumeClaim:
            claimName: test-vol
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: test-vol
          mountPath: /usr/share/nginx/html
```



- #### 创建手工供应的pvc/pv (无缺省storageclass时,使用NAS/NFS时)

```shell
仍使用上面的deployment,使用手在nfs上的手工建立目录的方式建立pv, chmod 777 xxx

＃先确认集群没有default storageclass,如果有，可以暂去掉storageclass 的default的annotation
kubectl  patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"False"}}}'

kubectl apply -f pvc-2.yaml


＃检查pvc的状态，
kubectl get pvc 

kubectl apply -f pv-2.yaml


＃再检查pvc的状态，
kubectl get pvc 
```

- pvc-2.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-vol
spec:
  resources:
    requests:
      storage: 10Mi
  accessModes:
  - ReadWriteOnce
```

- pv-2.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-vol
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Mi
  nfs:
    path: /mnt/static-vol
    server: nfs.taikang1.local
  persistentVolumeReclaimPolicy: Delete
  
  

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: test-vol
          persistentVolumeClaim:
            claimName: test-vol2
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: test-vol
          mountPath: /usr/share/nginx/html
```



 **在nfs上手工建立 的pv，当删除deployment, pvc, pv时，nfs上的目录 不会被 自动删除。即使pv中使用了Delete的ReclaimPolicy,也不会被 删除。**

下次重新使用此目录 时，数据会一直在。适合数据的复用。



nfs version 3 or 4 option

https://access.redhat.com/solutions/5846071

in ocp , By default coreOS tries to mount with `NFSv4`. The issue generally occurs because of `NFS version mismatch` during the mount between NFS server and NFS clien

```
piVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=3        # Replace the supported NFS version which is offered by the NFS server. 
  nfs:
    path: /tmp
    server: 10.X.X.X  
```

