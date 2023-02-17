证书存储在哪里？
如果是用kubeadm部署的，那么就存储在 /etc/kubernetes/pki. 如果是别的部署的，应该就在/etc/kubernetes 下

https://www.jianshu.com/p/01aff70f602d



查看证书,　in master node

```
[root@master1 ~]# cd /etc/kubernetes/pki
[root@master1 pki]# pwd
/etc/kubernetes/pki
[root@master1 pki]# ll
total 56
-rw-r--r--. 1 root root 1489 Nov  7 10:48 apiserver.crt
-rw-r--r--. 1 root root 1155 Nov  7 10:48 apiserver-etcd-client.crt
-rw-------. 1 root root 1679 Nov  7 10:48 apiserver-etcd-client.key
-rw-------. 1 root root 1679 Nov  7 10:48 apiserver.key
-rw-r--r--. 1 root root 1164 Nov  7 10:48 apiserver-kubelet-client.crt
-rw-------. 1 root root 1679 Nov  7 10:48 apiserver-kubelet-client.key
-rw-r--r--. 1 root root 1099 Nov  4 18:23 ca.crt
-rw-------. 1 root root 1679 Nov  4 18:23 ca.key
drwxr-xr-x. 2 root root  162 Nov  4 18:23 etcd
-rw-r--r--. 1 root root 1115 Nov  4 18:23 front-proxy-ca.crt
-rw-------. 1 root root 1675 Nov  4 18:23 front-proxy-ca.key
-rw-r--r--. 1 root root 1119 Nov  7 10:48 front-proxy-client.crt
-rw-------. 1 root root 1675 Nov  7 10:48 front-proxy-client.key
-rw-------. 1 root root 1675 Nov  4 18:23 sa.key
-rw-------. 1 root root  451 Nov  4 18:23 sa.pub

```





三类ＣＡ

| path                   | Default CN                | description                                                  |
| ---------------------- | ------------------------- | ------------------------------------------------------------ |
| ca.crt,key             | kubernetes                | Kubernetes general CA                                        |
| etcd/ca.crt,key        | etcd-ca                   | For all etcd-related functions                               |
| front-proxy-ca.crt,key | kubernetes-front-proxy-ca | For the [front-end proxy](https://links.jianshu.com/go?to=https%3A%2F%2Fkubernetes.io%2Fdocs%2Ftasks%2Fextend-kubernetes%2Fconfigure-aggregation-layer%2F) |



只有以上三个CA是CA文件,其他的crt文件都是由这些CA签发的证书

```
[root@master1 pki]# cat front-proxy-ca.crt |openssl x509 -text|grep -i true
                CA:TRUE
[root@master1 pki]# cat etcd/ca.crt |openssl x509 -text|grep -i true
                CA:TRUE
[root@master1 pki]# cat ca.crt |openssl x509 -text|grep -i true
                CA:TRUE
[root@master1 pki]# 

```

除了上面的证书，对于每一个CA，还要搞一个公钥|私钥 秘钥对用于 服务账户（service account）管理，比如 `sa.key` and `sa.pub`。



所有证书一览

| Default CN                    | Parent CA                 | O (in Subject) | kind           | hosts (SAN)                                         |
| ----------------------------- | ------------------------- | -------------- | -------------- | --------------------------------------------------- |
| kube-etcd                     | etcd-ca                   |                | server, client | `<hostname>`, `<Host_IP>`, `localhost`, `127.0.0.1` |
| kube-etcd-peer                | etcd-ca                   |                | server, client | `<hostname>`, `<Host_IP>`, `localhost`, `127.0.0.1` |
| kube-etcd-healthcheck-client  | etcd-ca                   |                | client         |                                                     |
| kube-apiserver-etcd-client    | etcd-ca                   | system:masters | client         |                                                     |
| kube-apiserver                | kubernetes                |                | server         | `<hostname>`, `<Host_IP>`, `<advertise_IP>`, `[1]`  |
| kube-apiserver-kubelet-client | kubernetes                | system:masters | client         |                                                     |
| front-proxy-client            | kubernetes-front-proxy-ca |                | client         |                                                     |



有效期检查

因为默认证书有效期为 1 年，CA 根证书是 10 年：

```
[root@master1 pki]# for line in `ls *.crt`; do echo $line && cat $line |openssl x509 -text |grep -A2 Validity;done
apiserver.crt
        Validity
            Not Before: Nov  4 10:23:12 2022 GMT
            Not After : Nov  7 02:48:35 2023 GMT
apiserver-etcd-client.crt
        Validity
            Not Before: Nov  4 10:23:14 2022 GMT
            Not After : Nov  7 02:48:35 2023 GMT
apiserver-kubelet-client.crt
        Validity
            Not Before: Nov  4 10:23:12 2022 GMT
            Not After : Nov  7 02:48:36 2023 GMT
ca.crt
        Validity
            Not Before: Nov  4 10:23:12 2022 GMT
            Not After : Nov  1 10:23:12 2032 GMT
front-proxy-ca.crt
        Validity
            Not Before: Nov  4 10:23:13 2022 GMT
            Not After : Nov  1 10:23:13 2032 GMT
front-proxy-client.crt
        Validity
            Not Before: Nov  4 10:23:13 2022 GMT
            Not After : Nov  7 02:48:38 2023 GMT


[root@master1 pki]# for line in `ls etcd/*.crt`; do echo $line && cat $line |openssl x509 -text |grep -A2 Validity;done
etcd/ca.crt
        Validity
            Not Before: Nov  4 10:23:14 2022 GMT
            Not After : Nov  1 10:23:14 2032 GMT
etcd/healthcheck-client.crt
        Validity
            Not Before: Nov  4 10:23:14 2022 GMT
            Not After : Nov  7 02:48:36 2023 GMT
etcd/peer.crt
        Validity
            Not Before: Nov  4 10:23:14 2022 GMT
            Not After : Nov  7 02:48:37 2023 GMT
etcd/server.crt
        Validity
            Not Before: Nov  4 10:23:14 2022 GMT
            Not After : Nov  7 02:48:38 2023 GMT


[root@master1 pki]# kubeadm certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W1127 22:06:20.282543 3494777 utils.go:69] The recommended value for "clusterDNS" in "KubeletConfiguration" is: [10.233.0.10]; the provided value is: [169.254.25.10]

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Nov 07, 2023 02:48 UTC   344d            ca                      no      
apiserver                  Nov 07, 2023 02:48 UTC   344d            ca                      no      
apiserver-etcd-client      Nov 07, 2023 02:48 UTC   344d            etcd-ca                 no      
apiserver-kubelet-client   Nov 07, 2023 02:48 UTC   344d            ca                      no      
controller-manager.conf    Nov 07, 2023 02:48 UTC   344d            ca                      no      
etcd-healthcheck-client    Nov 07, 2023 02:48 UTC   344d            etcd-ca                 no      
etcd-peer                  Nov 07, 2023 02:48 UTC   344d            etcd-ca                 no      
etcd-server                Nov 07, 2023 02:48 UTC   344d            etcd-ca                 no      
front-proxy-client         Nov 07, 2023 02:48 UTC   344d            front-proxy-ca          no      
scheduler.conf             Nov 07, 2023 02:48 UTC   344d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Nov 01, 2032 10:23 UTC   9y              no      
etcd-ca                 Nov 01, 2032 10:23 UTC   9y              no      
front-proxy-ca          Nov 01, 2032 10:23 UTC   9y              no      


```





证书的更新:, 参考:　https://blog.csdn.net/wo18237095579/article/details/119956018

https://blog.csdn.net/weixin_37288522/article/details/125059065

 kubeadm certs renew all



