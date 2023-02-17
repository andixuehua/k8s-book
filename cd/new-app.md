- #### 使用命令行部署应用的几个方式



-  from public docker image

```shell

kubectl create deployment log-collection --image=quay.io/qxu/log-collection 
kubectl expose deployment log-collection  --port=8080
```

- from yaml

  [参考](../basic/deployment.md)

  

  


