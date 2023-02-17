demo deployment中volume 的使用与statefulset中volume中的不同。

这里所有的pod使用的是同一个volume,

注意，后面的pv claim的建立不能省略：



```
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: ClusterIP

---
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
          persistentVolumeClaim:
            claimName: test-vol
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: test-vol
          mountPath: /usr/share/nginx/html

          

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-demo
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: nginx-demo2.apps.test.crc.test
    http:
      paths:
      - path:
        backend:
          serviceName: nginx
          servicePort: 80
          
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-vol
spec:
  accessModes:
    - ReadWriteOnce
  #storageClassName: fast
  resources:
    requests:
      storage: 10Mi
```



```
#in on nginx pod , touch cd /usr/share/nginx/html/index.html, echo differ content
```

