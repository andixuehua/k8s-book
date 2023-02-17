https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/security-context/

# 为 Pod 或容器配置安全上下文

安全上下文（Security Context）定义 Pod 或 Container 的特权与访问控制设置。 安全上下文包括但不限于：

- 自主访问控制（Discretionary Access Control）： 基于[用户 ID（UID）和组 ID（GID）](https://wiki.archlinux.org/index.php/users_and_groups) 来判定对对象（例如文件）的访问权限。
- [安全性增强的 Linux（SELinux）](https://zh.wikipedia.org/wiki/安全增强式Linux)： 为对象赋予安全性标签。
- 以特权模式或者非特权模式运行。
- [Linux 权能](https://linux-audit.com/linux-capabilities-hardening-linux-binaries-by-removing-setuid/): 为进程赋予 root 用户的部分特权而非全部特权。

- [AppArmor](https://kubernetes.io/zh-cn/docs/tutorials/security/apparmor/)：使用程序配置来限制个别程序的权能。
- [Seccomp](https://kubernetes.io/zh-cn/docs/tutorials/security/seccomp/)：过滤进程的系统调用。
- `allowPrivilegeEscalation`：控制进程是否可以获得超出其父进程的特权。 此布尔值直接控制是否为容器进程设置 [`no_new_privs`](https://www.kernel.org/doc/Documentation/prctl/no_new_privs.txt)标志。 当容器满足一下条件之一时，`allowPrivilegeEscalation` 总是为 true：
  - 以特权模式运行，或者
  - 具有 `CAP_SYS_ADMIN` 权能
- readOnlyRootFilesystem：以只读方式加载容器的根文件系统。

以上条目不是安全上下文设置的完整列表 -- 请参阅 [SecurityContext](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#securitycontext-v1-core) 了解其完整列表。





defualt pod run process by root uid(0),gid(0)

检查一个普通pod

```
# kubectl exec -it log-collection-6c454dc7f8-dl4t8 /bin/bash -n test
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@log-collection-6c454dc7f8-dl4t8:/# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 08:44 ?        00:00:00 /bin/sh /init.sh
root          8      1  0 08:44 ?        00:00:38 java -server -Xms256m -Xmx256m -Duser.timezone=Asia/Shanghai -Djava.security.egd=file:/dev/./urandom -jar /usr/local/log-collection-demo.jar
root         38      0  3 10:15 pts/0    00:00:00 /bin/bash
root         45     38  0 10:15 pts/0    00:00:00 ps -ef
root@log-collection-6c454dc7f8-dl4t8:/# id
uid=0(root) gid=0(root) groups=0(root)

```





建立一个带scc的pod

```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```

检查

```
# kubectl exec -it security-context-demo /bin/sh -n test
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ $ ls
bin   data  dev   etc   home  proc  root  sys   tmp   usr   var
/ $ id
uid=1000 gid=3000 groups=2000
```



典型使用scc的pod,如监控的exporter pod

```
 kubectl get pod pm-prometheus-node-exporter-59426 -o yaml -n monitoring|grep " securityContext" -A4
  securityContext:
    runAsNonRoot: true
    runAsUser: 65534


# kubectl exec -it node-exporter-dhtrw -n monitoring /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
Defaulting container name to node-exporter.
Use 'kubectl describe pod/node-exporter-dhtrw -n monitoring' to see all of the containers in this pod.
/ $ id
uid=65534(nobody) gid=65534(nogroup)

```

