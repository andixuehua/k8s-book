---
typora-copy-images-to: ./
---

- 部署一个java应用,此应用已经有/actuator/prometheus接口，显示prometheus监控数据
- 这里的svc要有label,后面servicemonitor要用， port要有name


```
# cat dep.txt 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection
    spec:
      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: log-collection
  labels:
    app: log-collection
    monitor: activity
spec:
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection
spec:
  rules:
  - host: log-collection.user1.apps.taikang1.local
  #- host: "www.log.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection
            port:
              number: 8080


            
＃测试url，及对应监控数据接口, /actuator/prometheus
＃检查prometheus中，此时不存在JVM对应的数据


```



- 如何在现有java项目中集成此jvm exporter,参考 http://micrometer.io/  和 https://dzone.com/articles/monitoring-using-spring-boot-2-prometheus-and-graf



- 在test1　namespace 中建立ServiceMonitor,注意下面matchLabels的配置,要和svc 中的lables匹配上.
-  这里的port: 8080-tcp 要和 svc 中的port name一致。

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: jvmmonitor 
  labels:
    k8s-app: jvmmonitor
  namespace: test1
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      app: log-collection
  endpoints:
    - interval: 30s
      path: /actuator/prometheus
      port: 8080-tcp
  
----
   
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: jvmmonitor 
  labels:
    k8s-app: jvmmonitor
  namespace: test1
spec:
  endpoints:
  - interval: 30s
    port: 8080-tcp
    scheme: http
    path: /actuator/prometheus
  selector:
    matchLabels:
      monitor: activity
      

      
```



```shell
# kubectl get servicemonitor -n test1

check target from promethues UI
应该发现不了上面的jvm target:
#授权prometheus service account可以访问你的应用的项目

kubectl logs -f prometheus-k8s-0 -c prometheus -n monitoring

level=error ts=2022-03-04T08:33:42.419Z caller=klog.go:94 component=k8s_client_runtime func=ErrorDepth msg="/app/discovery/kubernetes/kubernetes.go:361: Failed to list *v1.Endpoints: endpoints is forbidden: User
 \"system:serviceaccount:monitoring:prometheus-k8s\" cannot list resource \"endpoints\" in API group \"\" at the cluster scope"


这里要创建一个clusterrole:
#apiVersion: rbac.authorization.k8s.io/v1beta1
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources:
  - configmaps
  verbs: ["get"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
- nonResourceURLs: ["/actuator/prometheus"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus-k8s
  namespace: monitoring


＃然后再检查 发现promethues UI中target 和service discovery中已经有了jvm但是，没有打到对应的endpoints,可能是配置问题
一般是svc 的label和port与servicemonitor不匹配导致。



```



------



#create a service monitor in openshift-monitoring project,注意，这里的endpoints端口不是上面那个9799 ,而是应用的端口

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: jvmmonitor-2 
  labels:
    k8s-app: jvmmonitor-2
  namespace: openshift-monitoring
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      app: log-collection-git
  endpoints:
    - interval: 30s
      port: 8080-tcp

```



grafana template:  12856

如果grafana的页面总是刷新的放在,请点右上那个-的图标,使用绝对时间



在grafana中建立 template

```shell
12856  for jvm
7362 for mysql
```



![](/home/qxu/Documents/k8sbook-new/monitor/k8s-jvm.png)





![](/home/qxu/Documents/k8sbook-new/monitor/k8s-target.png)



taikang training, add label in servicemonitor:      release: pm

```
[root@trainee monitor]# cat cr.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources:
  - configmaps
  verbs: ["get"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
- nonResourceURLs: ["/actuator/prometheus"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: pm-kube-prometheus-stack-alertmanager
  namespace: monitoring

- kind: ServiceAccount
  name: pm-kube-prometheus-stack-operator
  namespace: monitoring

- kind: ServiceAccount
  name: pm-kube-prometheus-stack-prometheus
  namespace: monitoring


apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: jvmmonitor 
  labels:
    k8s-app: jvmmonitor
    release: pm

spec:
  endpoints:
  - interval: 30s
    port: 8080-tcp
    scheme: http
    path: /actuator/prometheus
  selector:
    matchLabels:
      monitor: activity

```





taikang granfa : view user: test / test123456
