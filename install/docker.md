修改现有docker ps为自动启动

```
docker update --restart always registry

```



1.24以后要安装cri-dockerd,可参考:

https://blog.csdn.net/weixin_38299857/article/details/125143330
