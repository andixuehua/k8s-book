## nfs sc配置
下载https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner.git



```
替换 deployment.yml中为阿里image
images: registry.cn-beijing.aliyuncs.com/mydlq/nfs-subdir-external-provisioner:v4.0.0

# cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          #image: k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          image: registry.cn-beijing.aliyuncs.com/mydlq/nfs-subdir-external-provisioner:v4.0.0
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: nfs
            - name: NFS_SERVER
              value: 192.168.146.90
            - name: NFS_PATH
              value: /data/exports
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.146.90
            path: /data/exports


```

```
在 default ns中
kubectl apply -f rbac.yaml deployment.yaml class.yaml
check pod in default 

in test ns: the busybox test pod will exit after touch a file in nfs directory

kubectl apply -f test-claim.yaml -n test1
kubectl apply -f test-pod.yaml -n test1

kubectl get pvc -n test1
kubectl get pod -n test1
kubectl get pv
check pod status and dir in nfs server pod

#default class
kubectl  patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```



如果provision的pod起不来,有下面的error,

```
Output: mount: /var/lib/kubelet/pods/ee9a272e-602f-4f1f-966b-9848d3b11367/volumes/kubernetes.io~nfs/nfs-client-root: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helpe
r program.

```



需要节点上先安装,

yum install nfs-utils -y 

