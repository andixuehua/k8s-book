### Token过期

通过kubeadm初始化后，都会提供[node](https://so.csdn.net/so/search?q=node&spm=1001.2101.3001.7020)加入的token。
默认token的有效期为24小时，当过期之后，该token就不可用了。

```
[root@worker02 ~]# kubeadm join 192.168.146.55:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:c61929e039f8ed9b96aeccc78083681aaf7edda1b17babd6d9450a976d2688e1 --cri-socket unix:///var
/run/cri-dockerd.sock
[preflight] Running pre-flight checks
error execution phase preflight: couldn't validate the identity of the API Server: could not find a JWS signature in the cluster-info ConfigMap for token ID "abcdef"
To see the stack trace of this error execute with --v=5 or higher

```



解决方法如下：在master01

kubeadm token list #查看token

 kubeadm token create #创建新的token

```
[root@master01 ~]#  kubeadm token create
8e5lmw.38nw0b7inl4h256i
[root@master01 ~]# kubeadm token list
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
8e5lmw.38nw0b7inl4h256i   23h         2022-10-10T01:11:38Z   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token

```

获取CA公钥的哈希值

```
获取CA公钥的哈希值
[root@master01 ~]# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed  's/^ .* //'
(stdin)= c61929e039f8ed9b96aeccc78083681aaf7edda1b17babd6d9450a976d2688e1

 
 
kubeadm join 192.168.40.8:6443 --token token填这里   --discovery-token-ca-cert-hash sha256:哈希值填这里
```



加入node:在新node上

```

[root@worker02 ~]# kubeadm join 192.168.146.55:6443 --token 8e5lmw.38nw0b7inl4h256i --discovery-token-ca-cert-hash sha256:c61929e039f8ed9b96aeccc78083681aaf7edda1b17babd6d9450a976d2688e1 --cri-socket unix:///var
/run/cri-dockerd.sock
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster. 

```






### 检查证书



```
 kubeadm certs check-expiration
 [check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Oct 07, 2023 10:57 UTC   363d            ca                      no
apiserver                  Oct 07, 2023 10:57 UTC   363d            ca                      no
apiserver-etcd-client      Oct 07, 2023 10:57 UTC   363d            etcd-ca                 no      
apiserver-kubelet-client   Oct 07, 2023 10:57 UTC   363d            ca                      no
controller-manager.conf    Oct 07, 2023 10:57 UTC   363d            ca                      no
etcd-healthcheck-client    Oct 07, 2023 10:57 UTC   363d            etcd-ca                 no
etcd-peer                  Oct 07, 2023 10:57 UTC   363d            etcd-ca                 no
etcd-server                Oct 07, 2023 10:57 UTC   363d            etcd-ca                 no
front-proxy-client         Oct 07, 2023 10:57 UTC   363d            front-proxy-ca          no
scheduler.conf             Oct 07, 2023 10:57 UTC   363d            ca                      no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Oct 04, 2032 10:57 UTC   9y              no
etcd-ca                 Oct 04, 2032 10:57 UTC   9y              no
front-proxy-ca          Oct 04, 2032 10:57 UTC   9y              no

```

