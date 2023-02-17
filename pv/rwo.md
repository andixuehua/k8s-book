https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/

### 访问模式

PersistentVolume 卷可以用资源提供者所支持的任何方式挂载到宿主系统上。 如下表所示，提供者（驱动）的能力不同，每个 PV 卷的访问模式都会设置为对应卷所支持的模式值。 例如，NFS 可以支持多个读写客户，但是某个特定的 NFS PV 卷可能在服务器上以只读的方式导出。 每个 PV 卷都会获得自身的访问模式集合，描述的是特定 PV 卷的能力。

访问模式有：

- `ReadWriteOnce`

  卷可以被一个节点以读写方式挂载。 ReadWriteOnce 访问模式也允许运行在同一节点上的多个 Pod 访问卷。

- `ReadOnlyMany`

  卷可以被多个节点以只读方式挂载。

- `ReadWriteMany`

  卷可以被多个节点以读写方式挂载。

- `ReadWriteOncePod`

  卷可以被单个 Pod 以读写方式挂载。 如果你想确保整个集群中只有一个 Pod 可以读取或写入该 PVC， 请使用 ReadWriteOncePod 访问模式。这只支持 CSI 卷以及需要 Kubernetes 1.22 以上版本。

这篇博客文章 [Introducing Single Pod Access Mode for PersistentVolumes](https://kubernetes.io/blog/2021/09/13/read-write-once-pod-access-mode-alpha/) 描述了更详细的内容。

在命令行接口（CLI）中，访问模式也使用以下缩写形式：

- RWO - ReadWriteOnce
- ROX - ReadOnlyMany
- RWX - ReadWriteMany
- RWOP - ReadWriteOncePod





- #### 挂载volume到pod中使用，以及 RWO模式的使用限制

  - accessModes由sc的性质决定，在pvc中必须指定，可以降级使用。如sc 支持rwx， pvc中可以使用rwo ,但是pod仍然会部署在多node上

- [先完成pv/pvc的实验](../pv/pvc.md)

- 创建部署使用volume

```shell
oc project test-vol

oc apply -f test-deploy.yaml

＃测试数据的持久性
#进入pod中，在/data 目录下写入文件，保存。删除pod后，再进入pod,检查/data目录下的文件的内容是否存在。

＃增加pod 的数量，先到2个； 再超过worker 的数量。
oc get pod -o wide

＃解答 当使用RWO volume时，所有的pod只能部署在同一个node上,不能调度到其他节点上(如果 是nfs存储 ，如果 在pvc中定义了rwo,所有的podb也可以部署在多个node 上，因为nfs支持rwx	)
```

- test-deploy.yaml

```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: log-collection
  labels:
    app: log-collection
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: log-collection
  template:
    metadata:
      labels:
        deployment: log-collection
    spec:
      volumes:
        - name: test-vol
          persistentVolumeClaim:
            claimName: test-vol

      containers:
        - name: log-collection
          image: >-
            quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
              protocol: TCP
          resources: {}
          volumeMounts:
            - name: test-vol
              mountPath: /data/
```



scale pod and check pod

```
kubectl scale deployment nginx --replicas=3

NAME                     READY   STATUS    RESTARTS   AGE     IP              NODE                     NOMINATED NODE   READINESS GATES
nginx-6fdd485f88-ghzmx   1/1     Running   0          49s     10.233.66.75    worker2.taikang1.local   <none>           <none>
nginx-6fdd485f88-jm9dm   1/1     Running   0          3h37m   10.233.73.64    worker1.taikang1.local   <none>           <none>
nginx-6fdd485f88-psbkn   1/1     Running   0          49s     10.233.64.136   worker3.taikang1.local   <none>           <none>

```

参考： https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/