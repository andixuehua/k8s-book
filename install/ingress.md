v1.19 nginx ingress controller： 安装 1.0.0可以成功,可使用nodeport或hostnetwork都测试 成功 OK
https://github.com/kubernetes/ingress-nginx



v1.24  v1.40 

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/cloud/deploy.yaml

修改为国内镜像(三个),

```
root@master01 install]# grep image ingress-controller-1.4.0-hostnetwork.yaml
        #image: registry.k8s.io/ingress-nginx/controller:v1.4.0@sha256:34ee929b111ffc7aa426ffd409af44da48e5a0eea1eb2207994d9e0c0882d143
        image: dyrnq/ingress-nginx-controller:v1.4.0@sha256:34ee929b111ffc7aa426ffd409af44da48e5a0eea1eb2207994d9e0c0882d143
        imagePullPolicy: IfNotPresent
        image: dyrnq/kube-webhook-certgen:v20220916-gd32f8c343@sha256:39c5b2e3310dc4264d638ad28d9d1d96c4cbb2b2dcfb52368fe4e3c63f61e10f
        imagePullPolicy: IfNotPresent
        image: dyrnq/kube-webhook-certgen:v20220916-gd32f8c343@sha256:39c5b2e3310dc4264d638ad28d9d1d96c4cbb2b2dcfb52368fe4e3c63f61e10f
        imagePullPolicy: IfNotPresent

```

使用hostnetwork:如下

```
    spec:
      dnsPolicy: ClusterFirst
      hostNetwork: true
      containers:
        image: dyrnq/ingress-nginx-controller:v1.4.0@sha256:34ee929b111ffc7aa426ffd409af44da48e5a0eea1eb2207994d9e0c0882d143



```

在ansible安装 目录 中查看对应的配置

```
# more ingress-ng-controller-addon.yml
---
- name: install addon
  hosts: kubernetes_master_nodes
  tasks:
    - name: copy nginx ingress controller yaml
      copy:
        src: ./ingress-1.0-hostnetwork.txt
        dest: /tmp/ingress.yml
    - name: install nginx ingress controller
      shell: kubectl apply -f /tmp/ingress.yml

```

检查

```
[root@master01 test]# kubectl get pod -n ingress-nginx -o wide
NAME                                        READY   STATUS      RESTARTS      AGE   IP               NODE       NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-kdxrr        0/1     Completed   0             33m   10.244.5.53      worker01   <none>           <none>
ingress-nginx-admission-patch-tr2l8         0/1     Completed   0             33m   10.244.5.54      worker01   <none>           <none>
ingress-nginx-controller-66497d9556-j865s   1/1     Running     1 (26m ago)   33m   192.168.146.54   worker01   <none>           <none>


```

配置haproxy,将80/443配置到192.168.146.54   的主机上



如果想安装 在固定的node上，请先modify yaml,加入node label的配置







配置4层端口映射：

参考https://pj1987111.github.io/posts/k8s/nginx+ingress-controller%E8%A7%A3%E5%86%B3l7%E5%A4%96%E7%BD%91web%E6%9C%8D%E5%8A%A1%E6%9A%B4%E9%9C%B2%E5%92%8C%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/

https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/

测试未通过 ，(mysql ingress要不要建立？ingress controller的部署方式有没有限定，hostnetwork or nodeport?)



##  配置TCP的configmap

还记得之前nginx-ingress-controller.yaml配置文件中的参数`--tcp-services-configmap=$(POD_NAMESPACE)/tcp-services`吗，这是对应TCP端口映射的。 编辑端口转发配置文件tcp-services-configmap.yaml写入下面内容，其中name和namespace对应–tcp-services-configmap中的参数：

POD_NAMESPACE为你ingress-controller 所在的namespace, 

configmap name: 为tcp-services，固定 的

test1/mysql:3306：为test1 namespace下的mysql svc, svc port 为3306

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
data:
  13306: "test1/mysql:3306"
```







## **default IngressClass**

可以将一个特定的 IngressClass 标记为集群默认IngressClass。 将一个 IngressClass 资源的 `ingressclass.kubernetes.io/is-default-class` 注解设置为 true, 将确保新的未指定 ingressClassName 字段的 Ingress 能够分配为这个默认的 IngressClass.

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
```
