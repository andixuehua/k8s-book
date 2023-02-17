https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/

# Deployments

一个 Deployment 为 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 和 [ReplicaSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/) 提供声明式的更新能力。

你负责描述 Deployment 中的 **目标状态**，而 Deployment [控制器（Controller）](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/) 以受控速率更改实际状态， 使其变为期望状态。你可以定义 Deployment 以创建新的 ReplicaSet，或删除现有 Deployment， 并通过新的 Deployment 收养其资源。





### 访问本地带认证的registry

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
spec:
  selector:
    matchLabels:
      app: whoami
  replicas: 1
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: hub.taikang1.local/spring-boot-whoami:latest
          ports:
            - containerPort: 8080
```



使用自签 证书时，pod拉取镜像里会报错，

```
x509: certificate signed by unknown authority
```

#### 请参考[这里](registry.md)中的第二个方式，在每个节点上建立对应的目录 ，cp registry的crt文件，不用重启dockery,就可以正常login到自签的registry：



如果 registry带用户认证，会报以下错误,

```
Failed to pull image "registry.cr
c.test:5000/test/spring-boot-whoami:latest": rpc error: code = Unknown desc = Error response from daemon: Get "
https://registry.crc.test:5000/v2/": dial tcp 192.168.146.90:5000: connect: connection refused
```



#### 参考[这里](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/),在每个需要访问registry的ns中创建对应的secret，

```
kubectl create secret docker-registry regcred --docker-server=hub.taikang1.local --docker-username=admin --docker-password=redhat -n test
```

修改deployment配置，重新部署，可以正常接取

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
spec:
  selector:
    matchLabels:
      app: whoami
  replicas: 1
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: hub.taikang1.local/spring-boot-whoami:latest
          ports:
            - containerPort: 8080
      imagePullSecrets:
      - name: regcred

```



### 命令行方式建立 ，deployment, svc, ingress



更新

```
kubectl set image deployment/log-collection log-collection=quay.io/qxu/log-collection:test -n test

kubectl scale deployment/log-collection --replicas=10
kubectl get pod -n test -w

```



### 策略[ ](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#strategy)

`.spec.strategy` 策略指定用于用新 Pod 替换旧 Pod 的策略。 `.spec.strategy.type` 可以是 “Recreate” 或 “RollingUpdate”。“RollingUpdate” 是默认值。

修改此配置,查看pod的建立过程



yaml deployment:

```
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

```



deployment中的containerPort可以省略,或者写错也可以,只要svc中的targetport写对就可以

          ports:
            - containerPort: 8080

svc:

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

```
kubectl create ns test
kubectl create deployment whoami --image=quay.io/qxu/spring-boot-whoami -n test

kubectl expose deployment whoami --port=8080 --target-port=8080 -n test


```



独立域名的ingress,注意在1.24+中ingress 的api version: networking.k8s.io/v1

```
cat ingress-whoami.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami
spec:
  ingressClassName: nginx 
  rules:
  - host: "whoami.crc.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: whoami
            port:
              number: 8080
```





泛域名的ingress

v1/  tested in v1.24

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami
spec:
  ingressClassName: nginx 
  rules:
  - host: "whoami.user1.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: whoami
            port:
              number: 8080

```



```
[root@master01 test]# kubectl get ingress -n test1
NAME     CLASS   HOSTS                      ADDRESS   PORTS   AGE
whoami   nginx   whoami.apps.test.icbc.io             80      4m5s

[root@master01 test]# curl whoami.apps.test.icbc.io
<div style="text-align:center;"><h1>Who Am I - whoami-6458676f5f-fd9jj</h1></div><div style="margin-left:10%"><br/><br/>Available processors (cores): 1<br/><br/>Free memory (bytes): 221468136<br/>Maximum memory 
(bytes): 4005888000<br/>Total memory available to JVM (bytes): 251527168<br/><br/>File system root: /<br/>Total space (bytes): 75125227520<br/>Free space (bytes): 67584208896<br/>Usable space (bytes): 6758420889
6<br/><br/>Network Details:<br/>Display name: eth0              InetAddress: /10.244.5.4<br/>Display name: lo           InetAddress: /127.0.0.1</div>[root@master01 test]# 

[root@master01 test]# kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.5", GitCommit:"e979822c185a14537054f15808a118d7fcce1d6e", GitTreeState:"clean", BuildDate:"2022-09-14T16:42:36Z", GoVersion:"go1.18.6", Compi
ler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.4

```



注意上面的ingress里面，kubernetes.io/ingress.class: "nginx"很重要，如果 没有，ingress访问 就不会正常。

可用以下命令查看 ，你当前的 ingress class name

```
kubectl get ingressclass

NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       22d

```

