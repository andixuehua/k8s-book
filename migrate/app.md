### DC的迁移

DC in OCP

```
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: log-collection
spec:
  selector:
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

```



deployment in K8S

```
apiVersion: apps/v1klja
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
```



注意以上三个不同:

apiVersion

kind

spec.selector.app   　vs   spec.selector.matchLabels.app　

#### 注意如果有镜像仓库的用户口令配置时的不同:

在k8s中需要在每个项目中配置secret,并在所有的deployment中加入对就的配置,可[参考](../basic/registry.md)



### route的迁移

router in OCP



```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: log-collection
spec:
  port:
    targetPort: 8080
  to:
    kind: Service
    name: log-collection
    weight: 100
```



ingress in K8S

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection
spec:
  rules:
  - host: "www.log.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection
            port:
              number: 8080
```

注意以上三个不同:

apiVersion

kind

spec:





svc相同

```
apiVersion: v1
kind: Service
metadata:
  name: log-collection
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection
  type: ClusterIP
```

