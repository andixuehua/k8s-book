### Dockerfile

```
FROM docker.io/library/python:3.7

# 镜像作者
MAINTAINER qxu

# 设置 python 环境变量

# 设置pip源为国内源
COPY pip.conf /root/.pip/pip.conf

# 在容器内创建mysite文件夹
RUN mkdir -p /var/www/html/mysite

# 设置容器内工作目录
WORKDIR /var/www/html/mysite

# 将当前目录文件加入到容器工作目录中（. 表示当前宿主机目录）
ADD . /var/www/html/mysite

# pip安装依赖
RUN pip install -r requirements.txt

cmd ["python3", "manage.py", "runserver", "0.0.0.0:8080"]

```



deployemnt & svc

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: projectapi
spec:
  selector:
    matchLabels:
      app: projectapi
  replicas: 1
  template:
    metadata:
      labels:
        app: projectapi
    spec:
      imagePullSecrets:
      - name: dockercred
      containers:
        - name: projectapi
          image: quay.io/qxu/projectapi:master
          #image: quay.io/qxu/projectapi:v2
          #image: hub.taikang1.local/vue/projectapi:master
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: projectapi
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: projectapi
  type: ClusterIP
```



```
 kubectl create secret generic dockercred \
     --from-file=.dockerconfigjson=/root/.docker/config.json \
     --type=kubernetes.io/dockerconfigjson -n user1

```

