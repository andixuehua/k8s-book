此demo展示statfulset与deployment的数据卷的不同
在deployment中，每个pod共享使用同一个数据卷
在statefuleset中，每个pod中使用各自不同的数据卷



```
kubectl get pv 
kubectl get pvc -n test
查看会发现多个pv/pvc

#in each nginx pod , touch cd /usr/share/nginx/html/index.html, echo differ content
echo "$HOSTNAME" > /usr/share/nginx/html/index.html
cat  /usr/share/nginx/html/index.html

#try curl the ingess url, 将显示不同的内容， 因为会轮询到不同的pod上去
```




```

apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  #clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx 
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx 
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      #storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 10M
          
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  ingressClassName: nginx 
  rules:
  - host: "nginx-ful.user1.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: nginx
            port:
              number: 80
              

```



测试,返回不同的内容

```
nginx-demo.apps.test.crc.test
# curl nginx-demo.apps.test.crc.test
web-1
[root@kubernetes-master ~/practice/statefulset] ens33 = 192.168.146.91
# curl nginx-demo.apps.test.crc.test
web-0
[root@kubernetes-master ~/practice/statefulset] ens33 = 192.168.146.91
# curl nginx-demo.apps.test.crc.test
web-2
[root@kubernetes-master ~/practice/statefulset] ens33 = 192.168.146.91
#


```

