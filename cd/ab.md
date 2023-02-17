- #### 金丝雀（AB）部署的实现,基于权重

```

oc patch route/log-collection --patch '{"spec": {"alternateBackends": [{"kind": "Service","name": "log-collection-b","weight": 100}]}}'  -n demo-icbc-ab-project




apiVersion: v1
kind: Namespace
metadata:
  name: demo-icbc-ab-project

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection-a
  namespace: demo-icbc-ab-project
spec:
  selector:
    matchLabels:
      app: log-collection-a
  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection-a
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
  name: log-collection-b
  namespace: demo-icbc-ab-project
spec:
  selector:
    matchLabels:
      app: log-collection-b
  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection-b
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
  name: log-collection-a
  namespace: demo-icbc-ab-project
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection-a
  type: ClusterIP
---

apiVersion: v1
kind: Service
metadata:
  name: log-collection-b
  namespace: demo-icbc-ab-project
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection-b
  type: ClusterIP
  
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection-a
  namespace: demo-icbc-ab-project
spec:
  ingressClassName: nginx 
  rules:
  - host: "ab-icbc.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection-a
            port:
              number: 8080
---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection-b
  namespace: demo-icbc-ab-project
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "30"
spec:
  ingressClassName: nginx 
  rules:
  - host: "ab-icbc.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection-b
            port:
              number: 8080



---
#old version
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: log-collection-a
  namespace: demo-icbc-ab-project
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: ab-icbc.apps.test.crc.test
    http:
      paths: 
      - path:
        backend:
          serviceName: log-collection-a
          servicePort: 8080
---
              
              
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: log-collection-b
  namespace: demo-icbc-ab-project
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "30"
spec:
  rules:
  - host: ab-icbc.apps.test.crc.test
    http:
      paths:
      - path:
        backend:
          serviceName: log-collection-b
          servicePort: 8080
```

refer to: https://www.tencentcloud.com/document/product/457/38413





调整权重

```
  kubectl get pod -n demo-icbc-ab-project
    kubectl get ingress -n demo-icbc-ab-project
    
  kubectl patch ing/log-collection-b  --type=json \
  -p='[{"op": "replace", "path": "/metadata/annotations/nginx.ingress.kubernetes.io~1canary-weight", "value":"10"}]' -n demo-icbc-ab-project
  
  
  
  
  for i in {1..10}; do curl   -L -w http://ab-icbc.apps.taikang1.local; done;
  
  Success! MasterSuccess! MasterSuccess! MasterSuccess! test branchSuccess! MasterSuccess! MasterSuccess! MasterSuccess! MasterSuccess! MasterSuccess! Master
```

基于header的灰度发布,基于上面应用,只需要建立两个ingress 



```
apiVersion: v1
kind: Namespace
metadata:
  name: demo-icbc-gray-project

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection-a
  namespace: demo-icbc-gray-project
spec:
  selector:
    matchLabels:
      app: log-collection-a
  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection-a
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
  name: log-collection-b
  namespace: demo-icbc-gray-project
spec:
  selector:
    matchLabels:
      app: log-collection-b
  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection-b
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
  name: log-collection-a
  namespace: demo-icbc-gray-project
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection-a
  type: ClusterIP
---

apiVersion: v1
kind: Service
metadata:
  name: log-collection-b
  namespace: demo-icbc-gray-project
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection-b
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection-a
spec:
  ingressClassName: nginx 
  rules:
  - host: "gray-icbc.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection-a
            port:
              number: 8080

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection-b
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "Region"
    nginx.ingress.kubernetes.io/canary-by-header-pattern: "cd"
spec:
  ingressClassName: nginx 
  rules:
  - host: "gray-icbc.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection-b
            port:
              number: 8080
              
              
----

#old k8s version
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: log-collection-a
  namespace: demo-icbc-gray-project
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: gray-icbc.apps.test.crc.test
    http:
      paths: 
      - path:
        backend:
          serviceName: log-collection-a
          servicePort: 8080
---



apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: log-collection-b
  namespace: demo-icbc-gray-project
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "Region"
    nginx.ingress.kubernetes.io/canary-by-header-pattern: "cd"
spec:
  rules:
  - host: gray-icbc.apps.test.crc.test
    http:
      paths:
      - path:
        backend:
          serviceName: log-collection-b
          servicePort: 8080
```



测试,先不带header

```
kubectl get pod -n demo-icbc-gray-project
kubectl get ingress -n demo-icbc-gray-project

curl  -w "\n" gray-icbc.apps.taikang1.local

for i in {1..10}; do curl -L -w "\n" -H "Region: cd" http://gray-icbc.apps.taikang1.local; done;

#测试带header, 带其他的header
 curl -H "Region: cd" gray-icbc.apps.taikang1.local
Success! test branch[root@kubernetes-master ~/cicd/jenkins] ens33 = 192.168.146.

# curl -H "Region: cd" gray-icbc.apps.taikang1.local
Success! test branch[root@kubernetes-master ~/cicd/jenkins] ens33 = 192.168.146.91

# curl -H "Region: xy" gray-icbc.apps.taikang1.local

```



