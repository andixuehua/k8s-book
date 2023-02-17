

Jenkins中配置

1. master节点加上maven label

3. 添加两个工具,不要自动安装,选择本地目录的安装目录

jdk1.8

maven3.5



4. 配置sharelib in jenkins

Global Pipeline Libraries

![](/home/qxu/Documents/k8sbook-new/jenkins/sharedlib.png)





##### 需要将tools.groovy中的podman 替换为docker命令,如果安装的是docker ,然后commit

５. 增加任务

创建一个pipeline,进行构建

 pipeline-sharedlib-jar,构建镜,**只演示这一个就可以,**,

```
library "local-sharedlib"

def map=[:]
map.put("GIT_URL","https://gitee.com/xuqw123/log-collection")
map.put("TARGET_FILE","target/log-collection-demo-0.0.1-SNAPSHOT.jar")
map.put("PACKAGE_NAME","log-collection.jar")
map.put("PROJECT_NAME","log-collection")
map.put("IMAGE_NAME","registry.crc.test:5000/test/log-collection")

def node="maven"
build(node,map)
```



只构建:

```
library "local-sharedlib"

def map=[:]
map.put("GIT_URL","https://gitee.com/xuqw123/log-collection")
map.put("TARGET_FILE","target/log-collection-demo-0.0.1-SNAPSHOT.jar")
map.put("PACKAGE_NAME","log-collection.jar")
map.put("PROJECT_NAME","log-collection")
map.put("IMAGE_NAME","registry.crc.test:5000/test/log-collection")

def node="maven"
build_only(node,map)
```



构建并修改argocd

```
//library "SharedLibrary"
library "local-sharedlib"


def map=[:]
map.put("GIT_URL","https://gitee.com/xuqw123/log-collection")
map.put("TARGET_FILE","target/log-collection-demo-0.0.1-SNAPSHOT.jar")
map.put("PACKAGE_NAME","log-collection.jar")
map.put("PROJECT_NAME","log-collection")
map.put("IMAGE_NAME","quay.io/qxu/log-collection")
map.put("CD_REPO","git@github.com:mikerain/argocodemo.git")
map.put("CD_BRANCH","main")
map.put("CD_PATH","log-collection")

// map.put("BUILD_EXEC","")

def node="master"

build_cd(node,map)
```









如果有以下错误,

You can allow local checkouts anyway by setting the system property 'hudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT' to true

hudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT  true

使用下面的命令启动jenkins

```
java -Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true -jar jenkins.wa
```

