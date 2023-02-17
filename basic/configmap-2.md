https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/



# Secret

Secret 是一种包含少量敏感信息例如密码、令牌或密钥的对象。 这样的信息可能会被放在 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 规约中或者镜像中。 使用 Secret 意味着你不需要在应用程序代码中包含机密数据。

由于创建 Secret 可以独立于使用它们的 Pod， 因此在创建、查看和编辑 Pod 的工作流程中暴露 Secret（及其数据）的风险较小。 Kubernetes 和在集群中运行的应用程序也可以对 Secret 采取额外的预防措施， 例如避免将机密数据写入非易失性存储。

Secret 类似于 [ConfigMap](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-pod-configmap/) 但专门用于保存机密数据。

**注意：**

默认情况下，Kubernetes Secret 未加密地存储在 API 服务器的底层数据存储（etcd）中。 任何拥有 API 访问权限的人都可以检索或修改 Secret，任何有权访问 etcd 的人也可以。 此外，任何有权限在命名空间中创建 Pod 的人都可以使用该访问权限读取该命名空间中的任何 Secret； 这包括间接访问，例如创建 Deployment 的能力。

为了安全地使用 Secret，请至少执行以下步骤：

1. 为 Secret [启用静态加密](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/encrypt-data/)。
2. 以最小特权访问 Secret 并[启用或配置 RBAC 规则](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authorization/)。
3. 限制 Secret 对特定容器的访问。
4. [考虑使用外部 Secret 存储驱动](https://secrets-store-csi-driver.sigs.k8s.io/concepts.html#provider-for-the-secrets-store-csi-driver)。

有关管理和提升 Secret 安全性的指南，请参阅 [Kubernetes Secret 良好实践](https://kubernetes.io/zh-cn/docs/concepts/security/secrets-good-practices)。

参见 [Secret 的信息安全](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#information-security-for-secrets)了解详情。

## Secret 的使用

Pod 可以用三种方式之一来使用 Secret：

- 作为挂载到一个或多个容器上的[卷](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/) 中的[文件](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)。

- 作为[容器的环境变量](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)。

- 由 [kubelet 在为 Pod 拉取镜像时使用](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#using-imagepullsecrets)。

  

#### 挂载的 Secret 是被自动更新的[ ](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#mounted-secrets-are-updated-automatically)

当卷中包含来自 Secret 的数据，而对应的 Secret 被更新，Kubernetes 会跟踪到这一操作并更新卷中的数据。更新的方式是保证最终一致性。

如果容器已经在通过环境变量来使用 Secret，Secret 更新在容器内是看不到的， 除非容器被重启。有一些第三方的解决方案，能够在 Secret 发生变化时触发容器重启。

对于以 [subPath](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes#using-subpath) 形式挂载 Secret 卷的容器而言， 它们无法收到自动的 Secret 更新。



```shell
kubectl create secret generic admin-access --from-file=user.txt --from-file=passwd.txt

#update
kubectl create secret generic admin-access --from-file=user.txt --from-file=passwd.txt --dry-run -o yaml|kubectl apply -f - -n test

oc apply -f secret-dep.yaml

查看
# kubectl get secret admin-access -o yaml -n test1
apiVersion: v1
data:
  passwd.txt: MTIzNDUK
  user.txt: YWRtaW4K
kind: Secret
metadata:
  name: admin-access

type: Opaque
```

- user.txt and passwd.txt

```shell
[root@qxu configmap]# cat user.txt 
admin

[root@qxu configmap]# cat passwd.txt 
123456



```

- secret-dep.yaml,将两个文件 挂到/data/access/目录 下

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: log-collection-2
  labels:
    app: log-collection-2
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: log-collection-2
  template:
    metadata:
      labels:
        deployment: log-collection-2
    spec:
      volumes:
        - name: secret-volume
          secret:
            secretName: admin-access
            defaultMode: 0440
      containers:
        - name: log-collection-2
          image: >-
            quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
              protocol: TCP
          resources: {}
          volumeMounts:
            - name: secret-volume
              mountPath: /data/access
              #subPath: access.ini
              #readOnly: true
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
```



将secret map到pod内的变量

使用secret部署mysql时，指定mysql的口令：

```
echo -n "redhat" | base64

```



```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.7
          ports:
            - containerPort: 3306
          volumeMounts:
            - mountPath: "/var/lib/mysql"
              subPath: "mysql"
              name: mysql-data
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: ROOT_PASSWORD
      volumes:
        - name: mysql-data
          persistentVolumeClaim:
            claimName: mysql-data-disk
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-data-disk
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
      
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secrets
type: Opaque
data:
  ROOT_PASSWORD: cmVkaGF0   
```



测试:

```
# kubectl exec -it mysql-deployment-694787d87b-vnhb9 /bin/sh

sh-4.2# mysql -u root -predhat       
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.39 MySQL Community Server (GPL)


```



更新测试 mysql 的password,环境变量的password不生效,要重启pod才可以

```
# echo -n admin |base64
YWRtaW4=

kubectl edit secret mysql-secrets -n test
```



