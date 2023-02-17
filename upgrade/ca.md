证书到期更新:



https://blog.csdn.net/lduan_001/article/details/124441276



https://blog.csdn.net/jiangbenchu/article/details/109287837



检查过期时间　

```
kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Feb 06, 2023 14:12 UTC   10d                                     no
apiserver                  Feb 06, 2023 14:12 UTC   10d             ca                      no
apiserver-etcd-client      Feb 06, 2023 14:12 UTC   10d             etcd-ca                 no
apiserver-kubelet-client   Feb 06, 2023 14:12 UTC   10d             ca                      no
controller-manager.conf    Feb 06, 2023 14:12 UTC   10d                                     no
etcd-healthcheck-client    Feb 06, 2023 14:12 UTC   10d             etcd-ca                 no
etcd-peer                  Feb 06, 2023 14:12 UTC   10d             etcd-ca                 no
etcd-server                Feb 06, 2023 14:12 UTC   10d             etcd-ca                 no
front-proxy-client         Feb 06, 2023 14:12 UTC   10d             front-proxy-ca          no
scheduler.conf             Feb 06, 2023 14:12 UTC   10d                                     no

```



更新证书:

```
# kubeadm alpha certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

```



再检查:

```
# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jan 27, 2024 03:14 UTC   364d                                    no
apiserver                  Jan 27, 2024 03:14 UTC   364d            ca                      no
apiserver-etcd-client      Jan 27, 2024 03:14 UTC   364d            etcd-ca                 no
apiserver-kubelet-client   Jan 27, 2024 03:14 UTC   364d            ca                      no
controller-manager.conf    Jan 27, 2024 03:14 UTC   364d                                    no
etcd-healthcheck-client    Jan 27, 2024 03:14 UTC   364d            etcd-ca                 no
etcd-peer                  Jan 27, 2024 03:14 UTC   364d            etcd-ca                 no
etcd-server                Jan 27, 2024 03:14 UTC   364d            etcd-ca                 no
front-proxy-client         Jan 27, 2024 03:14 UTC   364d            front-proxy-ca          no
scheduler.conf             Jan 27, 2024 03:14 UTC   364d                                    no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Feb 04, 2032 14:12 UTC   9y              no
etcd-ca                 Feb 04, 2032 14:12 UTC   9y              no
front-proxy-ca          Feb 04, 2032 14:12 UTC   9y              no
[root@kubernetes-master ~] ens33 = 192.168.146.91

```

查看文件

```
# pwd
/etc/kubernetes/pki
[root@kubernetes-master /etc/kubernetes/pki] ens33 = 192.168.146.91
# ll
total 60
-rw-r--r-- 1 root root 1277 Jan 27 11:14 apiserver.crt
-rw-r--r-- 1 root root 1135 Jan 27 11:14 apiserver-etcd-client.crt
-rw------- 1 root root 1675 Jan 27 11:14 apiserver-etcd-client.key
-rw------- 1 root root 1679 Jan 27 11:14 apiserver.key
-rw-r--r-- 1 root root 1143 Jan 27 11:14 apiserver-kubelet-client.crt
-rw------- 1 root root 1675 Jan 27 11:14 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1066 Feb  6  2022 ca.crt
-rw------- 1 root root 1679 Feb  6  2022 ca.key
-rw-r--r-- 1 root root   17 Sep 27 08:24 ca.srl
drwxr-xr-x 2 root root  162 Feb  6  2022 etcd
-rw-r--r-- 1 root root 1078 Feb  6  2022 front-proxy-ca.crt
-rw------- 1 root root 1675 Feb  6  2022 front-proxy-ca.key
-rw-r--r-- 1 root root 1103 Jan 27 11:14 front-proxy-client.crt
-rw------- 1 root root 1679 Jan 27 11:14 front-proxy-client.key
-rw------- 1 root root 1679 Feb  6  2022 sa.key
-rw------- 1 root root  451 Feb  6  2022 sa.pub
[root@kubernetes-master /etc/kubernetes/pki] ens33 = 192.168.146.91

```



以上的证书得更新未包括 kubelet



# **kubelet证书自动续签**

##### 

参考https://www.cnblogs.com/LiuChang-blog/p/15347791.html



比如查看kubeadm方式部署的kubelet证书过期时间:
\# cd /var/lib/kubelet/pki/
\# openssl x509 -in kubelet-client-current.pem -noout -dates

