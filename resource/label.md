

- #### 调度到指定label

```shell
kubectl label node worker3.taikang1.local group=dev
kubectl get node -l group=dev --show-labels



kubectl apply -f selector-label.yaml

oc get pod -o wide

#删除节点的label
kubectl label node worker3.taikang1.local group-


kubectl get node -l group

```



- selector-label.yaml

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
      nodeSelector:
        group: dev
      containers:
        - name:  log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
```

