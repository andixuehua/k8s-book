- #### 部署一个mysql应用

```shell

 cat mysql.txt 
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: mysql
  labels:
    name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      name: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        name: mysql
        app: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-pv-claim
  labels:
    app: mysql
spec:
  accessModes:
   - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  type: ClusterIP
  ports:
    - port: 3306
  selector:
    app: mysql

```



- #### 部署mysql的exporter,收集监控数据 ,

- mysql exporter 有版本要求。参考https://github.com/prometheus/mysqld_exporter

- 此示例使用单独的部署方式，也可使用sidecar访问部署，将exporter与mysql集成到一个部署中

```shell
cat mysql-exporter.txt 
apiVersion: v1
kind: Service
metadata:
  name: mysqld-exporter
  labels:
    app: mysqld-exporter
spec:
  type: ClusterIP
  ports:
    - name: http-metrics
      port: 9104
  selector:
    app: mysqld-exporter

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysqld-exporter
  labels:
    app: mysqld-exporter
spec:
  selector:
    matchLabels:
      app: mysqld-exporter
  replicas: 1
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9104"
      labels:
        app: mysqld-exporter
    spec:
      containers:
      - name: mysqld-exporter
        image: prom/mysqld-exporter
        env:
        - name: DATA_SOURCE_NAME
          value: root:password@(mysql:3306)/
        ports:
        - containerPort: 9104
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mysqld-exporter
  labels:
    monitor-app: mysqld-exporter
    release: pm
spec:
  jobLabel: k8s-app
  selector:
    matchLabels:
      app: mysqld-exporter
  namespaceSelector:
    any: true
  endpoints:
  - port: http-metrics
    interval: 30s


```

以上两个yaml都在本地nm test1 进行部署就可以。

- 以上参考来自： https://docs.rackspace.com/docs/rkaas/v2.1.x/external/rkaas-userguide/rkaas-logging-mon/deploy-kubernetes-application
- 区别在于，它的sm部署在monitoring,我们的部在本地test1 namespace





```shell
检查UI 中的target和service discovery

多等一下，就会有
```

在prometheus中查询 mysql的指标，如果 发现很多，说明正常。如果只有几个，说明不正常。



- #### 检查target已经up

![image-20210217112535352](http://localhost:4000/monitor/image-20210217112535352.png)



- #### 检查对应的mysql指标要以查询

![image-20210217112652180](http://localhost:4000/monitor/image-20210217112652180.png)



grafana template: 

7362 for mysql

![](/home/qxu/Documents/k8sbook-new/monitor/k8s-mysql.png)