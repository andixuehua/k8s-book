## 虚机运行master

java -jar jenkins.war

java version "1.8.0_311"

/root/jenkins.war

apache-maven-3.5.4-bin.tar.gz





## k8s内运行master

https://devopscube.com/setup-jenkins-on-kubernetes-cluster/



使用下面的PVC,不使用仓库里的volume.yaml

```
# cat pvc.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pv-claim
  namespace: devops-tools
spec:
  #storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```



另,创建一个ingress,如果不想使用nodeport

```

apiVersion: v1
kind: Service
metadata:
  name: jenkins-service2
  namespace: devops-tools
  annotations:
      prometheus.io/scrape: 'true'
      prometheus.io/path:   /
      prometheus.io/port:   '8080'
spec:
  selector:
    app: jenkins-server
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
      
      
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: jenkins
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: jenkins.devops-tools.apps.test.crc.test
    http:
      paths:
      - path:
        backend:
          serviceName: jenkins-service2
          servicePort: 8080
```



测试,口令在　pod内的/var/jenkins_home/secrets/initialAdminPassword

安装部分plugin



![](/home/qxu/Documents/k8sbook-new/jenkins/plugin.png)



进入后,将安全性改为任何人可以做任何事.

