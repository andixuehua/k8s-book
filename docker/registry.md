## 如何建立个基本的本地registry

建立目录

```
mkdir -p /opt/registry/{data,auth,certs}
```

密码文件

```
htpasswd -bBc /data/registry/auth/htpasswd admin redhat

```

生成证书, and cp domain.crt domain.key /opt/registry/certs/

```
# cat ca.sh 
host_fqdn="registry.crc.test"
cert_c="CN"   # Country Name (C, 2 letter code)
cert_s="Beijing"          # Certificate State (S)
cert_l="Beijing"       # Certificate Locality (L)
cert_o="Lab"   # Certificate Organization (O)
cert_ou="DevOps"      # Certificate Organizational Unit (OU)
cert_cn="${host_fqdn}"    # Certificate Common Name (CN)

openssl req \
    -newkey rsa:4096 \
    -nodes \
    -sha256 \
    -keyout domain.key \
    -x509 \
    -days 3650 \
    -out domain.crt \
    -extensions "SAN=DNS:$host_fqdn"  \
    -subj "/C=${cert_c}/ST=${cert_s}/L=${cert_l}/O=${cert_o}/OU=${cert_ou}/CN=${cert_cn}"
```

启动registry

```
 cat start.sh 
docker run --name registry -p 5000:5000   \
  -v /opt/registry/data:/var/lib/registry:z   \
  -v  /opt/registry/auth:/auth:z    \
  -e "REGISTRY_AUTH=htpasswd"  \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v /opt/registry/certs:/certs:z \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt    \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -d docker.io/library/registry:2

```



docker login -u admin -p redhat registry.crc.test:5000

如果不能访问,信任自签证书的配置,请参考 http://localhost:4000/docker/config.html

