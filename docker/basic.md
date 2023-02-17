2.1 Build Image
------------------
	sudo docker build --tag my-image ./app

2.2 See Build History
---------------------
	sudo docker history my-image

2.3 List All Images
---------------------
	sudo docker images

2.4 Starting a Docker Container
-------------------------------
	sudo docker run -d -p 8090:8080 --name="my-container" my-image

2.5 See Build History
-------------------------------
	sudo docker logs my-container

2.6 List all Container
-------------------------------
	sudo docker ps
	sudo docker ps --all

2.7 Run Application through CURL
-------------------------------
	curl http://localhost:8090/greeting

2.8 SSH into a running container
-------------------------------
	sudo docker exec -it my-container /bin/bash
	exit
2.9 Stopping a Container
-------------------------------
	sudo docker stop my-container

2.10 Deleting a Container
-------------------------------
	sudo docker rm my-container

2.11 Deleting an Image
-------------------------------
	sudo docker rmi my-image





docker search 会从下面的search

```
https://hub.docker.com/
```



### .1 问题

普通用户直接使用 Docker 会报错权限不足：

```javascript
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/create: dial unix /var/run/docker.sock: connect: permission denied.
```

复制

这主要是因为 Docker 进程使用 Unix Socket，而 `/var/run/docker.sock` 需要 root 权限才能进行读写操作。

>  因此，如果不考虑安全问题的话，也可以使用 root 权限直接改写 `/var/run/docker.sock` 文件的权限，使得其对所有普通用户都有读写权限： sudo chmod 666 /var/run/docker.sock 

### 3.2 方案

参考[官方说明](https://docs.docker.com/engine/install/linux-postinstall/)，使用 root 权限创建一个 `docker` 组，并将普通用户加入到该组中，然后刷新一下 `docker` 组使其修改生效即可：

```javascript
sudo groupadd docker			# 有则不用创建
sudo usermod -aG docker USER	# USER 为加入 docker 组的用户
newgrp docker					# 刷新 docker 组
docker run hello-world			# 测试无 root 权限能否使用 docker
```

复制

>  【注】如果在运行上述命令时，`USER` 一直是登录状态，则也要使用 `newgrp docker` 来刷新以获取改变。 