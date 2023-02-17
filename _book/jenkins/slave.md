配置kubernetes pod as jenkins salve



1, add jenkins credential :

kind: secrete file

from your kubeconfig file



2, get k8s api url

kubectl config view 



3, add a kubernetes cloud 

url and credential use from two steps,

指定一个test namespace

test connection, 





参考：

https://devopscube.com/jenkins-build-agents-kubernetes/