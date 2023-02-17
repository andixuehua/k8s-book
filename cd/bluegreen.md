

- #### 蓝绿部署的实现

```
＃curl route to get the result
oc get route


---
apiVersion: v1
kind: Namespace
metadata:
  name: demo-icbc-bluegreen-project

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection-blue
  namespace: demo-icbc-bluegreen-project
spec:
  selector:
    matchLabels:
      app: log-collection-blue
  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection-blue
    spec:
      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection:main
          ports:
            - containerPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection-green
  namespace: demo-icbc-bluegreen-project
spec:
  selector:
    matchLabels:
      app: log-collection-green

  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection-green
    spec:
      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection:latest
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: log-collection-blue
  namespace: demo-icbc-bluegreen-project
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection-blue
  type: ClusterIP
---

apiVersion: v1
kind: Service
metadata:
  name: log-collection-green
  namespace: demo-icbc-bluegreen-project
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection-green
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection-bluegreen
  namespace: demo-icbc-bluegreen-project
spec:
  ingressClassName: nginx 
  rules:
  - host: "bluegreen-icbc.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection-green
            port:
              number: 8080


----
#old k8s version
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: log-collection-bluegreen
  namespace: demo-icbc-bluegreen-project
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: bluegreen-icbc.apps.test.crc.test
    http:
      paths:
      - path:
        backend:
          serviceName: log-collection-green
          servicePort: 8080

```



切换service:

```

kubectl patch ing/log-collection-bluegreen  --type=json \
  -p='[{"op": "replace", "path": "/spec/rules/0/http/paths/0/backend/service/name", "value":"log-collection-blue"}]' -n demo-icbc-bluegreen-project
  
  kubectl patch ing/log-collection-bluegreen  --type=json \
  -p='[{"op": "replace", "path": "/spec/rules/0/http/paths/0/backend/service/name", "value":"log-collection-green"}]' -n demo-icbc-bluegreen-project
```

测试

```
curl bluegreen-icbc.apps.test.crc.test
```

