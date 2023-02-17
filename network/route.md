- #### 路由层面会话保持

  建立 以下配置，在chrome中访问  ingress  地址，发现是两个pod轮流访问 ，加入affinity后再访问 ，发现只到一个ip了（一定要在浏览器中，不要在curl中访问 ）
  
  ```
  kubectl scale deployment whoami --replicas=2 
  ```
  
  

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
spec:
  selector:
    matchLabels:
      app: whoami
  replicas: 2
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: quay.io/qxu/spring-boot-whoami
          ports:
            - containerPort: 8080
            
---
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: whoami
  type: ClusterIP

---
                       
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami
  annotations:
    nginx.ingress.kubernetes.io/affinity: "cookie"
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

可参考： https://www.jianshu.com/p/ff0463ba7482



如果使用cli,加上一个自定义的cookie的k/v配置就可以看到,只返回一个pod ip  (tk环境不生效)

```
curl whoami.user1.apps.taikang1.local --cookie 'user=123'

```

自定义cookie名等 

```
  nginx.ingress.kubernetes.io/affinity: "cookie"
  nginx.ingress.kubernetes.io/affinity-mode: "persistent"
  nginx.ingress.kubernetes.io/session-cookie-name: "route"
```



路由层面会话保持

```

 #test in laptop console:
 curl route to see the different pod IP
```

svc层面会话保持

```
kubectl patch svc/whoami --patch '{"spec": {"sessionAffinity": "ClientIP"}}'  -n test1

取消
kubectl patch svc whoami -p '{"spec":{"sessionAffinity":"None"}}'    #取消session 


#test one pod's console to see the IP

curl whoami:8080
```

[Kubernetes学习之路（十四）之服务发现Service - 烟雨浮华 - 博客园](https://www.cnblogs.com/linuxk/p/9605901.html)
