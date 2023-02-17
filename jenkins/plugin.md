在jenkins上安装 kubernetes （集成参考 https://www.youtube.com/watch?v=DAe2Md9sGNA）

 kubernetes continus deploy插件, （集成参考https://www.youtube.com/watch?v=IluhOk86prA）not work, will post error during deploy in some case

Kubernetes CLI  ,

安装jdk, maven,git插件

创建credential访问 gitlab， 访问 harbor, 访问k8s( kubernetes continus deploy)

配置maven的setting.xml访问 nexus



jenkins master:

```
#install jdk
yum install java (not work for mvn build it's jre not JDK)
download jdk 8 and extract , add path and setup JAVA_HOME in bash
https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html


#down latest lts jenkins.war
https://www.jenkins.io/download/


#down maven: https://archive.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
and add mvn in to path

java -jar jenkins.war


#install recommended plugin form jenkins first start



```





配置kubernetes continous deploy plugin (not work for some case wil post error:)

```
#1 add credential
credentials->system->new credentials
kind->kubernetes configuration(kubeconfig)

enter directly(get the conent from you ~/.kube/config file)

in free style:
add a step: kubernetes deployment,
use the pre-defined yaml file to deploy
```



建议，少使用plugin,

因为经常要升级，迁移jenkins 版本时会成为负担。

尽量使用命令行，一个脚本中包括所有内容
