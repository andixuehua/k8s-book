metric server :

https://github.com/kubernetes-sigs/metrics-server



１.24->0.6.0

没安装之前 top 命令不显示

```
[root@master01 ~]# kubectl top pod -n monitoring
error: Metrics not available for pod monitoring/alertmanager-main-0, age: 2h47m29.238143203s

```



```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.0/components.yaml

or
sed -i '/image:/s|k8s.gcr.io/metrics-server|willdockerhub|' metric-server-0.6.0.yaml
```

如果 metric pod不能正常ready, metrics-server pod 提示 no metrics known for pod 或者 x509: cannot validate certificate 

modify metric-server-0.6.0.yaml,在启动参数中加入以下参数

```
args:
  - --kubelet-insecure-tls
```

以上参考:　

https://ssoor.github.io/2020/03/25/k8s-metrics-server-error-1/

https://github.com/kubernetes-sigs/metrics-server/issues/1025



测试:　

```
[root@master01 install]# kubectl top node
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
master01   472m         11%    1816Mi          11%
master02   361m         9%     1399Mi          8%
master03   323m         8%     1217Mi          15%
worker01   454m         11%    2617Mi          34%

```

当有节点不是ready状态时, top node也会不能正常返回数据,可以查看metric-server pod的log进行查看
