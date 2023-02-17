https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/

### emptyDir

当 Pod 分派到某个节点上时，`emptyDir` 卷会被创建，并且在 Pod 在该节点上运行期间，卷一直存在。 就像其名称表示的那样，卷最初是空的。 尽管 Pod 中的容器挂载 `emptyDir` 卷的路径可能相同也可能不同，这些容器都可以读写 `emptyDir` 卷中相同的文件。 当 Pod 因为某些原因被从节点上删除时，`emptyDir` 卷中的数据也会被永久删除。

**说明：** 容器崩溃并**不**会导致 Pod 被从节点上移除，因此容器崩溃期间 `emptyDir` 卷中的数据是安全的。

`emptyDir` 的一些用途：

- 缓存空间，例如基于磁盘的归并排序。
- 为耗时较长的计算任务提供检查点，以便任务能方便地从崩溃前状态恢复执行。
- 在 Web 服务器容器服务数据时，保存内容管理器容器获取的文件。

取决于你的环境，`emptyDir` 卷存储在该节点所使用的介质上；这里的介质可以是磁盘或 SSD 或网络存储。但是，你可以将 `emptyDir.medium` 字段设置为 `"Memory"`，以告诉 Kubernetes 为你挂载 tmpfs（基于 RAM 的文件系统）。 虽然 tmpfs 速度非常快，但是要注意它与磁盘不同。 tmpfs 在节点重启时会被清除，并且你所写入的所有文件都会计入容器的内存消耗，受容器内存限制约束。



示例１　

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection
    spec:
      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
          volumeMounts:
          - mountPath: /data
            name: cache-volume
      volumes:
      - name: cache-volume
        emptyDir: {}
```



测试,查看/data目录

```
# kubectl exec -it log-collection-84bb64f765-24b8l /bin/bash -n test
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@log-collection-84bb64f765-24b8l:/# ls /data
root@log-collection-84bb64f765-24b8l:/# df -h 
Filesystem               Size  Used Avail Use% Mounted on
overlay                   50G   11G   40G  22% /
tmpfs                     64M     0   64M   0% /dev
tmpfs                    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/mapper/centos-root   50G   11G   40G  22% /data
shm                       64M     0   64M   0% /dev/shm
tmpfs                    7.8G   12K  7.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                    7.8G     0  7.8G   0% /proc/acpi
tmpfs                    7.8G     0  7.8G   0% /proc/scsi
tmpfs                    7.8G     0  7.8G   0% /sys/firmware
```





示例２,两个container共享数据

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: counter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: counter
  template:
    metadata:
      labels:
        app: counter
    spec:
      containers:
      - name: count
        image: busybox:1.28
        args:
        - /bin/sh
        - -c
        - >
          i=0;
          mkdir -p /var/log;
          ln -s /dev/stdout /var/log/1.log;
          ln -s /dev/stdout /var/log/2.log;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            echo "$(date) INFO $i" >> /var/log/2.log;
            i=$((i+1));
            sleep 1;
          done    
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      - name: count-log-1
        image: busybox:1.28
        args: [/bin/sh, -c, 'tail -n+1 -F /var/log/1.log']
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        emptyDir: {}


```



```
kubectl exec -it counter-864f9c746-7ttzl -c count-log-1 /bin/sh 

 kubectl exec -it counter-864f9c746-7ttzl -c count /bin/sh 
```

