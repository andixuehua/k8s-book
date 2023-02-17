## podman 常用的的配置文件



### */etc/containers/registries.conf*

远程仓库访问配置,包括搜索,insecure访问



### /etc/containers/storage.conf

本地存储目录配置,

存储driver配置, 参考:　https://docs.docker.com/storage/storagedriver/select-storage-driver/



| Driver            | Description                                                  |
| :---------------- | :----------------------------------------------------------- |
| `overlay2`        | `overlay2` is the preferred storage driver for all currently supported Linux distributions, and requires no extra configuration. |
| `fuse-overlayfs`  | `fuse-overlayfs`is preferred only for running Rootless Docker on a host that does not provide support for rootless `overlay2`. On Ubuntu and Debian 10, the `fuse-overlayfs` driver does not need to be used, and `overlay2` works even in rootless mode. Refer to the [rootless mode documentation](https://docs.docker.com/engine/security/rootless/) for details. |
| `btrfs` and `zfs` | The `btrfs` and `zfs` storage drivers allow for advanced options, such as creating “snapshots”, but require more maintenance and setup. Each of these relies on the backing filesystem being configured correctly. |
| `vfs`             | The `vfs` storage driver is intended for testing purposes, and for situations where no copy-on-write filesystem can be used. Performance of this storage driver is poor, and is not generally recommended for production use. |
| `aufs`            | The `aufs` storage driver Was the preferred storage driver for Docker 18.06 and older, when running on Ubuntu 14.04 on kernel 3.13 which had no support for `overlay2`. However, current versions of Ubuntu and Debian now have support for `overlay2`, which is now the recommended driver. |
| `devicemapper`    | The `devicemapper` storage driver requires `direct-lvm` for production environments, because `loopback-lvm`, while zero-configuration, has very poor performance. `devicemapper` was the recommended storage driver for CentOS and RHEL, as their kernel version did not support `overlay2`. However, current versions of CentOS and RHEL now have support for `overlay2`, which is now the recommended driver. |
| `overlay`         | The legacy `overlay` driver was used for kernels that did not support the “multiple-lowerdir” feature required for `overlay2` All currently supported Linux distributions now provide support for this, and it is therefore deprecated. |



查看本地的配置:



```
docker info

mulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
host:
  arch: amd64
  buildahVersion: 1.19.8
  cgroupManager: systemd
  cgroupVersion: v1
  conmon:
    package: conmon-2.0.22-3.module+el8.3.1+9857+68fb1526.x86_64
    path: /usr/bin/conmon
    version: 'conmon version 2.0.22, commit: a40e3092dbe499ea1d85ab339caea023b74829b9'
  cpus: 8
  distribution:
    distribution: '"rhel"'
    version: "8.3"
  eventLogger: file
  hostname: qxu
  idMappings:
    gidmap: null
    uidmap: null
  kernel: 4.18.0-240.22.1.el8_3.x86_64
  linkmode: dynamic
  memFree: 2282749952
  memTotal: 33259089920
  ociRuntime:
    name: runc
    package: runc-1.0.0-70.rc92.module+el8.3.1+9857+68fb1526.x86_64
    path: /usr/bin/runc
    version: 'runc version spec: 1.0.2-dev'
  os: linux
  remoteSocket:
    path: /run/podman/podman.sock
  security:
    apparmorEnabled: false
    capabilities: CAP_AUDIT_WRITE,CAP_CHOWN,CAP_DAC_OVERRIDE,CAP_FOWNER,CAP_FSETID,CAP_KILL,CAP_MKNOD,CAP_NET_BIND_SERVICE,CAP_NET_RAW,CAP_SETFCAP,CAP_SETGID,CAP_SETPCAP,CAP_SETUID,CAP_SYS_CHROOT
    rootless: false
    seccompEnabled: true
    selinuxEnabled: false
  slirp4netns:
    executable: ""
    package: ""
    version: ""
  swapFree: 16798183424
  swapTotal: 16798183424
  uptime: 376h 35m 50.81s (Approximately 15.67 days)
registries:
  quay-quay-myquay.apps.cluster-4e0c.4e0c.sandbox1083.opentlc.com:
    Blocked: false
    Insecure: true
    Location: quay-quay-myquay.apps.cluster-4e0c.4e0c.sandbox1083.opentlc.com
    MirrorByDigestOnly: false
    Mirrors: []
    Prefix: quay-quay-myquay.apps.cluster-4e0c.4e0c.sandbox1083.opentlc.com
  search:
  - registry.access.redhat.com
  - registry.redhat.io
  - docker.io
  - quay.io
  - registry.aliyuncs.com
store:
  configFile: /etc/containers/storage.conf
  containerStore:
    number: 52
    paused: 0
    running: 1
    stopped: 51
  graphDriverName: overlay
  graphOptions:
    overlay.mountopt: nodev,metacopy=on
  graphRoot: /var/lib/containers/storage
  graphStatus:
    Backing Filesystem: xfs
    Native Overlay Diff: "false"
    Supports d_type: "true"
    Using metacopy: "true"
  imageStore:
    number: 206
  runRoot: /var/run/containers/storage
  volumePath: /var/lib/containers/storage/volumes
version:
  APIVersion: 3.0.0
  Built: 1623138726
  BuiltTime: Tue Jun  8 15:52:06 2021
  GitCommit: ""
  GoVersion: go1.15.13
  OsArch: linux/amd64
  Version: 3.0.2-dev

```









# Docker的配置文件  (Version:  20.10.12)



###  /etc/docker/daemon.json,　如不存在,可新建

可参考:　https://docs.docker.com/engine/reference/commandline/dockerd/

配置driver,如下,修改配置需要重启 systemctl restart docker

```
{    "storage-driver": "overlay2"}
```

默认定义的存储位置:　/var/lib/containerd/

```
  "data-root": "",

```



```
"registry-mirrors":　[]
```

容器的日志存储:

```
/var/lib/docker/containers/<container_id>/<container_id>-json.log

如:　/var/lib/docker/containers/9e26a8b19644a2f1b1e7617906bb8cf99a18f0b340d86fe029296d5db44d21b7
9e26a8b19644a2f1b1e7617906bb8cf99a18f0b340d86fe029296d5db44d21b7 可通过docker ps获得

```

查看本地配置

```
docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.9.1-docker)
  scan: Docker Scan (Docker Inc., v0.21.0)

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 42
 Server Version: 20.10.21
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runtime.v1.linux runc io.containerd.runc.v2
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 1c90a442489720eec95342e1789ee8a5e1b9536f
 runc version: v1.1.4-0-g5fd4c4d
 init version: de40ad0
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 4.18.0-408.el8.x86_64
 Operating System: CentOS Stream 8
 OSType: linux
 Architecture: x86_64
 CPUs: 16
 Total Memory: 15.45GiB
 Name: trainee.taikang1.local
 ID: BQ3I:LS65:J4KH:HULG:WPHB:FA5X:2SLM:7XQ7:FWEL:VTTR:7GSR:D6CV
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  hub.taikang1.local
  127.0.0.0/8
 Registry Mirrors:
  https://gfmnzvu1.mirror.aliyuncs.com/
  https://docker.mirrors.ustc.edu.cn/
  http://hub-mirror.c.163.com/
 Live Restore Enabled: false

```



Docker　daemon的日志,不同的系统保存的位置不不一样,

在rhel 下可　grep docker messages*　,　message文件按天分片

　以下是对照表:　https://stackoverflow.com/questions/30969435/where-is-the-docker-daemon-log

```
Ubuntu (old using upstart ) - /var/log/upstart/docker.log
Ubuntu (new using systemd ) - sudo journalctl -fu docker.service
Amazon Linux AMI - /var/log/docker
Boot2Docker - /var/log/docker.log
Debian GNU/Linux - /var/log/daemon.log
CentOS - cat /var/log/message | grep docker
CoreOS - journalctl -u docker.service
Fedora - journalctl -u docker.service
Red Hat Enterprise Linux Server - /var/log/messages | grep docker
OpenSuSE - journalctl -u docker.service
macOS - ~/Library/Containers/com.docker.docker/Data/log/vm/d‌​ocker.log
Windows - Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time, as mentioned here.
```



Docker的systemctl 配置文件位置

/usr/lib/systemd/system/docker.service



添加国内仓库

```
[root@trainee docker]# cat daemon.json 
{
"registry-mirrors": [
"https://gfmnzvu1.mirror.aliyuncs.com",
"https://docker.mirrors.ustc.edu.cn",
"http://hub-mirror.c.163.com"
]
}

```





# 其他配置文件



### 自签registry的访问方式可通过以下两个方式实现:

[root@trainee docker]# docker push hub.taikang1.local/log-collection
Using default tag: latest
The push refers to repository [hub.taikang1.local/log-collection]
Get "https://hub.taikang1.local/v2/": x509: certificate signed by unknown authority
[root@trainee docker]# vi /etc/docker/daemon.json 





1. cp ca.crt到以下目录,　update-ca-trust
 /etc/pki/ca-trust/source/anchors/

可参考:　https://docs.docker.com/registry/insecure/

如果不能解决,建议使用下面的方式



2.在/etc//docker/cert.d目录下建立对应的目录,放对应的证书,并重起docker service,测试　by mike

```
# pwd
/etc/docker/certs.d

# ls
registry.crc.test:5000
[root@haproxy-01 /etc/docker/certs.d] ens33 = 192.168.146.90
# ls -r *
ca.crt


```

可参考:

https://docs.docker.com/engine/security/certificates/



或者,加入到"insecure-registries"配置中,

```
[root@trainee docker]# cat /etc/docker/daemon.json 
{
"registry-mirrors": [
"https://gfmnzvu1.mirror.aliyuncs.com",
"https://docker.mirrors.ustc.edu.cn",
"http://hub-mirror.c.163.com"
],

"insecure-registries": ["hub.taikang1.local"]
}

```



### ~/.docker/config.json

保存远程仓库的用登录信息

```
{
	"auths": {
		"hub.taikang1.local": {
			"auth": "YWRtaW46cmVkaGF0"
		}
	}
}


docker login后产生
可使用base64 -d解码口令
```



```
 docker pull hub.taikang1.local/log-collection
Using default tag: latest
Error response from daemon: error parsing HTTP 404 response body: invalid character '<' looking for beginning of value: "<!DOCTYPE HTML PUBLIC \"-//IETF//DTD HTML 2.0//EN\">\n<html><head>\n<title>404 Not Found</title>\n</head><body>\n<h1>Not Found</h1>\n<p>The requested URL was not found on this server.</p>\n</body></html>\n"
[user2@trainee ~]$ docker login -u admin -p redhat hub.taikang1.local/log-collection
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /home/user2/.docker/config.json.

```



