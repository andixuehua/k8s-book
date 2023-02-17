https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers/



# Ingress 控制器

为了让 Ingress 资源工作，集群必须有一个正在运行的 Ingress 控制器。

与作为 `kube-controller-manager` 可执行文件的一部分运行的其他类型的控制器不同， Ingress 控制器不是随集群自动启动的。 基于此页面，你可选择最适合你的集群的 ingress 控制器实现。

Kubernetes 作为一个项目，目前支持和维护 [AWS](https://github.com/kubernetes-sigs/aws-load-balancer-controller#readme)、 [GCE](https://git.k8s.io/ingress-gce/README.md#readme) 和 [Nginx](https://git.k8s.io/ingress-nginx/README.md#readme) Ingress 控制器



## 使用多个 Ingress 控制器

你可以使用 [Ingress 类](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/#ingress-class)在集群中部署任意数量的 Ingress 控制器。 请注意你的 Ingress 类资源的 `.metadata.name` 字段。 当你创建 Ingress 时，你需要用此字段的值来设置 Ingress 对象的 `ingressClassName` 字段（请参考 [IngressSpec v1 reference](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/service-resources/ingress-v1/#IngressSpec)）。 `ingressClassName` 是之前的[注解](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/#deprecated-annotation)做法的替代。

如果你不为 Ingress 指定 IngressClass，并且你的集群中只有一个 IngressClass 被标记为默认，那么 Kubernetes 会将此集群的默认 IngressClass [应用](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/#default-ingress-class)到 Ingress 上。 IngressClass。 你可以通过将 [`ingressclass.kubernetes.io/is-default-class` 注解](https://kubernetes.io/zh-cn/docs/reference/labels-annotations-taints/#ingressclass-kubernetes-io-is-default-class) 的值设置为 `"true"` 来将一个 IngressClass 标记为集群默认。

理想情况下，所有 Ingress 控制器都应满足此规范，但各种 Ingress 控制器的操作略有不同。





## Ingress 类

Ingress 可以由不同的控制器实现，通常使用不同的配置。 每个 Ingress 应当指定一个类，也就是一个对 IngressClass 资源的引用。 IngressClass 资源包含额外的配置，其中包括应当实现该类的控制器名称。



```
# c
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       232d

 kubectl get pod -A|grep ingress
ingress-nginx                 ingress-nginx-controller-6kddq                         1/1     Running       6 (82m ago)    101d
ingress-nginx                 ingress-nginx-controller-nrg4p                         1/1     Running       6 (80m ago)    101d
ingress-nginx                 ingress-nginx-controller-xqtmb                         1/1     Running       6 (82m ago)    101d

```



### 默认 Ingress 类[ ](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/#default-ingress-class)

你可以将一个特定的 IngressClass 标记为集群默认 Ingress 类。 将一个 IngressClass 资源的 `ingressclass.kubernetes.io/is-default-class` 注解设置为 `true` 将确保新的未指定 `ingressClassName` 字段的 Ingress 能够分配为这个默认的 IngressClass.





```
kubectl get IngressClass -o yaml

apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  labels:
    app.kubernetes.io/component: controller
  name: nginx-example
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
```



```
kubectl get daemonset  -o yaml  -n ingress-nginx 
ports:
          - containerPort: 80
            hostPort: 80
            name: http
            protocol: TCP
          - containerPort: 443
            hostPort: 443
            name: https
            protocol: TCP
          - containerPort: 10254
            hostPort: 10254
            name: metrics
            protocol: TCP

     hostNetwork: true
        nodeSelector:
          kubernetes.io/os: linux

```

