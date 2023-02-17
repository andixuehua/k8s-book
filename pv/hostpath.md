https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/

### hostPath

**警告：**

HostPath 卷存在许多安全风险，最佳做法是尽可能避免使用 HostPath。 当必须使用 HostPath 卷时，它的范围应仅限于所需的文件或目录，并以只读方式挂载。

如果通过 AdmissionPolicy 限制 HostPath 对特定目录的访问，则必须要求 `volumeMounts` 使用 `readOnly` 挂载以使策略生效。

`hostPath` 卷能将主机节点文件系统上的文件或目录挂载到你的 Pod 中。 虽然这不是大多数 Pod 需要的，但是它为一些应用程序提供了强大的逃生舱。

例如，`hostPath` 的一些用法有：

- 运行一个需要访问 Docker 内部机制的容器；可使用 `hostPath` 挂载 `/var/lib/docker` 路径。
- 在容器中运行 cAdvisor 时，以 `hostPath` 方式挂载 `/sys`。
- 允许 Pod 指定给定的 `hostPath` 在运行 Pod 之前是否应该存在，是否应该创建以及应该以什么方式存在。

除了必需的 `path` 属性之外，你可以选择性地为 `hostPath` 卷指定 `type`。

支持的 `type` 值如下：



挂载本地目录 到pod中, /tmp/nginx不用提前 建立 

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: test-vol
          hostPath:
            path: /tmp/nginx
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: test-vol
          mountPath: /usr/share/nginx/html

```



以上适合，不用重启pod就可以生效的配置，比如静态文件 



测试:

到pod所在的主机的/tmp/ngnix目录下建立文件

然后到pod中查看文件,可以内外实时同步数据

