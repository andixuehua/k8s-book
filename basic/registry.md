自签https证书的registry无法login，出现以下错误 

```
docker login -u admin -p redhat registry.crc.test:5000
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Error response from daemon: Get "https://registry.crc.test:5000/v2/": x509: certificate signed by unknown autho
rity

cp domain.crt到 /etc/pki/ca-trust/source/anchors/
update-ca-trust 未能解决此问题。



```

- 解决方式1 



```
vi /etc/docker/daemon.json
{ "insecure-registries" : ["registry.crc.test:5000"] }

systemctl restart docker

docker login -u admin -p redhat registry.crc.test:5000
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
```

或在docker的systemctl服务中加入insecure-registry的配置



```
systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/etc/systemd/system/docker.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/docker.service.d
           └─docker-options.conf


cd /etc/systemd/system/docker.service.d
 more docker-options.conf 
[Service]
Environment="DOCKER_OPTS= --iptables=false \
--exec-opt native.cgroupdriver=systemd \
--insecure-registry=hub.taikang1.local  \
 \
--data-root=/var/lib/docker \
--log-opt max-size=50m --log-opt max-file=5"


```



- 解决方式2 ，***推荐此方式***

参考： https://www.ibm.com/docs/en/cloud-paks/cp-management/2.1.x?topic=monitoring-pods-displaying-imagepullbackoff-status

```
mkdir -p /etc/docker/certs.d/registry.crc.test:5000
cp domain.crt /etc/docker/certs.d/registry.crc.test:5000/ca.crt

不用重启docker,就可以直接访问 
docker login -u admin -p redhat registry.crc.test:5000

```





## 基于现有Docker凭据创建secret

kubernetes集群使用docker注册表类型的秘密对容器注册表进行身份验证，以获取私有映像。

如果您已经运行了Docker登录，则可以将该凭证复制到Kubernetes中的当前namespace中：测试可用

```
kubectl create secret generic harborsecret \
    --from-file=.dockerconfigjson=/root/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson \
    -n test3
    
kubectl create secret generic dockercred \
    --from-file=.dockerconfigjson=/root/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson \
    -n test3    
```



```
# cat /root/.docker/config.json 
{
        "auths": {
                "registry.crc.test:5000": {
                        "auth": "YWRtaW46cmVkaGF0"
                }
        }
}
```



或者(测试可用)

```
kubectl create secret docker-registry regcred --docker-server=registry.crc.test:5000 --docker-username=admin --docker-password=redhat -n test
```



# 在demployment yaml文件中的使用示例

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
... 
spec:
      imagePullSecrets:
      - name: harborsecret
      containers:
      - name: eureka
        image: 192.168.10.122/library/alpine:latest
...
```



参考,[k8s的imagePullSecrets如何生成及使用](https://www.cnblogs.com/liujunjun/p/14383285.html)

　https://www.cnblogs.com/liujunjun/p/14383285.html

