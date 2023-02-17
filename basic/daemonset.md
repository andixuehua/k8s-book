https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/

# DaemonSet

**DaemonSet** 确保全部（或者某些）节点上运行一个 Pod 的副本。 当有节点加入集群时， 也会为他们新增一个 Pod 。 当有节点从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

DaemonSet 的一些典型用法：

- 在每个节点上运行集群守护进程
- 在每个节点上运行日志收集守护进程
- 在每个节点上运行监控守护进程

一种简单的用法是为每种类型的守护进程在所有的节点上都启动一个 DaemonSet。 一个稍微复杂的用法是为同一种守护进程部署多个 DaemonSet；每个具有不同的标志， 并且对不同硬件类型具有不同的内存、CPU 要求。





demo daemonset的删除 ，以及master节点的部署



不能使用scale 删除 pod

```
如下，无效
kubectl scale daemonset/fluentd --replicas=0  -n logging

可使用以下方式临时删除：

kubectl -n logging  patch daemonset fluentd -p '{"spec": {"template": {"spec": {"nodeSelector": {"non-existing": "true"}}}}}'

kubectl -n logging patch daemonset fluentd --type json -p='[{"op": "remove", "path": "/spec/template/spec/nodeSelector/non-existing"}]'



```

注意下面的 tolerations: 如果 不加此配置，无法在master节点上运行pod



```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```



```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

```





以下也可以用于演示和测试

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: test
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # 这些容忍度设置是为了让该守护进程集在控制平面节点上运行
      # 如果你不希望自己的控制平面节点运行 Pod，可以删除它们
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```





```
kubectl get daemonset -A

NAMESPACE        NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR              AGE
elastic-system   filebeat-filebeat             3         3         3       3            3           <none>                     99d
ingress-nginx    ingress-nginx-controller      3         3         3       3            3           kubernetes.io/os=linux     100d
kube-system      calico-node                   6         6         6       6            6           kubernetes.io/os=linux     100d
kube-system      kube-multus-ds-amd64          6         6         6       6            6           kubernetes.io/arch=amd64   100d
kube-system      kube-proxy                    6         6         6       6            6           kubernetes.io/os=linux     100d
kube-system      nodelocaldns                  6         6         6       6            6           kubernetes.io/os=linux     100d
monitoring       pm-prometheus-node-exporter   6         6         6       6            6           <none>                     99d



```

