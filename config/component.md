架构图:

https://kubernetes.io/zh-cn/docs/concepts/overview/components/



查看集群状态：



```
# kubectl cluster-info
Kubernetes master is running at https://192.168.146.91:6443
KubeDNS is running at https://192.168.146.91:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'

# kubectl get cs #or componentstatus
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
etcd-0               Healthy     {"health":"true"}
```



master: api, proxy scheduler

```
docker ps|grep api
357e8a12a34f   72efb76839e7                                        "kube-apiserver --ad…"   2 hours ago   Up 2 hours             k8s_kube-apiserver_kube-apiserver-kubernetes-master_kube-system_483018e842ec09512d
a92c0b096b66e3_10
202ca1ed1afc   registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 2 hours ago   Up 2 hours             k8s_POD_kube-apiserver-kubernetes-master_kube-system_483018e842ec09512da92c0b096b6
6e3_10

# docker ps |grep controller
9535eef934cf   f196e958af67                                        "kube-controller-man…"   2 hours ago   Up 2 hours             k8s_kube-controller-manager_kube-controller-manager-kubernetes-master_kube-system_
062c848068a6bbb8b82922ce48f0a50b_12
88d28f144274   registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 2 hours ago   Up 2 hours             k8s_POD_kube-controller-manager-kubernetes-master_kube-system_062c848068a6bbb8b829
22ce48f0a50b_10
   

# docker ps|grep sche
08c50429e1cc   350a602e5310                                        "kube-scheduler --au…"   2 hours ago   Up 2 hours             k8s_kube-scheduler_kube-scheduler-kubernetes-master_kube-system_c2620add3cc3961167
e561442295f42c_11
67c3aa52ec77   registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 2 hours ago   Up 2 hours             k8s_POD_kube-scheduler-kubernetes-master_kube-system_c2620add3cc3961167e561442295f
42c_10


```



node and master:

proxy

```
 docker ps |grep proxy
dced90dea173   70eeaa7791f2                                        "./kube-rbac-proxy -…"   2 hours ago   Up 2 hours             k8s_kube-rbac-proxy_node-exporter-v7g4l_monitoring_b721bd62-b05c-4496-85f5-74fb7c3
7a34e_6
16e1a90f268d   6e5666d85a31                                        "/usr/local/bin/kube…"   2 hours ago   Up 2 hours             k8s_kube-proxy_kube-proxy-sjwxp_kube-system_8ee24ddf-62c3-427b-bad0-12619c8f62b5_1
0
98cfe6f7c388   registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 2 hours ago   Up 2 hours             k8s_POD_kube-proxy-sjwxp_kube-system_8ee24ddf-62c3-427b-bad0-12619c8f62b5_10 
```







kubelet

```
 systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Mon 2022-02-14 19:55:38 CST; 1h 35min ago
     Docs: https://kubernetes.io/docs/
 Main PID: 973 (kubelet)
    Tasks: 20
   Memory: 144.6M
   CGroup: /system.slice/kubelet.service
           └─973 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra...


```





etcd

```
# docker ps |grep etcd
ec8a75704dde   0369cf4303ff                                        "etcd --advertise-cl…"   2 hours ago   Up 2 hours             k8s_etcd_etcd-kubernetes-master_kube-system_cf456611d0b61f2aaec72d0382b7d7aa_10   
68f71a8a7738   registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 2 hours ago   Up 2 hours             k8s_POD_etcd-kubernetes-master_kube-system_cf456611d0b61f2aaec72d0382b7d7aa_10  
```



查看所有api object:

```
kubectl api-resources

NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
pods                                           metrics.k8s.io                 true         PodMetrics
alertmanagers                                  monitoring.coreos.com          true         Alertmanager
podmonitors                                    monitoring.coreos.com          true         PodMonitor
prometheuses                                   monitoring.coreos.com          true         Prometheus
prometheusrules                                monitoring.coreos.com          true         PrometheusRule
servicemonitors                                monitoring.coreos.com          true         ServiceMonitor
thanosrulers                                   monitoring.coreos.com          true         ThanosRuler
ingressclasses                                 networking.k8s.io              false        IngressClass
ingresses                         ing          networking.k8s.io              true         Ingress
networkpolicies                   netpol       networking.k8s.io              true         NetworkPolicy
runtimeclasses                                 node.k8s.io                    false        RuntimeClass
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io      true         RoleBinding
roles                                          rbac.authorization.k8s.io      true         Role
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
csidrivers                                     storage.k8s.io                 false        CSIDriver
csinodes                                       storage.k8s.io                 false        CSINode
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment

```



kubectl auto complete  https://kubernetes.io/docs/reference/kubectl/cheatsheet/

```
source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc 
```

