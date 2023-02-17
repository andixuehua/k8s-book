https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/

# Ingress



**特性状态：** `Kubernetes v1.19 [stable]`

Ingress 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP。

Ingress 可以提供负载均衡、SSL 终结和基于名称的虚拟托管。



Ingress 可为 Service 提供外部可访问的 URL、负载均衡流量、终止 SSL/TLS，以及基于名称的虚拟托管。 [Ingress 控制器](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers) 通常负责通过负载均衡器来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

Ingress 不会公开任意端口或协议。 将 HTTP 和 HTTPS 以外的服务公开到 Internet 时，通常使用 [Service.Type=NodePort](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport) 或 [Service.Type=LoadBalancer](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#loadbalancer) 类型的 Service。



```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log-collection
spec:
  selector:
    matchLabels:
      app: log-collection
  replicas: 1
  template:
    metadata:
      labels:
        app: log-collection
    spec:
      containers:
        - name: log-collection
          image: quay.io/qxu/log-collection
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: log-collection
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: log-collection
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection
spec:
  rules:
  - host: "log-collection.user1.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection
            port:
              number: 8080
```





单独domain name,配置dns解析



<1.24

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: log-collection1
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: log-collection.crc.test
    http:
      paths:
      - path:
        backend:
          serviceName: log-collection
          servicePort: 8080
          
          
```



wilde name: 配置dns解析和haproxy的配置

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection
spec:
  rules:
  - host: "log-collection.user1.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection
            port:
              number: 8080
```



dnsmasq 的配置：

```
# cat /etc/dnsmasq.d/k8s.conf 
port=53
domain-needed
bogus-priv
no-negcache
cache-size=10000
bind-interfaces
expand-hosts
listen-address=::1,127.0.0.1,192.168.146.90
dns-forward-max=15000

address=/registry.crc.test/192.168.146.90
address=/.apps.test.crc.test/192.168.146.90
address=/whoami.crc.test/192.168.146.90
address=/log-collection.crc.test/192.168.146.90

#对于log-collection.crc.test 这个不是泛域名的，也可以直接将dns解板到pod所在的主机相如下，也可以，但只有一个pod了
＃address=/log-collection.crc.test/192.168.146.91

```



haproxy的配置:

```
# cat /etc/haproxy/haproxy.cfg |grep -v "#"
global
   log /dev/log local0
   log /dev/log local1 notice
   chroot      /var/lib/haproxy
   pidfile     /var/run/haproxy.pid
   maxconn     4000
   stats timeout 30s
   user        haproxy
   group       haproxy
   daemon
   stats socket /var/lib/haproxy/stats

defaults
    mode                    tcp
    log                     global
    option                  httplog
    option                  dontlognull
    option                  http-server-close
    option                  redispatch
    retries                 3
    maxconn                 50000

frontend http_stats
   bind *:58080
   mode http
   stats uri /haproxy?stats



frontend http
    bind *:80
    default_backend http
    mode tcp
    option tcplog


backend http
    balance source
    mode tcp
    server http0 192.168.146.92:80 check
    server http1 192.168.146.93:80 check

frontend https
    bind *:443
    default_backend https
    mode tcp
    option tcplog


backend https
    balance source
    mode tcp
    server http0 192.168.146.92:443 check
    server http1 192.168.146.93:443 check

```





ingress-nginx本身有https的证书,

新建立的ingress泛域名可以使用http和https访问

```
[root@master01 test]# curl https://whoami.apps.test.icbc.io -sSk
<div style="text-align:center;"><h1>Who Am I - whoami-6458676f5f-fd9jj</h1></div><div style="margin-left:10%"><br/><br/>Available processors (cores): 1<br/><br/>Free memory (bytes): 226855328<br/>Maximum memory 
(bytes): 4005888000<br/>Total memory available to JVM (bytes): 251527168<br/><br/>File system root: /<br/>Total space (bytes): 75125227520<br/>Free space (bytes): 67617087488<br/>Usable space (bytes): 6761708748
8<br/><br/>Network Details:<br/>Display name: eth0              InetAddress: /10.244.5.15<br/>Display name: lo          InetAddress: /127.0.0.1</div>[root@master01 test]# 

```



ingress pod中nginx upstream到pod的配置,是由lua动态获取的,不会写到nginx.conf文件中,这个与ocp的router有所不同.



```
kubectl exec -it ingress-nginx-controller-6kddq -n ingress-nginx /bin/sh

grep -A135 "start server log-collection.user1" nginx.conf
	## start server log-collection.user1.apps.taikang1.local
	server {
		server_name log-collection.user1.apps.taikang1.local ;
		
		listen 80  ;
		listen [::]:80  ;
		listen 443  ssl http2 ;
		listen [::]:443  ssl http2 ;
		
		set $proxy_upstream_name "-";
		
		ssl_certificate_by_lua_block {
			certificate.call()
		}
		
		location / {
			
			set $namespace      "user1";
			set $ingress_name   "log-collection";
			set $service_name   "log-collection";
			set $service_port   "8080";
			set $location_path  "/";
			set $global_rate_limit_exceeding n;
			
			rewrite_by_lua_block {
				lua_ingress.rewrite({
					force_ssl_redirect = false,
					ssl_redirect = true,
					force_no_ssl_redirect = false,
					preserve_trailing_slash = false,
					use_port_in_redirects = false,
					global_throttle = { namespace = "", limit = 0, window_size = 0, key = { }, ignored_cidrs = { } },
				})
				balancer.rewrite()
				plugins.run()
			}
			
			# be careful with `access_by_lua_block` and `satisfy any` directives as satisfy any
			# will always succeed when there's `access_by_lua_block` that does not have any lua code doing `ngx.exit(ngx.DECLINED)`
			# other authentication method such as basic auth or external auth useless - all requests will be allowed.
			#access_by_lua_block {
			#}
			
			header_filter_by_lua_block {
				lua_ingress.header()
				plugins.run()
			}
			
			body_filter_by_lua_block {
				plugins.run()
			}
			
			log_by_lua_block {
				balancer.log()
				
				monitor.call()
				
				plugins.run()
			}
			
			port_in_redirect off;
			
			set $balancer_ewma_score -1;
			set $proxy_upstream_name "user1-log-collection-8080";
			set $proxy_host          $proxy_upstream_name;
			set $pass_access_scheme  $scheme;
			
			set $pass_server_port    $server_port;
			
			set $best_http_host      $http_host;
			set $pass_port           $pass_server_port;
			
			set $proxy_alternative_upstream_name "";
			
			client_max_body_size                    1m;
			
			proxy_set_header Host                   $best_http_host;
			
			# Pass the extracted client certificate to the backend
			
			# Allow websocket connections
			proxy_set_header                        Upgrade           $http_upgrade;
			
			proxy_set_header                        Connection        $connection_upgrade;
			
			proxy_set_header X-Request-ID           $req_id;
			proxy_set_header X-Real-IP              $remote_addr;
			
			proxy_set_header X-Forwarded-For        $remote_addr;
			
			proxy_set_header X-Forwarded-Host       $best_http_host;
			proxy_set_header X-Forwarded-Port       $pass_port;
			proxy_set_header X-Forwarded-Proto      $pass_access_scheme;
			proxy_set_header X-Forwarded-Scheme     $pass_access_scheme;
			
			proxy_set_header X-Scheme               $pass_access_scheme;
			
			# Pass the original X-Forwarded-For
			proxy_set_header X-Original-Forwarded-For $http_x_forwarded_for;
			
			# mitigate HTTPoxy Vulnerability
			# https://www.nginx.com/blog/mitigating-the-httpoxy-vulnerability-with-nginx/
			proxy_set_header Proxy                  "";
			
			# Custom headers to proxied server
			
			proxy_connect_timeout                   5s;
			proxy_send_timeout                      60s;
			proxy_read_timeout                      60s;
			
			proxy_buffering                         off;
			proxy_buffer_size                       4k;
			proxy_buffers                           4 4k;
			
			proxy_max_temp_file_size                1024m;
			
			proxy_request_buffering                 on;
			proxy_http_version                      1.1;
			
			proxy_cookie_domain                     off;
			proxy_cookie_path                       off;
			
			# In case of errors try the next upstream server before returning an error
			proxy_next_upstream                     error timeout;
			proxy_next_upstream_timeout             0;
			proxy_next_upstream_tries               3;
			
			proxy_pass http://upstream_balancer;
			
			proxy_redirect                          off;
			
		}
		
	}
	## end server log-collection.user1.apps.taikang1.local


```



```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ${KEY_FILE} -out ${CERT_FILE} -subj "/CN=${HOST}/O=${HOST}" -addext "subjectAltName = DNS:${HOST}"

#生成crt
export HOST=www.log.local
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ${HOST}.key -out ${HOST}.crt -subj "/CN=${HOST}/O=${HOST}" -addext "subjectAltName = DNS:${HOST}"

#查看crt
cat www.log.local.crt |openssl x509 -text

#生成secret
kubectl create secret tls ingress-secret  --key ${HOST}.key --cert ${HOST}.crt -n user1
```

ingress



```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - www.log.local
    secretName: ingress-secret
  rules:
  - host: "www.log.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection
            port:
              number: 8080
```



​	加入hosts,测试,　https成功,http失败

```
[root@trainee ingress]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.3.100 www.log.local


curl https://www.log.local/ -k
Success! Master branch for guangda bank


curl www.log.local
<html>
<head><title>308 Permanent Redirect</title></head>
<body>
<center><h1>308 Permanent Redirect</h1></center>
<hr><center>nginx</center>
</body>
</html>
[root@trainee ingress]# curl http://www.log.local
<html>
<head><title>308 Permanent Redirect</title></head>
<body>
<center><h1>308 Permanent Redirect</h1></center>
<hr><center>nginx</center>
</body>
</html>

```



要想http, https都能正常访问,使用下面的配置,将  nginx.ingress.kubernetes.io/ssl-redirect配置为false

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: log-collection
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  tls:
  - hosts:
    - www.log.local
    secretName: ingress-secret
  rules:
  - host: www.log.local
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: log-collection
            port:
              number: 8080

```

测试:



```
[root@trainee ingress]# curl https://www.log.local/ -k
Success! Master branch for guangda bank


[root@trainee ingress]# curl http://www.log.local/
Success! Master branch for guangda bank
```



原因:

k8s ingress-nginx 默认是http 强制跳转到https的，可以通过Ingress 的annotations进行配置，nginx.ingress.kubernetes.io/ssl-redirect: ‘false’ # true 为强制跳转，完整配置如下：

https://www.65535.fun/?p=641





whoami ingress:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
spec:
  selector:
    matchLabels:
      app: whoami
  replicas: 2
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: quay.io/qxu/spring-boot-whoami
          ports:
            - containerPort: 8080
            
---
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: whoami
  type: ClusterIP

---
                       
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami
  annotations:
    #nginx.ingress.kubernetes.io/affinity: "cookie"
spec:
  ingressClassName: nginx 
  rules:
  - host: "whoami.user1.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: whoami
            port:
              number: 8080    
```

