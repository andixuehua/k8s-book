#### 什么是 Static Pod

静态 Pod 在指定的节点上由 kubelet 守护进程直接管理，不需要 API 服务器 监管。 与由控制面管理的 Pod（例如，Deployment） 不同；kubelet 监视每个静态 Pod（在它崩溃之后重新启动）。

静态 Pod 永远都会绑定到一个指定节点上的 Kubelet。

kubelet 会尝试通过 Kubernetes API 服务器为每个静态 Pod 自动创建一个 镜像 Pod。 这意味着节点上运行的静态 Pod 对 API 服务来说是可见的，但是不能通过 API 服务器来控制。 Pod 名称将把以连字符开头的节点主机名作为后缀。

最常见的 Static Pod
etcd
kube-apiserver
kube-controller-manager
kube-scheduler

#### 运行

拷贝 yaml 到文件夹 /etc/kubernetes/manifests 即可只在本机自动运行



#### 为什么需要Static pod

Kubernetes官方文档，在介绍Static pod时，特别做了如下的标注说明：

Note: If you are running clustered Kubernetes and are using static Pods to run a Pod on every node, you should probably be using a DaemonSet instead.

也就是说，如果你在使用Static pod来实现kubernetes集群中每个Node的上启动Pod，那么你更应该使用DaemonSet，而不是Static pod。

既然官方文档都推荐使用DaemonSet了，为什么还存在static pod这种机制？

早期的Kubernetes，为了在集群各个节点上启动例如日志采集（fluentd）、网络组件(kube-proxy)等服务，使用了Static pod的机制。后来提出了DaemonSet的概念，于是这些需要在每个节点上都启动的服务，都逐渐被DaemonSet所替代，官方文档也建议优先选用DaemonSet。

Static pod机制一直保留下来，一方面是为了兼容广大开发者采用了Static pod的使用场景；另一方面则是Static pod具有DaemonSet无法替代的特性：不需要Kubernetes集群来直接启动、管理Pod。Static pod最大的特点是无需调用Kube-apiserver即可快速启动Pod，也就是说，不需要一个完整的kubernetes集群，只要安装了kubelet、容器运行时，即可快速地让kubelet来接管你的yaml文件定义的Pod。前面章节也简单介绍了如何快速使用这一特性。Static pod优点是不需要集群，缺点也是相对应的，没有集群层面的管理、调度功能。而Static pod最经典的使用场景，就是用来Bootstrap一个Kubernetes集群。


一般只在master上有以上四类static pod