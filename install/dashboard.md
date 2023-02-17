安装前查看版本匹配:

https://github.com/kubernetes/dashboard/releases

v.124 ->2.6.1



k8s v.19 dashboard 安装 2.0.5 可以成功，需要自定义证书
https://github.com/kubernetes/dashboard
kubectl --namespace kubernetes-dashboard  port-forward svc/kubernetes-dashboard 443 --address 192.168.146.91  (这样可以)

使用token进行login



创建管理员token：
https://blog.csdn.net/weixin_38320674/article/details/107328982



执行这步，就可以给现有的sa授权，可通过 dashboard访问 所有的nm

```
kubectl create clusterrolebinding dashboard-cluster-admin  --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:kubernetes-dashboard

kubectl get secret -n kubernetes-dashboard
NAME                               TYPE                                  DATA   AGE
default-token-tmhcf                kubernetes.io/service-account-token   3      24d
kubernetes-dashboard-certs         Opaque                                0      24d
kubernetes-dashboard-csrf          Opaque                                1      24d
kubernetes-dashboard-key-holder    Opaque                                2      24d
kubernetes-dashboard-token-lgl5p   kubernetes.io/service-account-token   3      24d


kubectl  describe  secret kubernetes-dashboard-token-ngcmg  -n   kubernetes-dashboard

#token需要删除回车符，在一行内copy

```



给特定的用户赋权访问自己的nm 通过 dashboard,

```
kubectl create ns test1
kubectl create serviceaccount test1-admin -n test1
kubectl create rolebinding test1-admin -n test1 --clusterrole=cluster-admin --serviceaccount=test1:test1-admin
kubectl get secret -n test1

kubectl describe secret test1-admin-token-h5km5 -n test1

#use the token to access dashboard UI, 在UI中输入test1 nm, 一般namespace不会自动显示出来，需要输入

#token需要删除回车符，在一行内copy

如果以上没有发现secret,使用下面的命令创建一个token(估计是k8s版本差异)
 kubectl -n test1 create token test1-admin

```

以上,也要参考:　https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md



新版本的dashboard 2.6.1+ sa没有 token,用下面的命令给sa创建一个token,使用时,把要删除回车符，在一行内copy

```
 kubectl -n kubernetes-dashboard create token kubernetes-dashboard
eyJhbGciOiJSUzI1NiIsImtpZCI6InNxd1A5WTdHRXhRRW9ZRjJhR1ljUmVmZmtMOC1hUmUwSndVQm9fRDJyMkkifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjY1MTk3OTM4LCJpYXQiOjE2NjUxOTQzMzgs
ImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInVpZCI
6IjdkNTcyOGQ2LTljNjEtNGU2NC05MjMwLTdlZDAyYWM2ZmMxYiJ9fSwibmJmIjoxNjY1MTk0MzM4LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQifQ.yUkejK4PeIz56FaqhH681YG4Yrl08L-flJ9
xfYaaLl0YI9Cubfcd3fcdeZGLY7WYZXyjYHpszb7Sp1K5MXd8b6uEKEtGIx3EKGMDI_fWxB-jOvzhe72PDa5X5bPKPGu7HRZR6eyXI7Z-VznpOi1hWofgNvw1axw4wKnNMO649vqG6tbi4r61QmaQgLoKB8vBMgY3F3uroOppJkzV5sfT1EtSQgxtgHpfTZTRL8Nj0_uyz7TRB7hnL3
FSdEaUbwKEtVBa9365vdEizVkbSbRRIF9jfjn4SysfEvyspLskXrssAdVHm6LkRmY71muNuaPOccD21ZT43NRf-zze2kV4wA

```



可以修改svc为nodeport提供对外访问

```
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard

    sessionAffinity: None
    type: NodePort


[root@master01 install]# kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.96.238.150   <none>        8000/TCP        47m
kubernetes-dashboard        NodePort    10.96.125.3     <none>        443:32459/TCP   47m

```





ingress:目前不可用，因为证书问题

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubernetes-dashboard
  annotations:
    kubernetes.io/ingress.class: "nginx"
  namespace: kubernetes-dashboard
spec:
  rules:
  - host: dashboard.apps.test.crc.test
    http:
      paths:
      - path:
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
```





taikakng的 dashboard toke使用了,要base64 -d后使用 , admin

```
[root@trainee ~]# kubectl get secret -n kube-system hyperkuber -o yaml|grep token:|awk '{print $2}'|base64 -d
eyJhbGciOiJSUzI1NiIsImtpZCI6ImJ1TEh0Ulc3VnVkX1dpbXlvUVBlUkpvN2MxaXg3bGpoN0JuUWtXMDJ3MFUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJoeXBlcmt1YmVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Imh5cGVya3ViZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIzYjdjMWMzMS00MWQ0LTRjNjYtYjgxMC1lZWU1MTI4NzU3ZWIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06aHlwZXJrdWJlciJ9.jpt6RL7dPzMBnwHhJbvwkzMnFXfrUWcnu90YUl4V4De7f3e-eytpzZ4tzqC1Orw2i4UPE7GbQ2slypQbi_u4d4RRnWLq5aV4i8Mu2rfTsD-3Ykfp4FP-PTmIObThkUQgAQuOalbtbl7Un9VE81hjOkyKpsoPDyZg5PH29-2zMF0KMq7Y7JYxD3sYYrRl5Ck-1MDUmaSy1qf5cAxgKEXsYWK_n7sRSF-Lc6Cwf0DZDVXu_L-qWy_Rr55PAJRRBj2sIyxaVrVhrsetBNv2XawmyZPOXv7vdv-JG653QEZ3NqwJhIhTcyygiO4lKJ-nt175RuihUWaE_ofNjreap27QfQ

```



测试用户　token:

