集成jenkins与k8s之前计划的事情：

#### 1,policy k8s与jenkins的角色定位

jenkins master是外部部署，还是k8s内部部署，初期 建议外部部署

jenkins slave是外部部署，还是k8s动态建立 ，初期 建议外部部署，方便调试，安装软件。也可部分部署在k8s内



#### 2. jenkins Job的功能定位

只有CI过程，即，构建，生成image, push到registry就结束

还是，CICD全过程，通过命令或API进行应用在k8s上的部署



#### 3. jenkins 调用k8s的方式定义

通过命令行，kubectl还是插件的API



#### 4 job的实现方式

是传统的shell方式，还是pipeline方式





### 可能实现的步骤：

1 Run CI Job  in  虚机Jenkins instance

2 Run CI Job  in  Pod Jenkins instance

3 Run CD Job in  虚机Jenkins instance

 4 Run CD Job  in  Pod Jenkins instance





### 规划几个工具的关系：

Git

Jenkins

Nexus

Harbor

Kubernetes



#### 规划构建过程（分支）与部署环境的关系

原因，配置是独立于构建的，还是和构建过程相关的？

是否使用了appollo类似的工具，也不是说测试环境的构建包，镜像是否可直接用于生产环境？



#### 规划版本的命名，管理镜版本，考虑升级与回退的策略







1demo一个完整的CICD过程in 虚机（）

2讲解pipeline不同的实现方式，内联, jenkinsfile, sharedlib的实现

