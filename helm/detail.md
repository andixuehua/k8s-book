使用Helm后会生成相应的缓存文件，使用过程中必要时可以主动清空。目录如下

- ~/.config/helm
- ~/.cache/helm



> ### 常用源

```bash
NAME                            URL
azure                 https://mirror.azure.cn/kubernetes/charts
aliyun                https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
elastic               https://helm.elastic.co
gitlab                https://charts.gitlab.io
harbor                https://helm.goharbor.io
bitnami               https://charts.bitnami.com/bitnami
incubator             https://kubernetes-charts-incubator.storage.googleapis.com
google                https://kubernetes-charts.storage.googleapis.com
ingress-nginx         https://kubernetes.github.io/ingress-nginx
kubernetes-dashboard  https://kubernetes.github.io/dashboard/
```



### 常用命令

**helm search hub**：会去helm hub上搜一些其他大佬们共享出来的chart。

**helm search repo**：会搜那些你已经通过 `helm repo add`添加到本地helm client的repository，这个搜索完全在搜本地数据，无需联外网。



```
#在远hub上search mysql
helm search hub mysql

helm search hub mysql --max-col-width=100

#以上命令在远程的hub　https://artifacthub.io/上查找


#指定远程或本地的hub进行查询
helm search hub mysql --endpoint https://hub.helm.sh


#在本地repo　search mysql
[root@trainee helm]# helm search repo mysql
Error: no repositories configured

```





```bash
helm repo list
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

#以上命令会加入配置到下面的文件中,记录本地的helm repo
~/.config/helm/repositories.yaml

#从本地search
helm search repo nginx
NAME                   	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyun/nginx-ingress   	0.9.5        	0.10.2     	An nginx Ingress controller that uses ConfigMap...
aliyun/nginx-lego      	0.3.1        	           	Chart for nginx-ingress-controller and kube-lego  
aliyun/gcloud-endpoints	0.1.0        	           	Develop, deploy, protect and monitor your APIs ...


#下载到本地目录,查看
helm pull aliyun/mysql
[root@trainee helm]# ll
total 12
-rw-r--r--. 1 root root 10830 Nov  8 20:32 mysql-0.3.5.tgz

```





## 安装一个包: helm install

```
#从repo安装 
helm install mysql-test aliyun/mysql -n test1

helm install happy-panda aliyun/mariadb -n test1

#使用aliyun repo常见的问题,就是template中的配置api版本较老,建议使用其他较新的源,如
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: resource mapping not found for name: "happy-panda-mariadb" namespace: "" from "": no matches for kind "Deployment" in version "extensions/v1beta1"

#如下,可以在k8s　1.24上安装成功
helm install happy-panda bitnami/mariadb -n test1    (需要default storageclass)
helm install nginx-test bitnami/nginx -n test1   (建议使用这个进行测试)

#查看
helm list -n test1
helm status  happy-panda -n test1

[root@trainee helm]# helm list -n test1
NAME       	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART         	APP VERSION
happy-panda	test1    	1       	2022-11-08 21:01:23.1252383 -0500 EST  	deployed	mariadb-11.3.4	10.6.10    
nginx-test 	test1    	1       	2022-11-08 21:04:09.839488038 -0500 EST	deployed	nginx-13.2.13 	1.23.2     

[root@trainee helm]# kubectl get pod -n test1
NAME                        READY   STATUS    RESTARTS   AGE
happy-panda-mariadb-0       1/1     Running   0          4m21s
nginx-test-cb4c79d4-5sscz   1/1     Running   0          94s


```



查看nginx 的svc配置 发现不是clusterip,查看values的配置为　LoadBalancer

```
[root@trainee nginx]# kubectl get svc -n test1
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-test   LoadBalancer   10.233.7.145   <pending>     80:32421/TCP   30m


# 查看当前的配置
helm show values bitnami/nginx 
```

更新指定使用ClusterIP

```
 helm upgrade nginx-test bitnami/nginx  --set service.type=ClusterIP
  helm upgrade nginx-test bitnami/nginx  --set service.type=LoadBalancer
```



安装过程中有两种方式传递配置数据：

- `--values` (或 `-f`)：使用 YAML 文件覆盖配置。可以指定多次，优先使用最右边的文件。

- `--set`：通过命令行的方式对指定项进行覆盖。

  



### 从本地安装,更新

```
 helm pull bitnami/nginx
 tar xfz nginx-13.2.13.tgz
 
 #修改 value.yaml中的service.type: ClusterIP
 service:
  ## @param service.type Service type
  ##
  type: ClusterIP

 cd nginx
 
helm upgrade nginx-test   ./ -f values.yaml
#or
helm install nginx-test ./ -f values.yaml
 
kubectl get svc -n test
 

#升级镜像,修改replica,注意不要全角空格
helm upgrade nginx-test   ./ -f values.yaml --set image.tag=latest
helm upgrade nginx-test   ./ -f values.yaml --set replicaCount=2 --set image.tag=latest


[root@trainee nginx]# kubectl get pod -n test1
NAME                          READY   STATUS              RESTARTS   AGE
nginx-test-5bd959f58f-mt7pn   0/1     ContainerCreating   0          31s
nginx-test-cb4c79d4-5sscz     1/1     Running             0          44m
nginx-test-cb4c79d4-69rsp     1/1     Running             0          61s

```



历史,rollback

```
[root@trainee nginx]# helm history nginx-test 
REVISION	UPDATED                 	STATUS    	CHART        	APP VERSION	DESCRIPTION     
3       	Tue Nov  8 21:33:02 2022	superseded	nginx-13.2.13	1.23.2     	Upgrade complete
4       	Tue Nov  8 21:34:16 2022	superseded	nginx-13.2.13	1.23.2     	Upgrade complete
5       	Tue Nov  8 21:34:25 2022	superseded	nginx-13.2.13	1.23.2     	Upgrade complete
6       	Tue Nov  8 21:37:36 2022	superseded	nginx-13.2.13	1.23.2     	Upgrade complete
7       	Tue Nov  8 21:37:50 2022	superseded	nginx-13.2.13	1.23.2     	Upgrade complete
8       	Tue Nov  8 21:39:33 2022	superseded	nginx-13.2.13	1.23.2     	Upgrade complete
9       	Tue Nov  8 21:47:14 2022	superseded	nginx-13.2.13	1.23.2     	Upgrade complete
10      	Tue Nov  8 21:47:21 2022	superseded	nginx-13.2.13	1.23.2     	Upgrade complete
11      	Tue Nov  8 21:47:46 2022	superseded	nginx-13.2.13	1.23.2     	Upgrade complete
12      	Tue Nov  8 21:48:16 2022	deployed  	nginx-13.2.13	1.23.2     	Upgrade complete


[root@trainee nginx]# helm rollback nginx-test 10 -n test1
Rollback was a success! Happy Helming!

[root@trainee nginx]# kubectl get pod -n test1
NAME                        READY   STATUS    RESTARTS   AGE
nginx-test-cb4c79d4-72gpl   1/1     Running   0          21s


```





### 删除　install

```
helm uninstall happy-panda -n test1

```





### 其他:

hub不能被加到本地repo中,参考

https://stackoverflow.com/questions/60994725/k8s-how-to-install-charts-from-the-helm-hub

```
root@trainee nginx]# helm repo add hub https://artifacthub.io/
Error: looks like "https://artifacthub.io/" is not a valid chart repository or cannot be reached: error unmarshaling JSON: while decoding JSON: json: cannot unmarshal string into Go value of type repo.IndexFile
```



### 使用harbor建立私有repo

https://blog.csdn.net/u013332975/article/details/120672173



# 比较好的helm3 常用命令 示例

https://www.kongzid.com/archives/helm7
