* Docker

  * [常用命令](docker/basic.md)
  * [配置文件](docker/config.md)
  * [Dockerfile](docker/dockerfile.md)
  * [Registry Server](docker/registry.md)
  
* cri-o

  * [cri-o](cri-o/cri-o.md)
  * [crictl](cri-o/crictl.md)

* k8s架构

  * [架构](arch/arch.md)
  * [设计理念](arch/concept.md)
  * [etcd](arch/etcd.md)
  * [开放式接口](arch/api.md)
  * [资源对象](arch/resource.md)

* 查看安装配置及访问
  
  * [kubeconfig](config/kubeconfig.md)
  * [kubectl](basic/kubectl.md)
  * [API](config/api.md)
  * [核心服务组件](config/component.md)
  * [网络](config/network.md)
  * [证书](config/ca.md)
  
* 基本配置

  * [label & annoation](basic/label.md)
  * [namespace](basic/namespace.md)
  * [配置registry的访问](basic/registry.md)
  * pod
    * [pod](basic/pod.md)
    * [init-container](basic/init-container.md)
    * [sidecar](basic/sidecar.md)
    * [生命周期lifecycle](basic/lifecycle.md)
    * [health检查](basic/health.md)
    * [static pod](basic/static.md)
  * controller
    * [ReplicasSet](basic/replicaset.md)
    * [deployment](basic/deployment.md)
    * [statefulset](basic/statefulset.md)
    * [daemonset](basic/daemonset.md)
    * [job & cronjob](basic/job.md)
  * 配置挂载
    * [挂载configmap文件](basic/configmap-1.md)
    * [挂载secret](basic/configmap-2.md)
    * [挂载空目录](pv/empty.md)
  * [Service/svc](basic/svc.md)

* 网络

  * [coreDNS/kubeDNS/pod DNS/DNS policy](network/dns.md)
  * [ingress controller/class](network/controller.md)
  * [应用ingress的建立](network/ingress.md)
  * [ingress/会话保持](network/route.md)
  * [hostnetwork部署](network/hostnetwork.md)
  * [paused容器](network/paused.md)

* pod高级资源与调度
  
  * [按label调度](resource/label.md)
  * [按节点调度亲和性](resource/node.md)
  * [污点与容忍](resource/taint.md)
  * [node管理](resource/node.md)
  * [limitrange](resource/limits.md)
  * [配额](resource/quota.md)
  * [hpa](basic/hpa.md)
  * [downwardapi](basic/downwardapi.md)
  
* 用户与权限

  * [身份认证Authenticating](user/auth.md)
  * [user/role/rolebinding](user/user.md)
  * [serviceaccount](user/sa.md)
  * [security context](user/scc.md)
  * [networkpolicy](network/policy.md)

* 存储
  
  * [pvc/pv/storageclass/手工建立](pv/pvc.md)
  * [卷的挂载和读写模式](pv/rwo.md)
  * [卷的扩容](pv/pv-extend.md)
  * [卷数据的保留和复用](pv/claimpolicy.md)
  * [statefulset的卷](pv/statefulset.md)
  * [demployment的卷](pv/deployment.md)
  * [hostpath](pv/hostpath.md)
  * [mysql pv](pv/mysql.md)
  
* 监控
  
  * [集群的监控](install/monitor.md)
  * 应用的监控
    * [JVM的监控](monitor/jvm.md)
    * [mySQL的监控](monitor/mysql.md)
  * [自定义报警](monitor/alert.md)
  * [promethues指南](monitor/prometheus.md)
  * 报警规则梳理
  
* 日志

  * 收集
    * [EFK部署](install/efk.md)
    * [filebeat sidecar](log/filebeat-sidecar.md)

* CI

  * [CICD规划](jenkins/prelog.md)
  * 使用Jenkins
    * [规划](jenkins/plan.md)
    * [安装](jenkins/install.md)
    * [配置jenkins](jenkins/plugin.md)
    * [job构建示例](jenkins/job.md)
    * [pipeline声明式基础](jenkins/basic.md)
    * [piepline示例](jenkins/pipeline.md)
    * [Jenkinsfile](jenkins/jenkinsfile.md)
    * [使用sharedlibrary进行构建](jenkins/sharedlib.md)
    * [jenkins与k8s的集成配置](jenkins/k8s.md)
  * gitlab CI
    * [安装gitlab/runner](gitlab/install.md)
    * [pipeline基础](gitlab/basic.md)
    * [runner流水线](gitlab/pipeline.md)
    * [CI/CD](gitlab/ci.md)
    * [关联k8s](gitlab/k8s.md)
  * [Dockerfile](docker/dockerfile.md)
  * [案例](ci/case.md)

* CD
  
  * [应用的建立的基本方式](cd/new-app.md)
  * [部署策略](cd/policy.md)
  * [蓝绿](cd/bluegreen.md)
  * [金丝雀](cd/ab.md)
  * [使用Jenkins发布](cd/jenkins-cd.md)
  * [使用argoCD](cd/argocd.md)
  
* Helm
  
  * [架构](helm/basic.md)
  * [模板语法](helm/arch.md)
  * [部署命令](helm/detail.md)
  
* operator
  
  * [operator](operator/operator.md)
  
* 从ocp到k8s

  * [应用迁移](migrate/app.md)

* API调用
  
  * Python Lib
  
* 前端vue部署

  * [vue frontend](frontend/vue.md)
  * [django api backend](frontend/django.md)

  

* 基础安装
  
  * [阿里源替换,doker](install/docker.md)
  * [离线yum repo/httpd](install/repo.md)
  * [dnsmasq](install/dns.md)
  * [haproxy](install/ha.md)
  * [keepalvied](install/keepalived.md)
  * [registry](basic/registry.md)
  * [harbor]
  
* 安装

  * [单master测试环境 ](install/single-1.19.md)
  * [多master生产环境](install/ha.md)
  * [kubeadm token/ca](install/token.md)
  * 离线安装
  * 安装前的证书规划
  * 1.21以上containered替换安装
  * Network
  * [metric server](install/metric.md)
  * [nginx ingress controller](install/ingress.md)
  * [dashboard](install/dashboard.md)
  * [Prometheus/grafana](install/monitor.md)
  * [NFS storageclass](install/nfs.md)
  * [EFK](install/efk.md)
  * [ETCD](install/etcd.md)

* Day2配置
  
  * [ntp](day2/ntp.md) 
  * [basic user]
  * [ladp/AD]
  
* Upgrade
  
  * [离线升级集群](upgrade/cluster.md)
  * [证书更新](upgrade/ca.md)

- servicemesh
  
  - [安装](istio/control.md)
  - [bookinfo 演示](istio/bookinfo.md)

