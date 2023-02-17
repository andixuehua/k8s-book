https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/downward-api/

# Downward API

对于容器来说，在不与 Kubernetes 过度耦合的情况下，拥有关于自身的信息有时是很有用的。 **Downward API** 允许容器在不使用 Kubernetes 客户端或 API 服务器的情况下获得自己或集群的信息。

例如，现有应用程序假设某特定的周知的环境变量是存在的，其中包含唯一标识符。 一种方法是对应用程序进行封装，但这很繁琐且容易出错，并且违背了低耦合的目标。 更好的选择是使用 Pod 名称作为标识符，并将 Pod 名称注入到周知的环境变量中。

在 Kubernetes 中，有两种方法可以将 Pod 和容器字段暴露给运行中的容器：

- 作为[环境变量](https://kubernetes.io/zh-cn/docs/tasks/inject-data-application/environment-variable-expose-pod-information/#the-downward-api)
- 作为 [`downwardAPI` 卷中的文件](https://kubernetes.io/zh-cn/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)

这两种暴露 Pod 和容器字段的方式统称为 **Downward API**。





```
oc new-project test-downapi
oc apply -f downward-api.yaml


#go to pod, run env command to show the environment vars
```



- downward-api.yaml

```yaml
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
      containers:
        - name: log-collection
          image: >-
            quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
              protocol: TCP
          resources: {}
          imagePullPolicy: IfNotPresent
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: MY_POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}

```



- 这里有所有可以传递的变量数据

  https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/#the-downward-api