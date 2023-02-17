https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/



## Kubernetes中的用户

参考：https://zhuanlan.zhihu.com/p/43237959

K8S中有两种用户(User)——服务账号(ServiceAccount)和普通意义上的用户(User)



# 使用 RBAC 鉴权

基于角色（Role）的访问控制（RBAC）是一种基于组织中用户的角色来调节控制对计算机或网络资源的访问的方法。

RBAC 鉴权机制使用 `rbac.authorization.k8s.io` [API 组](https://kubernetes.io/zh-cn/docs/concepts/overview/kubernetes-api/#api-groups-and-versioning)来驱动鉴权决定， 允许你通过 Kubernetes API 动态配置策略。

要启用 RBAC，在启动 [API 服务器](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-apiserver)时将 `--authorization-mode` 参数设置为一个逗号分隔的列表并确保其中包含 `RBAC`。

```shell
kube-apiserver --authorization-mode=Example,RBAC --<其他选项> --<其他选项>
```

## API 对象

RBAC API 声明了四种 Kubernetes 对象：**Role**、**ClusterRole**、**RoleBinding** 和 **ClusterRoleBinding**。你可以像使用其他 Kubernetes 对象一样，通过类似 `kubectl` 这类工具[描述对象](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/kubernetes-objects/#understanding-kubernetes-objects), 或修补对象。







## 为用户生成证书

假设我们操作的用户名为tom

1. 首先需要为此用户创建一个私钥
   `openssl genrsa -out tom.key 2048`
2. 接着用此私钥创建一个csr(证书签名请求)文件，其中我们需要在subject里带上用户信息(CN为用户名，O为用户组)
   `openssl req -new -key tom.key -out tom.csr -subj "/CN=tom/O=MGM"`
   其中/O参数可以出现多次，即可以有多个用户组
3. 找到K8S集群(API Server)的CA证书文件，其位置取决于安装集群的方式，通常会在`/etc/kubernetes/pki/`路径下，会有两个文件，一个是CA证书(ca.crt)，一个是CA私钥(ca.key)
4. 通过集群的CA证书和之前创建的csr文件，来为用户颁发证书
   `openssl x509 -req -in tom.csr -CA path/to/ca.crt -CAkey path/to/ca.key -CAcreateserial -out tom.crt -days 365`
   -CA和-CAkey参数需要指定集群CA证书所在位置，-days参数指定此证书的过期时间，这里为365天



## 角色(Role)

在RBAC中，角色有两种——普通角色(Role)和集群角色(ClusterRole)，ClusterRole是特殊的Role，相对于Role来说：

- Role属于某个命名空间，而ClusterRole属于整个集群，其中包括所有的命名空间
- ClusterRole能够授予集群范围的权限，比如node资源的管理，比如非资源类型的接口请求(如"/healthz")，比如可以请求全命名空间的资源(通过指定 --all-namespaces)



```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: a-1
  name: admin
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```



## 将角色和用户绑定

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin-binding
  namespace: a-1
subjects:
- kind: User
  name: user1
  apiGroup: ""
roleRef:
  kind: Role
  name: admin
  apiGroup: ""
```





## 添加命名空间管理员的另一种方式

前面说过，K8S内置了一个名为admin的ClusterRole，所以实际上我们无需创建一个admin Role，直接对集群默认的admin ClusterRole添加RoleBinding就可以了

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin-binding
  namespace: a-1
subjects:
- kind: User
  name: tom
  apiGroup: ""
roleRef:
  kind: ClusterRole
  name: admin
  apiGroup: ""
```



可以看关注 三个clusterrole， admin,view,edit ,用上面这个方式用用户nm增加相关的用户权限

```
kubectl get clusterrole
kubectl get clusterrolebinding

kubectl get role -A
kubectl get rolebinding -A
```



新机器，新用户配置完用户后，如果 出现以下问题：

```
 kubectl config set-credentials tom --client-certificate=/root/tom.crt --client-key=/root/tom.key
 kubectl get pod -n test1
The connection to the server localhost:8080 was refused - did you specify the right host or port?


```

在目标机器上使用以下/root/.kube/config

```
# cat config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1ESXdOakUwTVRJME1Gb1hEVE15TURJd05
ERTBNVEkwTUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSjdCCi84YXZkak0yMUZGNVpMVXU1Smd6dERodmlKeDNpblRrM3ozaXF1NlV3cnRwMTgyWW9ZZUl3a1ZCa2JaNzdMVzgKajJ4Qm
hLaWNEN3dyZTdmYmV1MFMvcU93cWU2N0x1aGRVMERFZHRGQ2FyVjBiZW5nUm1CZU52ODZkRlZFa2JSagpoRTFYQ1g5U2M3QVJIMG5EVlNCK2xPT3ZiUkhOV0U3Ti9UMlQwdFlIcWRoUE5uc2VaWUtFaDlhODlXWmM5TUkvCnNQOXZGMndKcU9KWG9CTVZMbVhuSllmVHNMTHFHaERWS
0d1UjdmUFMweHlQbG0xeGRCelRUdEJMVFhLTzJUeW0KUWMvMk1CbnpXbVhDVWNzRDE5SWZWb0YzdEJDMWs2am55ekZlMjNwb3NiS01US2htSDhwbU8wTjFESXNjWTR0UApBL0xGMkVrOXJqT2Zsd2pCYVJjQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRF
d0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZBaHFTR003UExaQXIvNzZYL01TMUdnc3BnMTVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCODRTdm51ZlpEdnN4Qml1OVRRK0VBbmQrY1dlS1pGTlpJaGhlQ2o0cERjeExaQVFkcwpBNkhHRERJbUpJY29IQUVnT3NhS21VT0J
3SFFSM041R2F6cmJvQzBKRW8vTWwydGNrazZyR3dIMExuZWZGYkxTClVxQ0tNaVgwNHYwSzRLckp2ZnZNZ24rU095SEJTM1ZRSTdUbkxCNngyeW43aHFKa3VHd2ZqSWQ1bm9LWExDcVoKM1lLdzZQVlE1OVVlWHFiSkNaTXp5WXRHeUxlN1diVkZyUDU5SjRzUnZvcjJwL0VsV1JIRX
VrbWQ0K096aFVWaQpsQk8zQU9kdklHOVQvK1k5YkIwVWNhN0toWHhkMnJ0WmgrWHV2YUlMZ01WZXgvSS9hYXo5bUcyUlFvNmtndGtGCnFSekhQeG1ZSTEzdUVmVSt2LzJ3RkUxQ1pIN3VDaGYrZFdHcAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.146.91:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: test1
    user: tom
  name: tom@kubernetes
current-context: "tom@kubernetes"
kind: Config
preferences: {}
users:
- name: tom
  user:
    client-certificate: /root/tom.crt
    client-key: /root/tom.key


```



其中：

```
clusters：部分来自原有k8s安装完成后的config文件 
/etc/kubernetes/admin.conf

users：
使用命令生成
kubectl config set-credentials tom --client-certificate=/root/practice/user/tom.crt --client-key=/root/practice/user/tom.key 生成

contexts：
使用命令生成
kubectl config set-context tom@kubernetes --cluster=kubernetes --namespace=test1 --user=tom


current-context
kubectl config use-context tom@kubernetes
kubectl get node  --- will failed, no permission

#switch back to admin
kubectl config use-context kubernetes-admin@kubernetes


kubectl get RoleBinding  admin-binding -n test1



以上可参考 192.168.146.91:
/root/practice/user
```



```

```

可参考：

https://zhuanlan.zhihu.com/p/43237959

https://www.cnblogs.com/ooops/p/14536715.html



实验,建立三个帐号或一个帐号分别给某个namespace的view, edit ,admin权限,以view为例,　tested by mike

```
kubectl create namespace test-rbac

kubectl describe clusterrole view
kubectl describe clusterrole edit
kubectl describe clusterrole admin

#创建一个serviceaccount,一定在kube-system ,不要以项目下
kubectl create sa sa-blue -n kube-system 

#将其进行绑定
kubectl create rolebinding sa-blue-view --clusterrole=view --serviceaccount=kube-system:sa-blue -n test-rbac

kubectl get rolebinding sa-blue-view  -n test-rbac
#查看rolebinding token
# kubectl get secret -n kube-system|grep sa-blue
sa-blue-view                            kubernetes.io/service-account-token   3      5m22s

kubectl get secret sa-blue-view-token-swp7v -o yaml -n kube-system
```

**将以上的token　base64 -d解密后放在下面的kubeconfig中***

```
kubectl get secret sa-blue-token-hxr2j -o yaml -n kube-system|grep " token:"|awk '{print $2}'|base64 -d
eyJhbGciOiJSUzI1NiIsImtpZCI6ImEzNkhvUVM1M3FuUTBvajcxOEVzcTFxeFBhQWJkRnFkVEJkQVBteG95T00ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIs
Imt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJzYS1ibHVlLXRva2VuLWh4cjJqIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InNhLWJsdWUiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3N
lcnZpY2UtYWNjb3VudC51aWQiOiI0YTk3ZTkwMy1kZTliLTQ4MWYtYjdlMS0wNGRmNThiMWU5NWMiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06c2EtYmx1ZSJ9.ozek38WWxW5pdROVGcH4L_k1C9GIHtcgNR8ldcEGtTTxuJSWzChV78GtZ6RXRD7ZD
DwcMd740e4q4RdSi92br2GJpaCeVil6mUqSXw2WNBV95KEZf-_1b6ueh6r-lWjOVN4iT1Aoo1BRH8uyaVnBftXazoSfb5HdIsvLCTWAVD7go5qgxEHgyHWrYDwY22s5JGJ9ByGgU1NC_e8K1Tag63MdYs4BtQZD5jcli0D0i_pvab8AxPgWZBirwp16UsxtYVJn4XBT4ggfr2EndFL-
3N166nv1WzJYQy68QbdNLG7tsMSszdLljGjA7zUKiIkXszozve0wUzrU4MS4hwRZTw
```



copy一个现有的kubeconfig文件,替换user下面的部分,保存为sa-blue.kubeconfig

```
users:
- name: my-service
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6ImEzNkhvUVM1M3FuUTBvajcxOEVzcTFxeFBhQWJkRnFkVEJkQVBteG95T00ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlL
XN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJyZWFkb25seS10b2tlbi1zd3A3diIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJyZWFkb25seSIsImt1YmVybmV0ZXMuaW8vc2Vydmlj
ZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjY2OTdhOTZiLWQ5NTUtNGQxYy1hNDk5LTlhN2M3MDIyMjJkZCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpyZWFkb25seSJ9.Rx6cPIJQVdPAZtN7AoReS-EuP-q--gHOkUq0o-pueoVdhx00Jx
elx0F0KsVHR-OlyrX_PNYpDS0ZmvJShuy1tnRnY6-pJcxxYHht6x2r6X85PpfP4F-nC04TFRlS3memNjNP8mVZJDy3Y_9xi5f4hkAWS4KO6jDQggJ3N-Lc26tfsnhosh6u6U6eNkw83QsYAE80Eqtqg-87P7NsQyP6E0Q6k69IzFbS0CYQdHg49OYulA8Zil2lQkrEqWjxACWDDvn8u
2q3tCuaDzkJTdIIpuNbKbJaTnXB2pAjHQfeM8ALdilqZAvbZ_rXujHIdonIaYZySFEr4co6fl4L84r3sw
```



测试

```
 kubectl --kubeconfig=./sa-blue.kubeconfig get pod -n test-rbac
 
 kubectl --kubeconfig=token-blue.kubeconfig get pod -n default
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:kube-system:sa-blue" cannot list resource "pods" in API group "" in the namespace "default"
[root@kubernetes-master ~/practice/user] ens33 = 192.168.146.91
# kubectl --kubeconfig=token-blue.kubeconfig get pod -n test
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:kube-system:sa-blue" cannot list resource "pods" in API group "" in the namespace "test"
[root@kubernetes-master ~/practice/user] ens33 = 192.168.146.91
# kubectl --kubeconfig=token-blue.kubeconfig get pod -n test-rbac
No resources found in test-rbac namespace.
[root@kubernetes-master ~/practice/user] ens33 = 192.168.146.91
#

测试创建权限,无
# kubectl --kubeconfig=token-blue.kubeconfig create sa test -n test-rbac
Error from server (Forbidden): serviceaccounts is forbidden: User "system:serviceaccount:kube-system:sa-blue" cannot create resource "serviceaccounts" in API group "" in the namespace "test-rbac"

```



添加cluster edit 权限

```
kubectl create rolebinding sa-blue-edit --clusterrole=edit --serviceaccount=kube-system:sa-blue -n test-rbac

检查　sa-blue-token-hxr2j无变化,也没新增加secret
kubectl get secret -n kube-system

测试edit 权限,可以创建sa 成功
kubectl --kubeconfig=token-blue.kubeconfig create sa test -n test-rbac
serviceaccount/test created

#　list role失败,　edit和admin的一个区别就是edit没有加权限的功能,admin可以有
# kubectl --kubeconfig=token-blue.kubeconfig get role -n test-rbac
Error from server (Forbidden): roles.rbac.authorization.k8s.io is forbidden: User "system:serviceaccount:kube-system:sa-blue" cannot list resource "roles" in API group "rbac.authorization.k8s.io" in the namespac
e "test-rbac"

```



添加cluster admin 权限

```
kubectl create rolebinding sa-blue-admin --clusterrole=admin --serviceaccount=kube-system:sa-blue -n test-rbac

测试
# kubectl --kubeconfig=token-blue.kubeconfig get role -n test-rbac

# kubectl --kubeconfig=token-blue.kubeconfig get role -n test-rbac
No resources found in test-rbac namespace.

测试其他项目,失败
# kubectl --kubeconfig=token-blue.kubeconfig get role -n test
Error from server (Forbidden): roles.rbac.authorization.k8s.io is forbidden: User "system:serviceaccount:kube-system:sa-blue" cannot list resource "roles" in API group "rbac.authorization.k8s.io" in the namespac
e "test"

```

查看

```
# kubectl get rolebinding -n test-rbac
NAME            ROLE                AGE
sa-blue-admin   ClusterRole/admin   16m
sa-blue-edit    ClusterRole/edit    27m
sa-blue-view    ClusterRole/view    41m

```



```
# kubectl get rolebinding sa-blue-admin -o yaml  -n test-rbac
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:	
  creationTimestamp: "2022-09-29T01:40:31Z"
  name: sa-blue-admin
  namespace: test-rbac

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: sa-blue
  namespace: kube-system

```





------

注意在1.24以后,serice acount的secret不会自动创建了,要手工创建

```
kubectl create sa sa-user1 -n kube-system
kubectl create rolebinding sa-user1-admin --clusterrole=admin --serviceaccount=kube-system:sa-user1 -n user1

#这个是另加一个clusterrole
kubectl create clusterrolebinding sa-readonly --clusterrole=view --serviceaccount=kube-system:sa-user1 
```

https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md#urgent-upgrade-notes

https://stackoverflow.com/questions/72256006/service-account-secret-is-not-listed-how-to-fix-it

#in kube-system , create secret

```
apiVersion: v1
kind: Secret
metadata:
  name: user1-token
  annotations:
    kubernetes.io/service-account.name: sa-user1
type: kubernetes.io/service-account-token


```



在user1　ns中添加一个view权限,让其他用户也能看user1中的资源

```
kubectl create rolebinding sa-user1-view --clusterrole=view --serviceaccount=kube-system:sa-user2 -n user1

#修改rolebinding ,加入subjects就可以授权其他用的sa 帐号
kubectl get rolebinding sa-user1-view -n user1 -o yaml
 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-user1-view
  namespace: user1
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
- kind: ServiceAccount
  name: sa-user2
  namespace: kube-system
  subjects:
- kind: ServiceAccount
  name: sa-user3
  namespace: kube-system
  subjects:
- kind: ServiceAccount
  name: sa-user4
  namespace: kube-system

```



##### 解释:

resources,apiGroups: 有哪些可以通过以下命令查询, resource是第一列的name, apigroups是第二列的apiversion

```
[root@trainee prepare]# kubectl api-resources 
NAME                              SHORTNAMES         APIVERSION                             NAMESPACED   KIND
bindings                                             v1                                     true         Binding
componentstatuses                 cs                 v1                                     false        ComponentStatus
configmaps                        cm                 v1                                     true         ConfigMap
endpoints                         ep                 v1                                     true         Endpoints
events                            ev                 v1                                     true         Event
limitranges                       limits             v1                                     true         LimitRange
namespaces                        ns                 v1                                     false        Namespace
nodes                             no                 v1                                     false        Node
persistentvolumeclaims            pvc                v1                                     true         PersistentVolumeClaim
persistentvolumes                 pv                 v1                                     false        PersistentVolume
pods                              po                 v1                                     true         Pod
podtemplates                                         v1                                     true         PodTemplate
replicationcontrollers            rc                 v1                                     true         ReplicationController
resourcequotas                    quota              v1                                     true         ResourceQuota
secrets                                              v1                                     true         Secret
serviceaccounts                   sa                 v1                                     true         ServiceAccount
services                          svc                v1                                     true         Service
mutatingwebhookconfigurations                        admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
validatingwebhookconfigurations                      admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds           apiextensions.k8s.io/v1                false        CustomResourceDefinition
apiservices                                          apiregistration.k8s.io/v1              false        APIService
controllerrevisions                                  apps/v1                                true         ControllerRevision
daemonsets                        ds                 apps/v1                                true         DaemonSet
deployments                       deploy             apps/v1                                true         Deployment

```



```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: xi-{{instanceId}}
  name: deployment-creation
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["batch", "extensions"]
  resources: ["jobs"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```



##### 建立一个cluster-read cluster-role,给特定的sa或user使用,只能看特定的数据,不能查看secret等数据

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods","nodes","deployments","statefulsets","daemonsets","jobs","configmaps","services","endpoints","namespaces","ingresses"]
  verbs: ["get", "list", "watch"]
 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-reader-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-reader
subjects:
- kind: ServiceAccount
  name: sa-user2
  namespace: kube-system



```



注意以上的配置,用户user2并不能获得 ingress的查看权限

```
kubectl get ingress -n kube-system

Error from server (Forbidden): ingresses.networking.k8s.io is forbidden: User "system:serviceaccount:kube-system:sa-user2" cannot list resource "ingresses" in API group "networking.k8s.io" in the namespace "kube-system"

```

可使用下面的配置,加入* in apiGroups

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: ["*"]
  resources: ["pods","nodes","deployments","statefulsets","daemonsets","jobs","configmaps","services","endpoints","namespaces","ingresses"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-reader-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-reader
subjects:
- kind: ServiceAccount
  name: sa-user2
  namespace: kube-system

```

或者使用下面的配置:,增加一个networking.k8s.io

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: [""]
  resources: ["pods","nodes","deployments","statefulsets","daemonsets","jobs","configmaps","services","endpoints","namespaces"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get", "list", "watch"]
 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-reader-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-reader
subjects:
- kind: ServiceAccount
  name: sa-user2
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user3
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user4
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user5
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user6
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user7
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user8
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user9
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user10
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user11
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user12
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user13
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user14
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user15
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user16
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user17
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user18
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user19
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user20
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user21
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user22
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user23
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user24
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user25
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user26
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user27
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user28
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user29
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user30
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user31
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user32
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user33
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user34
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user35
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user36
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user37
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user38
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user39
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user40
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user41
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user42
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user43
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user44
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user45
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user46
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user47
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user48
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user49
  namespace: kube-system
- kind: ServiceAccount
  name: sa-user50
  namespace: kube-system

```



演示clusterrole的授权

/root/practice/prepare/cluster-reader.yaml



#####  core API group

https://kubernetes.io/zh-cn/docs/reference/using-api/

一般就是下面两个

v1

apps/v1

