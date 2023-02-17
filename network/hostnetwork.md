hostNetwork
hostNetwork设置对象为pod，当hostNetwork为true时，pod中的容器直接暴露在宿主机的网络环境中，可以直接通过宿主机的网络访问pod中的应用程序，即PodIp就是Node的IP。该模式下，每一个node只能启动一个同deployment的pod。



```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      hostNetwork: true
      containers:
      - image: mysql:5.6
        name: mysql
        env:
          # 在实际中使用 secret
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
```





```
# kubectl get pod -n test -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP               NODE                 NOMINATED NODE   READINESS GATES

mysql-75599f5b9f-fvzkb            1/1     Running   0          8s    192.168.146.92   kubernetes-worker1   <none>           <none>


#telnet 192.168.146.92  3306
```





```
[root@worker1 ~]# ss -ntlp|grep 3306
LISTEN 0      80                 *:3306             *:*    users:(("mysqld",pid=1705863,fd=10)) 
```

