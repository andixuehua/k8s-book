https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/service-accounts-admin/

## 用户账号与服务账号[ ](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/service-accounts-admin/#user-accounts-versus-service-accounts)

Kubernetes 区分用户账号和服务账号的概念，主要基于以下原因：

- 用户账号是针对人而言的。而服务账号是针对运行在 Pod 中的进程而言的。
- 用户账号是全局性的。其名称在某集群中的所有名字空间中必须是唯一的。服务账号是名字空间作用域的。
- 通常情况下，集群的用户账号可能会从企业数据库进行同步，其创建需要特殊权限， 并且涉及到复杂的业务流程。 服务账号创建有意做得更轻量，允许集群用户为了具体的任务创建服务账号以遵从权限最小化原则。
- 对人员和服务账号审计所考虑的因素可能不同。
- 针对复杂系统的配置包可能包含系统组件相关的各种服务账号的定义。 因为服务账号的创建约束不多并且有名字空间域的名称，这种配置是很轻量的。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

在openshift 下,以上部署出出错,需要给一个特别的权限



```
oc adm policy add-scc-to-user anyuid system:serviceaccount:<NAMESPACE>:default

或
oc adm policy add-scc-to-user anyuid -z default
```



使用指定的sa启动容器:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
      serviceAccountName: nginxuser 
```





```
[root@qxu role]# kubectl create sa nginxuser 
serviceaccount/nginx-user created

[root@qxu role]# kubectl get sa
NAME         SECRETS   AGE
builder      2         145m
default      2         145m
deployer     2         145m
nginx-user   2         6s

```



```
oc adm policy add-scc-to-user anyuid  system:serviceaccount:a-1:nginxuser
```



 检查:

```
# kubectl get pod -n test -w
NAME                              READY   STATUS    RESTARTS   AGE
http-stress-75774bfbfb-2lz2n      1/1     Running   1          13h
log-collection-6c454dc7f8-dl4t8   1/1     Running   1          13h
whoami-8565878d9f-4q7b2           1/1     Running   3          23h
nginx-deployment-68878448b5-7qrh2   0/1     Pending   0          0s
nginx-deployment-68878448b5-7qrh2   0/1     Pending   0          0s
nginx-deployment-68878448b5-7qrh2   0/1     ContainerCreating   0          0s
nginx-deployment-68878448b5-7qrh2   1/1     Running             0          13s


```

