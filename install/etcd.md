install etcdctl

https://github.com/etcd-io/etcd/releases/tag/v3.5.5

```
ETCD_VER=v3.5.5

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}



curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

tar xfz etcd-v3.5.5-linux-amd64.tar.gz
```



查看etcd member list



```
alias etcdctl='/tmp/etcd-v3.5.5-linux-amd64/etcdctl --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt  --cacert=/etc/kubernetes/pki/etcd/ca.crt'

[root@master01 etcd]# etcdctl member list -w table
+------------------+---------+----------+-----------------------------+-----------------------------+------------+
|        ID        | STATUS  |   NAME   |         PEER ADDRS          |        CLIENT ADDRS         | IS LEARNER |
+------------------+---------+----------+-----------------------------+-----------------------------+------------+
| 84b9c16176b85993 | started | master01 | https://192.168.146.51:2380 | https://192.168.146.51:2379 |      false |
| ad98b7d566eaef3f | started | master03 | https://192.168.146.53:2380 | https://192.168.146.53:2379 |      false |
| c9e9a2be14b29135 | started | master02 | https://192.168.146.52:2380 | https://192.168.146.52:2379 |      false |
+------------------+---------+----------+-----------------------------+-----------------------------+------------+



[root@master01 etcd]#  etcdctl endpoint status --endpoints https://192.168.146.51:2379,https://192.168.146.52:2379,https://192.168.146.53:2379 -w table
+-----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|          ENDPOINT           |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+-----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://192.168.146.51:2379 | 84b9c16176b85993 |   3.5.3 |  3.8 MB |      true |      false |         3 |       5840 |               5840 |        |
| https://192.168.146.52:2379 | c9e9a2be14b29135 |   3.5.3 |  3.8 MB |     false |      false |         3 |       5840 |               5840 |        |
| https://192.168.146.53:2379 | ad98b7d566eaef3f |   3.5.3 |  3.8 MB |     false |      false |         3 |       5840 |               5840 |        |
+-----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
[root@master01 etcd]# 


```

