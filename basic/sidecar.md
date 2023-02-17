https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/logging/





- ### 边车容器，常用于主容器的日志收集

```
oc new-project test-side

oc apply -f sidecar.yaml

＃此demo中，假定主容器的日志输出到文件中，在边车容器中将日志中的内容输出到console,以便于EFK自动收集。
两个容器，共同挂载同一个卷，实现数据的共享。

＃到pod中边车容器中，显示其console的输出内容

```



无边车的pod

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: counter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: counter
  template:
    metadata:
      labels:
        app: counter
    spec:
      containers:
      - name: count
        image: busybox:1.28
        args:
        - /bin/sh
        - -c
        - >
          i=0;
          mkdir -p /var/log;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            echo "$(date) INFO $i" >> /var/log/2.log;
            i=$((i+1));
            sleep 1;
          done  
```



- sidecar.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: counter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: counter
  template:
    metadata:
      labels:
        app: counter
    spec:
      containers:
      - name: count
        image: busybox:1.28
        args:
        - /bin/sh
        - -c
        - >
          i=0;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            echo "$(date) INFO $i" >> /var/log/2.log;
            i=$((i+1));
            sleep 1;
          done      
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      - name: count-log-1
        image: busybox:1.28
        args: [/bin/sh, -c, 'tail -n+1 -F /var/log/1.log']
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        emptyDir: {}

```



##### 练习,再给上面的配置加另一个sidecar,显示/var/log/2.log的日志到console





```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: counter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: counter
  template:
    metadata:
      labels:
        app: counter
    spec:
      containers:
      - name: count
        image: busybox:1.28
        args:
        - /bin/sh
        - -c
        - >
          i=0;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            echo "$(date) INFO $i" >> /var/log/2.log;
            i=$((i+1));
            sleep 1;
          done      
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      - name: count-log-1
        image: busybox:1.28
        args: [/bin/sh, -c, 'tail -n+1 -F /var/log/1.log']
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      - name: count-log-2
        image: busybox:1.28
        args: [/bin/sh, -c, 'tail -n+1 -F /var/log/2.log']
        volumeMounts:
        - name: varlog
          mountPath: /var/log    
      volumes:
      - name: varlog
        emptyDir: {}
```

可参考： https://openliberty.io/blog/2020/05/19/log4j-openshift-container-platform.html



或者下面的例子

```
apiVersion: apps/v1    
kind: Deployment       
metadata:
  labels:
    app: log-collection
  name: log-collection 
spec:
  replicas: 1
  selector:
    matchLabels:
      app: log-collection
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: log-collection
    spec:
      containers:
      - image: quay.io/qxu/log-collection:v3
        imagePullPolicy: IfNotPresent
        name: log-collection
        volumeMounts:
          - name: outputlog
            mountPath: /usr/local/logs
      - name: app-sidecar
        image: 'linyusha/java-microprofile:latest'
        args:
          - /bin/sh
          - '-c'
          - tail -n+1 --retry -f /usr/local/logs/log*.log
        volumeMounts:
          - name: outputlog
            mountPath: /usr/local/logs

      volumes:
        - name: outputlog
          emptyDir: {}

      dnsPolicy: ClusterFirst
      restartPolicy: Always

```



另一种不用边车的方式,使用link将日志到 /dev/stdout

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: counter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: counter
  template:
    metadata:
      labels:
        app: counter
    spec:
      containers:
      - name: count
        image: busybox:1.28
        args:
        - /bin/sh
        - -c
        - >
          i=0;
          mkdir -p /var/log;
          ln -s /dev/stdout /var/log/1.log;
          ln -s /dev/stdout /var/log/2.log;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            echo "$(date) INFO $i" >> /var/log/2.log;
            i=$((i+1));
            sleep 1;
          done  
```

