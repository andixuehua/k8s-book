## CMD和ENTRYPOINT的区别

区别可查看

https://segmentfault.com/a/1190000040781214



下载仓库:　https://github.com/mikerain/log-collection

或在本地目录:　/home/qxu/repo/log-collection

分别用以下dockerfile构建image然后使用docker run -it命令进入容器查看



### CMD

```
FROM docker.io/library/openjdk:8
ENV TZ=Asia/Shanghai

COPY init.sh /init.sh 
COPY target/log-collection-demo-0.0.1-SNAPSHOT.jar /usr/local/app.jar

EXPOSE 8080
ENTRYPOINT ["/bin/sh", "/init.sh"]
#CMD ["/bin/sh", "/init.sh"]


```

```
docker build -t  registry.crc.test:5000/log-collection:cmd .

docker run -it registry.crc.test:5000/log-collection:cmd /bin/bash


```





### ENTRYPOINT

```
FROM docker.io/library/openjdk:8
ENV TZ=Asia/Shanghai

COPY init.sh /init.sh 
COPY target/log-collection-demo-0.0.1-SNAPSHOT.jar /usr/local/app.jar

EXPOSE 8080
ENTRYPOINT ["/bin/sh", "/init.sh"]
#CMD ["/bin/sh", "/init.sh"]
```



```
docker build -t  registry.crc.test:5000/log-collection:ep .

#以下命令不能成功
docker run -it registry.crc.test:5000/log-collection:cmd /bin/bash

#下面可以成功
docker run -it   --entrypoint /bin/bash registry.crc.test:5000/log-collection:ep 
```



## Dockerfile优化

在:　/root/ops/training/dockerfile

### image size优化



bad:

```
FROM ubuntu:trusty
ENV VER     3.0.0
ENV TARBALL http://download.redis.io/releases/redis-$VER.tar.gz
# ==> Install curl and helper tools...
RUN apt-get update
RUN apt-get install -y  curl make gcc
# ==> Download, compile, and install...
RUN curl -L $TARBALL | tar zxv
WORKDIR  redis-$VER
RUN make
RUN make install
#...
# ==> Clean up...
WORKDIR /
RUN apt-get remove -y --auto-remove curl make gcc
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*  /redis-$VER
#...
CMD ["redis-server"]

```



good:

```
FROM ubuntu:trusty
ENV VER     3.0.0
ENV TARBALL http://download.redis.io/releases/redis-$VER.tar.gz
# ==> Install curl and helper tools...

RUN echo "==> Install curl and helper tools..."  && \
    apt-get update                      && \
    apt-get install -y  curl make gcc   && \
    \
    echo "==> Download, compile, and install..."  && \
    curl -L $TARBALL | tar zxv  && \
    cd redis-$VER               && \
    make                        && \
    make install                && \
    echo "==> Clean up..."  && \
    apt-get remove -y --auto-remove curl make gcc  && \
    apt-get clean                                  && \
    rm -rf /var/lib/apt/lists/*  /redis-$VER


#...
CMD ["redis-server"]

```



```
docker build -t redis:bad . -f Dockerfile.bad 

docker build -t redis:good . -f Dockerfile.good
```



### 构建速度优化

经常变化的文件放在后面,镜像是分层存储的

build firstly, then change test.txt file, and rebuild it, 

slow, 

```
ROM ubuntu:trusty
ENV VER     3.0.0
ENV TARBALL http://download.redis.io/releases/redis-$VER.tar.gz


ADD test.txt /tmp
# ==> Install curl and helper tools...

RUN apt-get update
RUN apt-get install -y  curl make gcc
# ==> Download, compile, and install...
RUN curl -L $TARBALL | tar zxv
WORKDIR  redis-$VER
RUN make
RUN make install
#...
# ==> Clean up...
WORKDIR /
RUN apt-get remove -y --auto-remove curl make gcc
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*  /redis-$VER
#...
CMD ["redis-server"]

```



quick:

```
FROM ubuntu:trusty
ENV VER     3.0.0
ENV TARBALL http://download.redis.io/releases/redis-$VER.tar.gz


# ==> Install curl and helper tools...
RUN apt-get update
RUN apt-get install -y  curl make gcc
# ==> Download, compile, and install...
RUN curl -L $TARBALL | tar zxv
WORKDIR  redis-$VER
RUN make
RUN make install
#...
# ==> Clean up...
WORKDIR /
RUN apt-get remove -y --auto-remove curl make gcc
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*  /redis-$VER


ADD test.txt /tmp

```





```
docker build -t redis:slow . -f Dockerfile.slow 

docker build -t redis:quick . -f Dockerfile.quick
```





# docker中copy和add指令有什么区别

https://www.php.cn/docker/484994.html

区别：COPY指令不支持从远程URL获取资源，只能从执行docker build所在的主机上读取资源并复制到镜像中；

而ADD指令支持从远程URL获取资源，可以通过URL从远程服务器读取资源并复制到镜像中





**ADD指令有如下的优越性：**

- 1、如果源路径是个文件，且目标路径是以 / 结尾， 则docker会把目标路径当作一个目录，会把源文件拷贝到该目录下。
  如果目标路径不存在，则会自动创建目标路径。
- 2、如果源路径是个文件，且目标路径是不是以 / 结尾，则docker会把目标路径当作一个文件。
  如果目标路径不存在，会以目标路径为名创建一个文件，内容同源文件；
  如果目标文件是个存在的文件，会用源文件覆盖它，当然只是内容覆盖，文件名还是目标文件名。
  如果目标文件实际是个存在的目录，则会源文件拷贝到该目录下。 注意，这种情况下，最好显示的以 / 结尾，以避免混淆。
- 3、如果源路径是个目录，且目标路径不存在，则docker会自动以目标路径创建一个目录，把源路径目录下的文件拷贝进来。
  如果目标路径是个已经存在的目录，则docker会把源路径目录下的文件拷贝到该目录下。
- 4、如果源文件是个归档文件（压缩文件），则docker会自动帮解压。





ARG,参数化构建,可用于通用化Dockerfile的实现

```
ARG
docker build -f Dockerfile-arg  --build-arg JAR=log-collection-demo-0.0.1-SNAPSHOT.jar -t aa:v1 .


more Dockerfile-arg 
FROM docker.io/library/openjdk:8
ENV TZ=Asia/Shanghai
#RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
#RUN touch /tmp/test.txt &&  echo "aaa" >/tmp/test.txt
ARG JAR
COPY init.sh /init.sh 
COPY target/$JAR /usr/local/app.jar

EXPOSE 8080
#ENTRYPOINT ["/bin/sh", "/init.sh"]
CMD ["/bin/sh", "/init.sh"]

#CMD ["java", "$JAVA_OPTIONS", "-jar", "/usr/local/app.jar"]

```

