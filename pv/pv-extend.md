

- #### volume的扩容

- [先完成卷的挂载](../pv/rwo.md)



```shell
# kubectl get pvc -n test1
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
test-vol   Bound    pvc-8523504b-00c9-4d69-9513-258fe5475dd8   10Mi       RWO            nfs-client     28


＃修改原来pvc的配置，增加声明的容量
kubectl project test-vol
kubectl apply -f pvc.yaml


＃观察pvc, pv的容量是否新增,如果有pod，此时pv, pvc的size不会发生变化
＃如果有pod存在，要pod数量为0后，才会重新挂载生效
kubectl scale deployment nginx --replicas=0 -n test1
kubectl scale deployment nginx --replicas=1 -n test1
＃检查pod中 /data/ 目录中，原来的数据，还会存在，不会丢失
```



- 修改后的 extend-pvc.yaml  : storage: 10Mi -> 20Mi

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
      storage: 20Mi
  #storageClassName: gp2
  volumeMode: Filesystem
```



注意：  gp2 格式只能扩展一次容量， NFS实际不支持扩容。

```
modify pvc.yaml的size后，重新apply,如果 发现以下报错，说明sc不支持动态扩容

for: "extend-pvc.yaml": persistentvolumeclaims "test-vol" is forbidden: only dynamically provisioned pvc can be resized and the storageclass that provisions the pvc must support r
esize

，可以查看 get sc -yaml,有没有以下配置，如果没有就edit sc加入
allowVolumeExpansion: true

然后扩容pvc, kubectl apply -f extend-pvc.yaml
可以成功，但pv/pvc的size 不发生变化，scale pod=0后，也不发现变化。
```

此步骤可参考：

https://www.cnblogs.com/zphqq/p/13330732.html

