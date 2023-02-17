- #### 使用集群部署策略

  

[部署参考](../basic/deployment.md)

```shell
#1 deploy below yaml, and change the pod number to 4, to observe the update process, you can find the rollingupdate strategy
change the image: quay.io/qxu/log-collection:main

change the image: quay.io/qxu/log-collection:123 (the wrong image)

change the maxSurge，maxUnavailable to 50%, to observe the pod status

#2 access the http url during the update

#3 change the strategy to Recreate, to observer the update process
  strategy:
    type: Recreate

```



deploy.yaml

```yaml
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
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%          
```



maxSurge，maxUnavailable

参考： https://blog.51cto.com/14034751/2494054?source=drh