kubectl label node worker{1..3}.taikang1.local node-role.kubernetes.io/worker=





- #### 调度到指定节点

```shell
oc project test-node
oc apply -f selector-node.yaml

oc get pod -o wide
```

- selector-node.yaml



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 
  template:
    metadata:
      labels:
        app:  log-collection
    spec:
      nodeName: worker2.taikang1.local
      containers:
        - name:  log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
```



#### 亲和性和反亲和性

https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/



1. ***软性要求*--不作这个实验**
2. *# 如果节点上的pod标签存在满足app=log-collection，也可以部署到节点上，尽可能先部署到其它节点，如果没有满足也可以部署到此节点*

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 2
  template:
    metadata:
      labels:
        app:  log-collection
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - log-collection
              topologyKey: "kubernetes.io/hostname"
      containers:
        - name:  log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
```





1. *硬性要求*

2. *# 如果节点上的pod标签存在满足app=logcollection，则不能部署到节点上*

   **做这个实验**　

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 4
  template:
    metadata:
      labels:
        app:  log-collection
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - log-collection
            topologyKey: "kubernetes.io/hostname"
      containers:
        - name:  log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
```



查看pod

```
  Warning  FailedScheduling  31s (x2 over 31s)  default-scheduler  0/3 nodes are available: 1 node(s) didn't match pod affinity/anti-affinity, 1 node(s) didn't match pod anti-affinity rules, 1 node(s) had taint 

```





#### 节点管理

https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/safely-drain-node/

```
 kubectl cordon  kubernetes-worker2

 kubectl uncordon  kubernetes-worker2
 
  kubectl drain  kubernetes-worker2


kubectl label node worker02 node-role.kubernetes.io/worker=
```



新安装的master有taint

```
[root@master01 install]# kubectl describe node master01
Name:               master01
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=master01
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/cri-dockerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.146.51/24
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 07 Oct 2022 18:58:54 +0800
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
                    node-role.kubernetes.io/master:NoSchedule

```





```yaml
kubectl taint nodes --all node-role.kubernetes.io/master-

```
