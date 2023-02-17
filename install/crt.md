kubeadm 是 kubernetes 提供的一个初始化集群的工具，使用起来非常方便。但是它创建的apiserver、controller-manager,ETCD等证书默认只有一年的有效期，同时kubelet 证书也只有一年有效期，一年之后 kubernetes 将停止服务。



在安装前，一定要手工生成证书，换成10年以上的证书。



证书查看：

https://www.cnblogs.com/kuku0223/p/11867391.html

使用kubeadm创建完Kubernetes集群后, 默认会在/etc/kubernetes/pki目录下存放集群中需要用到的证书文件, 整体结构如下图所示：









证书过期替换操作：

https://blog.csdn.net/ywq935/article/details/88355832

