日志日志框架:

https://mp.weixin.qq.com/s?__biz=MzI3MTQyNDc5MA==&mid=2247488395&idx=1&sn=9135410b1aa2701a536d18c807d9e5dd&chksm=eac35ff2ddb4d6e435b5df9612bbc7278abc37583c3dd2aa0359b66f63bf2a012eb9b29fcf72&mpshare=1&scene=24&srcid=1205CYNtCajYOImduIQZEbzX&sharer_sharetime=1670194833917&sharer_shareid=1b071e140471a5427f1c43ba67735110&ascene=14&devicetype=android-29&version=28001759&nettype=cmnet&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=AF&exportkey=n_ChQIAhIQYAnQ7yY4Zj2sibSEnhZK%2FRLbAQIE97dBBAEAAAAAAMhLCTZXMscAAAAOpnltbLcz9gKNyK89dVj06WbLqxrk8uhuP3G8ZoJpxj7%2Bbae9DUqhAAem6amj%2BLzKtxsJZ9j%2B%2FQOH5liemtn51uukf0kF1wHwKLZ1zFu8abGZXveoiyY74YiM%2F6DdOOA69GGCV%2FrGfNgypqdRvN8hu%2BegtASnuKQyVCsyZpLLWWXxBkr2C1d3ylzRYJrBRJA4qvaBmanT9BF7yFddpqDrNrDp5HnxA5xOAvAtSR4YWMieGlMBUd2%2BxX0eSJYvS%2FbPAPCrKA%3D%3D&pass_ticket=KEon9IncUp1EvKJKhz92VjENn5SZ3EtDBTg6pIxBtctJMENYp5EBdE6QjSOh%2FZjVvAjT1s0bUIzNjnhTDPXdPQ%3D%3D&wx_header=3





日志技术选择:



EFK , Elasticsearch`、`Fluentd` 和 `Kibana

https://github.com/scriptcamp/kubernetes-efk



EFK, Elasticsearch`、filebeat和 `Kibana

https://blog.csdn.net/qq_35270805/article/details/124862134



ELK, [Log-Pilot + ES + Kibana 日志方案,

 https://github.com/easzlab/kubeasz/blob/master/docs/guide/log-pilot.md



### install EFK

参考： https://devopscube.com/setup-efk-stack-on-kubernetes/
下载：https://github.com/scriptcamp/kubernetes-efk

extend 3 node to 4c/16G , 

##### v.124一定要在default　ns下安装,不然会有很多权限问题



```
kubectl create ns logging

1.修改所有的yaml中的配置，在logging下部署，主要是fluted中的两个yaml  (缺省在default下部署)
2. 修改fluted-ds.yaml,加入以下tolerations配置，这样才能在master节点上部署（1.16以后，缺省不在master上部署daemonset的pod）
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      -key: node-role.kubernetes.io/control-plane
        effect: NoSchedule

3. 依次部署es, kibana, fluentd的配置

 kubectl apply -f elasticsearch/. -n logging
 kubectl apply -f fluentd/. -n logging
 kubectl apply -f kibana/. -n logging
        
        
4. kibana 的expose方式缺省是node port 30000,，测试可以使用。也可以增加一个ingress

5. kibana界面上新增一个logstash的index 就可以看到日志了。
```



kibana ingress

```

```

fluentd　pod如果不能启动,有如下错误:

```
"system:serviceaccount:logging:fluentd\\\" cannot list resource \\\"namespaces
```

 

修改下面的文件,加入,因为我们部署在非logging　ns中,所以要添加对应的权限

- kind: ServiceAccount
  name: fluentd
  namespace: logging
  
  

```
vi fluentd/fluentd-rb.yaml 

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: default
- kind: ServiceAccount
  name: fluentd
  namespace: logging
```



```
# kubectl get pod -n logging
NAME                     READY   STATUS    RESTARTS   AGE
es-cluster-0             1/1     Running   0          4m20s
es-cluster-1             1/1     Running   0          2m55s
es-cluster-2             1/1     Running   0          2m38s
fluentd-8gfm9            1/1     Running   0          111s
fluentd-8jc7x            1/1     Running   0          111s
fluentd-wc69r            1/1     Running   0          111s
kibana-cd67d89cd-rbzlr   1/1     Running   0          4m12s
```



临时修改为不可用

kubectl edit daemonset fluentd -n logging 

```
      dnsPolicy: ClusterFirst
      nodeSelector:
        non-existing: "true"


kubectl scale statefulset es-cluster --replicas=0 -n logging
kubectl scale deployment kibana --replicas=0 -n logging
```

创建一个log



![](/home/qxu/Documents/k8sbook-new/log/k8s-kibana.png)







```
kubectl get pod -n elastic-system
NAME                                           READY   STATUS      RESTARTS      AGE
curator-elasticsearch-curator-27937500-h54c9   0/1     Completed   0             2d5h
curator-elasticsearch-curator-27938940-d57cx   0/1     Completed   0             28h
curator-elasticsearch-curator-27940380-2ljpn   0/1     Completed   0             5h5m
elasticsearch-master-0                         1/1     Running     2 (28h ago)   101d
elasticsearch-master-1                         1/1     Running     2 (28h ago)   101d
elasticsearch-master-2                         1/1     Running     2 (28h ago)   101d
filebeat-filebeat-d4ppr                        1/1     Running     2 (28h ago)   101d
filebeat-filebeat-ddccl                        1/1     Running     3 (27h ago)   101d
filebeat-filebeat-lccgv                        1/1     Running     5 (27h ago)   101d
kibana-kibana-75754cfdc8-cqn24                 0/1     Running     2 (28h ago)   101d
```



```
#增加toleration,让filebeat部到master上
kubectl edit daemonsets.apps filebeat-filebeat -n elastic-system

```

TK: kibana index:

https://kibana.elastic-system.apps.taikang1.local/app/discover#/?_g=(filters:!(),refreshInterval:(pause:!t,value:0),time:(from:now-15m,to:now))&_a=(columns:!(_source),filters:!(),index:'5c9a8ca0-acf9-11ed-b8e9-cbf5175a5b61',interval:auto,query:(language:kuery,query:''),sort:!())



index manange:

https://kibana.elastic-system.apps.taikang1.local/app/management



kibana问题处理

https://www.codenong.com/cs105117875/

```
kubectl exec -it elastic-0 -n elastic-system
curl -XDELETE http://localhost:9200/.kibana*
```

