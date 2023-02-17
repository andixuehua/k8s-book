

https://github.com/prometheus-operator/kube-prometheus/

安装时,注意版本兼容, 1.24->0.11.0



prometheus: 1.19.5 安装0.6.0可以成功： OK

参考：

https://github.com/prometheus-operator/kube-prometheus/tree/release-0.6



安装：

```
wget https://github.com/prometheus-operator/kube-prometheus/archive/refs/tags/v0.11.0.zip
unzip v0.11.0.zip
kubectl create -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl create -f manifests/

如果上面出错,可使用　kubectl replace  -f manifests/setup

如果k8s.gcr.iog　下载失败,使用下面的替换
sed -i '/image:/s@k8s.gcr.io/kube-state-metrics@dyrnq/kube-state-metrics@' $(grep -l image: manifests/*.yaml)
sed -i '/image:/s@k8s.gcr.io/prometheus-adapter@willdockerhub@' $(grep -l image: manifests/*.yaml)


开启对外访问:
kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090 --address 192.168.146.91

kubectl port-forward grafana-xxx -n monitoring 3000 --address 192.168.146.91

kubectl port-forward -n monitoring alertmanager-main-0 9093 --address  192.168.146.91
 
 
grafana ui: admin/admin
 
```



以下参考:https://zhuanlan.zhihu.com/p/469202200

添加存储配置,vi  prometheus-prometheus.yaml在最后加入

```

apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  labels:
    prometheus: k8s
  name: k8s
  namespace: monitoring
spec:
  alerting:
    alertmanagers:
    - name: alertmanager-main
      namespace: monitoring
      port: web
  image: quay.io/prometheus/prometheus:v2.20.0
  nodeSelector:
    kubernetes.io/os: linux
  podMonitorNamespaceSelector: {}
  podMonitorSelector: {}
  replicas: 2
  resources:
    requests:
      memory: 400Mi
  ruleSelector:
    matchLabels:
      prometheus: k8s
      role: alert-rules
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: prometheus-k8s
  serviceMonitorNamespaceSelector: {}
  serviceMonitorSelector: {}
  version: v2.20.0
  storage:
    volumeClaimTemplate:
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 20Gi

```



vi  alertmanager-alertmanager.yaml

```
apiVersion: monitoring.coreos.com/v1
kind: Alertmanager
metadata:
  labels:
    alertmanager: main
  name: main
  namespace: monitoring
spec:
  image: quay.io/prometheus/alertmanager:v0.21.0
  nodeSelector:
    kubernetes.io/os: linux
  replicas: 3
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: alertmanager-main
  version: v0.21.0
  storage:
    volumeClaimTemplate:
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 5Gi

```

增加:　grafana pvc:

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: grafana
  namespace: monitoring  #---指定namespace为monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```



vi  grafana-deployment.yaml

```
由:
      volumes:
      - emptyDir: {}
        name: grafana-storage
修改为:

volumes:        
- name: grafana-storage       # 新增持久化配置
  persistentVolumeClaim:
    claimName: grafana
```



配置alert notifiy,在这个文件

```
apiVersion: v1
data: {}
kind: Secret
metadata:
  name: alertmanager-main
  namespace: monitoring
stringData:
  alertmanager.yaml: |-
    "global":
      "resolve_timeout": "5m"
    "inhibit_rules":
    - "equal":
      - "namespace"
      - "alertname"
      "source_match":
        "severity": "critical"
      "target_match_re":
        "severity": "warning|info"
    - "equal":
      - "namespace"
      - "alertname"
      "source_match":
        "severity": "warning"
      "target_match_re":
        "severity": "info"
    "receivers":
    - "name": "Default"
    - "name": "Watchdog"
    - "name": "Critical"
    "route":
      "group_by":
      - "namespace"
      "group_interval": "5m"
      "group_wait": "30s"
      "receiver": "Default"
      "repeat_interval": "12h"
      "routes":
      - "match":
          "alertname": "Watchdog"
        "receiver": "Watchdog"
      - "match":
          "severity": "critical"
        "receiver": "Critical"
type: Opaque

```



也可建立 三个ingress: 以下都测试 通过 v19

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: prometheus.apps.test.crc.test
    http:
      paths: 
      - path: /
        backend:
          serviceName: prometheus-k8s
          servicePort: 9090

```



```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: grafana.apps.test.crc.test
    http:
      paths: 
      - path: /
        backend:
          serviceName: grafana
          servicePort: 3000
```



```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: alertmanager
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: alertmanager.apps.test.crc.test
    http:
      paths: 
      - path: /
        backend:
          serviceName: alertmanager-main
          servicePort: 9093
```





ingress v1:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus
spec:
  rules:
  - host: "prometheus.apps.test.icbc.io"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: prometheus-k8s
            port:
              number: 9090
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alertmanager
spec:
  rules:
  - host: "alertmanager.apps.test.icbc.io"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: alertmanager-main
            port:
              number: 9093
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
spec:
  rules:
  - host: "grafana.apps.test.icbc.io"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: grafana
            port:
              number: 3000
```



查看配置

```
# kubectl get ingress -n monitoring
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME           CLASS    HOSTS                             ADDRESS          PORTS   AGE
alertmanager   <none>   alertmanager.apps.test.crc.test   192.168.146.92   80      209d
grafana        <none>   grafana.apps.test.crc.test        192.168.146.92   80      209d
prometheus     <none>   prometheus.apps.test.crc.test     192.168.146.92   80      209d

```





查看系统的prometheusrules,此部分为alert rules,和ocp中差不多

```
# kubectl get prometheusrules -A
NAMESPACE    NAME                   AGE
monitoring   jvm-alert-rules        208d
monitoring   prometheus-k8s-rules   232d

kubectl get prometheusrules prometheus-k8s-rules -n monitoring -o yaml

```

cm prometheus-k8s-rulefiles-0 为配置文件

```
# kubectl get cm -n monitoring
NAME                                                  DATA   AGE
adapter-config                                        1      232d
grafana-dashboard-apiserver                           1      232d
grafana-dashboard-cluster-total                       1      232d
grafana-dashboard-controller-manager                  1      232d
grafana-dashboard-k8s-resources-cluster               1      232d
grafana-dashboard-k8s-resources-namespace             1      232d
grafana-dashboard-k8s-resources-node                  1      232d
grafana-dashboard-k8s-resources-pod                   1      232d
grafana-dashboard-k8s-resources-workload              1      232d
grafana-dashboard-k8s-resources-workloads-namespace   1      232d
grafana-dashboard-kubelet                             1      232d
grafana-dashboard-namespace-by-pod                    1      232d
grafana-dashboard-namespace-by-workload               1      232d
grafana-dashboard-node-cluster-rsrc-use               1      232d
grafana-dashboard-node-rsrc-use                       1      232d
grafana-dashboard-nodes                               1      232d
grafana-dashboard-persistentvolumesusage              1      232d
grafana-dashboard-pod-total                           1      232d
grafana-dashboard-prometheus                          1      232d
grafana-dashboard-prometheus-remote-write             1      232d
grafana-dashboard-proxy                               1      232d
grafana-dashboard-scheduler                           1      232d
grafana-dashboard-statefulset                         1      232d
grafana-dashboard-workload-total                      1      232d
grafana-dashboards                                    1      232d
prometheus-k8s-rulefiles-0                            2      208d

```

以上配置也可以在UI中看到



grafana中默认的dashboard与OCP中基本一样

缺少ETCD方面的数据,

可查看API方面的数据,

resource/cluster

datasource





## pushgateway

https://blog.51cto.com/root/3033785

### 1 pushgateway的概念

客户端或者服务端安装pushgateway插件，被监控端使用运维自行开发的各种脚本把监控数据组织成K/V的形式 metrics形式发送给pushgateway，之后prometheus来pushgateway端拉取相关采集指标数据。
与exporter相反，pushgateway相当于prometheus与被监控端之间的代理，pushgateway只负责被动接收客户端运行脚本发送过来的metrics，不负责对客户端进行探测，主动采集。

### 2 pushgateway的利弊

##### （1）利

exporter虽然采集类型很丰富，但是我们依然需要很多自制的监控数据

exporter由于数据类型采集量大，其实很多数据我们监控中用不到，用pushgateway定制一项数据就节省一份采集资源

exporter虽然数据很丰富，但是依然无法提供一些我们需要的采集形式，使用pushgateway就可以使采集的数据形式任意灵活

一个新的自定义pushgateway脚本比开发一个全新的exporter简单快速

##### （2）弊

将多个节点数据汇总到 pushgateway, 如果 pushgateway 挂了，受影响比多个 target 大。
Prometheus 拉取状态 up 只针对 pushgateway, 无法做到对每个节点有效。

Pushgateway 可以持久化推送给它的所有监控数据，因此，即使你的监控已经下线，prometheus 还会拉取到旧的监控数据，需要手动清理 pushgateway 不要的数据




# prometheus监控k8s集群系列之kube-state-metrics

https://code84.com/839889.html

kube-state-metrics对k8s集群的监控，那它主要是监控哪些内容的呢？我们先看一下官方的介绍

```
kube-state-metrics is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects. 
(See examples in the Metrics section below.) It is not focused on the health of the individual Kubernetes components, but rather on 
the health of the various objects inside, such as deployments, nodes and pods.
```

从官方介绍中，我们可以知道，kube-state-metrics并不聚焦于k8s集群整体及相关组件的监控，而是更加关注deployments, nodes and pods等内部对象的状态。

例如：

- 现在计划了多少个副本？目前有多少可用？
- 有多少个 Pod 正在运行、已停止、已终止？
- 此 Pod 已重启多少次？

指标类型概览：

- CertificateSigningRequest Metrics
- ConfigMap Metrics
- CronJob Metrics
- DaemonSet Metrics
- Deployment Metrics
- Endpoint Metrics
- Horizontal Pod Autoscaler Metrics
- Ingress Metrics
- Job Metrics
- Lease Metrics
- LimitRange Metrics
- MutatingWebhookConfiguration Metrics
- Namespace Metrics
- NetworkPolicy Metrics
- Node Metrics
- PersistentVolume Metrics
- PersistentVolumeClaim Metrics
- Pod Disruption Budget Metrics
- Pod Metrics
- ReplicaSet Metrics
- ReplicationController Metrics
- ResourceQuota Metrics
- Secret Metrics
- Service Metrics
- StatefulSet Metrics
- StorageClass Metrics
- ValidatingWebhookConfiguration Metrics
- VerticalPodAutoscaler Metrics
- VolumeAttachment Metrics







### proemtheus opeartor介绍:



https://mp.weixin.qq.com/s?__biz=MzIzMTYzMTY0OQ==&mid=2247483744&idx=1&sn=e7169de660391c4e84c3d27387d80e39&chksm=e8a07f2edfd7f638d554e14abc1cf4ea3b89c66913e28d97e3daa7d72b8f91001102d9bb4aac&mpshare=1&scene=24&srcid=0127lbISjQvH5LUPqc5PuJfL&sharer_sharetime=1674791103059&sharer_shareid=1b071e140471a5427f1c43ba67735110&ascene=14&devicetype=android-29&version=28001759&nettype=cmnet&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=AF&exportkey=n_ChQIAhIQ3VVw3JYIvw78sRxdwQJa6RLbAQIE97dBBAEAAAAAAHWGAMTbTxcAAAAOpnltbLcz9gKNyK89dVj0LrqxO89%2FHIFXS3wCcsjWNvfJ1RqvWZMXPDTeon2S110qafdMDh2Q%2FITnbOhNHQYHJyZmGSBfGW9QBF9fgWgXNprEg0iG5lew61ajU1Lp0QEqpiKYr8NyvcFEXfmPuy4Kmq%2FdW%2Fg6rEGxRhqdqDoh768gbBH8r15nnkPEXTL3XDjA6Qj2IptJzWKwBXVpATdEcJOvkZtfBq5b10T4R0WeTplv47cFfe6b%2Bmi3yBJ5GGc3GMfQNg%3D%3D&pass_ticket=KEon9IncUp1EvKJKhz92VjENn5SZ3EtDBTg6pIxBtcuv%2BCGq13YB1Q8v6RI3WKO9ErE7fxWwPrUJ7dsjVu0oyA%3D%3D&wx_header=3





servicemonitor作用

运维过程可能会时常修改监控配置项,通过ServiceMonitor自动化管理prometheus配置

ServiceMonitor 也是一个自定义资源，它描述了一组被 Prometheus 监控的 targets 列表。该资源通过 Labels 来选取对应的 Service Endpoint，让 Prometheus Server 通过选取的 Service 来获取 Metrics 信息。



![图片](https://mmbiz.qpic.cn/mmbiz_png/DJdMtz9Xiar87tjXRpTGg2SvCRPFLLicnuuTQJvn8B5YlWSY65hEdlY8BdNjYFhqUTOCaCWgrlIGvTthx6dQ9oSQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
