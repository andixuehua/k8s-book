## label

https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/





## annotations

https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/annotations/

标签可以用来选择对象和查找满足某些条件的对象集合。 相反，注解不用于标识和选择对象。 注解中的元数据，可以很小，也可以很大，可以是结构化的，也可以是非结构化的，能够包含标签不允许的字符。



label dmeo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx
    ports:
    - containerPort: 80
    resources:
      limits:
        cpu: 50m
        memory: 50Mi
      requests:
        cpu: 50m
        memory: 50Mi
```



注意下面的matchLabels要和labels下的一致,不然会创建不成功

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      run: nginx
  replicas: 1
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          ports:
            - containerPort: 8080
```



用下面的yaml创建,会出错

* The Deployment "nginx" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"run":"nginx"}: `selector` does not match template `labels`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          ports:
            - containerPort: 8080

```



标签的例子后面介绍
