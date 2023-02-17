

https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/operator/



# kubernetes operator 简介

https://zhuanlan.zhihu.com/p/387457184

# Operator 模式

Operator 是 Kubernetes 的扩展软件， 它利用[定制资源](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/api-extension/custom-resources/)管理应用及其组件。 Operator 遵循 Kubernetes 的理念，特别是在[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller)方面。



**Operator 模式** 旨在记述（正在管理一个或一组服务的）运维人员的关键目标。 这些运维人员负责一些特定的应用和 Service，他们需要清楚地知道系统应该如何运行、如何部署以及出现问题时如何处理。

在 Kubernetes 上运行工作负载的人们都喜欢通过自动化来处理重复的任务。 Operator 模式会封装你编写的（Kubernetes 本身提供功能以外的）任务自动化代码。



## CRD是什么

全称Custom Resource Definition，顾名思义就是「自定义资源定义」，也就是按照官方Scheme来定义官方没有的资源struct，即创建你自己的“Pod”，Kubernetes提供了这样的入口就是方便用户扩展Kubernetes，适应更多使用场景。由于是官方配置，所以CRD有它自己的特点或者叫约束：

- 强类型

- 能够被订阅更新事件，本质上是让api server能够识别






查看资源

```
[root@qxu mysql]# kubectl get crd|grep oracle
innodbclusters.mysql.oracle.com                                   2022-11-07T13:36:28Z
mysqlbackups.mysql.oracle.com                                     2022-11-07T13:36:29Z


kubectl api-resources

kubectl api-resources|grep -i mysql

[root@qxu mysql]# kubectl api-resources|grep -i mysql
innodbclusters                        ic,ics              mysql.oracle.com/v2                           true         InnoDBCluster
mysqlbackups                          mbk                 mysql.oracle.com/v2                           true         MySQLBackup



kubectl api-resources|grep operator
clusteroperators                      co                  config.openshift.io/v1                        false        ClusterOperator
operatorhubs                                              config.openshift.io/v1                        false        OperatorHub
podnetworkconnectivitychecks                              controlplane.operator.openshift.io/v1alpha1   true         PodNetworkConnectivityCheck
configs                                                   imageregistry.operator.openshift.io/v1        false        Config
imagepruners                                              imageregistry.operator.openshift.io/v1        false        ImagePruner
dnsrecords                                                ingress.operator.openshift.io/v1              true         DNSRecord
egressrouters                                             network.operator.openshift.io/v1              true         EgressRouter
operatorpkis                                              network.operator.openshift.io/v1              true         OperatorPKI
authentications                                           operator.openshift.io/v1                      false        Authentication
cloudcredentials                                          operator.openshift.io/v1                      false        CloudCredential
clustercsidrivers                                         operator.openshift.io/v1                      false        ClusterCSIDriver
configs                                                   operator.openshift.io/v1                      false        Config
consoles                                                  operator.openshift.io/v1                      false        Console

```



## Controller是什么

也就是「控制器」，控制Kubernetes的资源实体。怎么控制呢？通过监听资源变化事件。这个事件可能是用户发起的（他希望把资源从A状态更新到B状态），Controller就会获取这个事件并处理事件，即更新目标资源。Kubernetes默认有很多控制器，他们控制着Kubernetes默认资源，如Pod、Deployment、Service等，他们都包含在Controller Manager中。但如果你的资源是个CRD，因为没有对应的控制器，你就得为它自己写Controller了



```
[root@qxu mysql]# kubectl api-resources|grep -i controller
replicationcontrollers                rc                  v1                                            true         ReplicationController
controllerrevisions                                       apps/v1                                       true         ControllerRevision
controllerconfigs                                         machineconfiguration.openshift.io/v1          false        ControllerConfig
csisnapshotcontrollers                                    operator.openshift.io/v1                      false        CSISnapshotController
ingresscontrollers                                        operator.openshift.io/v1                      true         IngressController
kubecontrollermanagers                                    operator.openshift.io/v1                      false        KubeControllerManager
openshiftcontrollermanagers                               operator.openshift.io/v1                      false        OpenShiftControllerManager

```



#### Ｏperator就是使用ＣＲＤ实现的定制化的controller,它与k8s内建的controller遵循同样的运行模式



### Operator与Helm

http://dockone.io/article/1464915



示例mysql operator

https://blog.csdn.net/qq_39458487/article/details/125260925

https://github.com/mysql/mysql-operator

#### 安装operator

```

kubectl apply -f https://raw.githubusercontent.com/mysql/mysql-operator/trunk/deploy/deploy-crds.yaml
kubectl apply -f https://raw.githubusercontent.com/mysql/mysql-operator/trunk/deploy/deploy-operator.yaml --namespace mysql-operator


kubectl get deployment -n mysql-operator mysql-operator


```

 如果是ocp4 ,执行一下步

```
oc adm policy add-scc-to-user privileged -z mysql-operator-sa　 -n mysql-operator 

```

#### 检查新增的自定义资源

```
kubectl api-resources|grep innodbclusters
innodbclusters                        ic,ics              mysql.oracle.com/v2                           true         InnoDBCluste
```



#### 使用operator部署一个mysql 实例



```
kubectl create ns test1

kubectl create secret generic mypwds \
        --from-literal=rootUser=root \
        --from-literal=rootHost=% \
        --from-literal=rootPassword="redhat" \
        -n test1

cat  mycluster.yaml

apiVersion: mysql.oracle.com/v2
kind: InnoDBCluster
metadata:
  name: mycluster
spec:
  secretName: mypwds
  tlsUseSelfSigned: true
  instances: 3
  router:
    instances: 1
```



如果是ocp4 ,增加权限

```
oc adm policy add-scc-to-user privileged -z mysql-operator-sa　
```

检查有4个对应的pod产生

```
[root@qxu mysql]# kubectl get pod  -n test1
NAME                                READY   STATUS    RESTARTS   AGE
mycluster-0                         2/2     Running   0          2m3s
mycluster-1                         2/2     Running   0          2m3s
mycluster-2                         2/2     Running   0          2m3s
mycluster-router-79565f87f6-twgkf   1/1     Running   0          37s

[root@qxu mysql]# kubectl get pvc -n test1
NAME                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
datadir-mycluster-0   Bound    pvc-4ba00270-0f6d-4f32-b1d2-b0eae6c96506   2Gi        RWO            gp2            8m54s
datadir-mycluster-1   Bound    pvc-e5c53ced-c4ff-4e96-ad8d-743e7e0985af   2Gi        RWO            gp2            6m51s
datadir-mycluster-2   Bound    pvc-1461736f-72d2-4f06-8c52-373f7e463282   2Gi        RWO            gp2            6m51s
```



#### 访问　mysql实例Using Port Forwarding

mycluster-router-79565f87f6-twgkf 要等这个pod　running

```
kubectl port-forward service/mycluster mysql
mysql -h127.0.0.1 -P3306 -uroot -predhat

show databases;


[qxu@qxu Desktop]$ mysql -uroot -predhat -P3306 -h 127.0.0.1
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 845
Server version: 8.0.31 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+-------------------------------+
| Database                      |
+-------------------------------+
| information_schema            |
| mysql                         |
| mysql_innodb_cluster_metadata |
| performance_schema            |
| sys                           |
+-------------------------------+
5 rows in set (0.23 sec)

mysql> exit
Bye

```

