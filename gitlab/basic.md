

# GitLabCI-CD流水线语法

[![一个人](https://picx.zhimg.com/v2-b92f3ca4263ed4e071f9f10b27d34b8a_xs.jpg?source=172ae18b)



![img](https://pic3.zhimg.com/80/v2-67d0709769868b1fbe1d7bec9468301a_720w.webp



### 目录



### 环境

```text
gitlab-ce:14.9.3-ce.0
```

### 1、GitLabCI Pipeline



![img](https://pic4.zhimg.com/80/v2-a6290ed25f350434b2ade3abf58bf21f_720w.webp)



### 1.Pipeline

在每个项目中，使用名为

![img](https://pic1.zhimg.com/80/v2-2762448bee32f4d5328d3c6c84764234_720w.webp)

的**YAML**文件配置GitLab CI/CD

![img](https://pic3.zhimg.com/80/v2-2d626679a77237ad1c4d7ad440d14746_720w.webp)

流水线。



![img](https://pic2.zhimg.com/80/v2-b6a7d09297eea0612adf01a6d7f1b5f5_720w.webp)



### 2.Stages

一条流水线可以包含若干个阶段， 一个阶段可以包含若干个作业。



![img](https://pic4.zhimg.com/80/v2-4bfac9f609ad3965a04ed63296e92097_720w.webp)



### 3.Job

作业是具体要执行的任务，命令脚本语句的集合；



![img](https://pic2.zhimg.com/80/v2-55094b46716d870aec01f05068c9a28d_720w.webp)



### 4.Runner

Runner是每个作业的执行节点 ；每个作业可以根据标签选择不同的构建节点；



![img](https://pic4.zhimg.com/80/v2-e57fb1ba3a71f4014683a89d3ca8b07b_720w.webp)



### 2、Pipeline开发工具

### 1.可视化编辑器

变更.gitlab-ci.yml文件后， 可以通过

```text
Visualize
```

对CI文件中的定义进行可视化；





![img](https://pic3.zhimg.com/80/v2-daeee2b976b01951fd3974eeebfbcaa6_720w.webp)



### 2.语法检测校验

通过Lint可以检测当前CI文件是否存在语法错误；若存在语法错误可以根据提示进行修正；



![img](https://pic1.zhimg.com/80/v2-1392b90a24fb880b7c56ed62a7e50f00_720w.webp)



### 3.作业运行日志

一条流水线包含很多个作业，每个作业的运行日志可以在

```text
Jobs
```

界面看到。





![img](https://pic3.zhimg.com/80/v2-212d7d46646d0eb1e2f4aebcf1e1273e_720w.webp)



### 3、Pipeline核心语法

### 1.stages 阶段控制

源文档

- .pre阶段的作业总是在流水线**开始时**执行；
- .post阶段的作业总是在流水线**结束时**执行；

```text
stages:
  - build
  - test
  - deploy

ciinit:
  tags:
    - build
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - build
  stage: .post
  script:
    - echo "Pipeline end  job"
```



![img](https://pic1.zhimg.com/80/v2-ff560bd02def12fcc46d3268c7791170_720w.webp)



如果两个或者多个作业，指向同一个阶段名称，则该阶段下的所有作业都并行运行；如果不能并行运行，需要检查runner的配置文件中的

```text
concurrent
```

值， 要大于1。



自己测试过程

- stages里可以定义各个stage执行的前后顺序，但是.pre和.post
  2个阶段不受其控制：



![img](https://pic2.zhimg.com/80/v2-f5fc0cebdde4eba1d86c47d90e1ef6c9_720w.webp)



- .pre和.post
  2个阶段测试：

编写如下代码，并提交运行：

```text
stages: #对stages的编排
  - build
  - test
  - deploy

ciinit:
  tags:
    - build
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - build
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - maven
  stage: build
  script:
    - echo "Do your build here"

test:
  tags:
    - maven  
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"

deploy:
  tags:
    - maven  
  stage: deploy
  script:
    - echo "Do your deploy here"
```

预览：



![img](https://pic4.zhimg.com/80/v2-d85681dd011d12e361c84108805582d3_720w.webp)





![img](https://pic4.zhimg.com/80/v2-d85681dd011d12e361c84108805582d3_720w.webp)



提交运行：



![img](https://pic1.zhimg.com/80/v2-1c50aaa9820c20303ecdfa8add78e868_720w.webp)



可以看到，符合预期，

```text
.pre
```

先运行，

```text
.post
```

最后运行：





![img](https://pic4.zhimg.com/80/v2-53d90a5f1b85b0c11698f6fe4fc1e407_720w.webp)



测试结束。

实践：

```text
gitlab-ruuner上作业并行设置
```

-2022.5.6



> 如果两个或者多个作业，指向同一个阶段名称，则该阶段下的所有作业都并行运行；如果不能并行运行，需要检查runner的配置文件中的
> concurrent
> 值， 要大于1。

- 此时，模拟有2个job指向同一个test阶段

```text
stages: #对stages的编排
  - build
  - test
  - deploy

ciinit:
  tags:
    - build
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - build
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - maven
  stage: build
  script:
    - echo "Do your build here"

test:
  tags:
    - maven  
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 10
test1:
  tags:
    - maven  
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 10



deploy:
  tags:
    - maven  
  stage: deploy
  script:
    - echo "Do your deploy here"
```



![img](https://pic3.zhimg.com/80/v2-6e41da646bed9f2accc336c2f566c7fa_720w.webp)





![img](https://pic2.zhimg.com/80/v2-72172f3f688674672e25fdfff193082d_720w.webp)



提交并观察运行状态：



![img](https://pic2.zhimg.com/80/v2-0365f7958d5fb1c9cf662bb1c92a9da9_720w.webp)



可以看到，此时Test阶段的2个作业不是并行运行的，这是为什么呢？

- 我们来看下gitlab-runner上的关于concurrent(并行)
  配置

```text
[root@gitlab-runner ~]#cat /etc/gitlab-runner/config.toml #这里可以看到当前的
```



![img](https://pic2.zhimg.com/80/v2-4bc201c772710233abef83faf5e54ff5_720w.webp)



```text
[root@gitlab-runner ~]#vim /etc/gitlab-runner/config.toml
concurrent = 10 
#修改完保存退出就好，自动生效
```

- 再次运行刚才那条pipeline，观察效果



![img](https://pic1.zhimg.com/80/v2-59c6d167e673268f6ef659c3e1072788_720w.webp)



此时就可以看到Test阶段2个作业同时运行了，符合预期！

### 2.variables 环境变量

变量可以分为全局变量和局部变量；全局变量是整个流水线可以用的，局部变量是仅在作业中生效的；

```text
variables:
  DEPLOY_ENV: "dev"

deploy_job:
  stage: deploy
  tags:
    - maven
  variables:
    DEPLOY_ENV: "test"
  script:
     - echo  ${DEPLOY_ENV}
```

实践：

```text
局部变量优先级高于全局变量
```

-0222.5.6



- 编写测试代码(完整代码)



![img](https://pic1.zhimg.com/80/v2-8fc42f05c574bb4c6e17840062539c14_720w.webp)



```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"

deploy_job:
  stage: deploy
  tags:
    - maven
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - build
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - build
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - maven
  stage: build
  script:
    - echo "Do your build here"

test:
  tags:
    - maven  
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 10
test1:
  tags:
    - maven  
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 10



deploy:
  tags:
    - maven  
  stage: deploy
  script:
    - echo "Do your deploy here"
```

- 提交并观察效果



![img](https://pic2.zhimg.com/80/v2-f977a4367dc909a81bffbaf4bdcad9b9_720w.webp)



看下

```text
deploy_job
```

的运行日志：





![img](https://pic2.zhimg.com/80/v2-40ff1ccfd380f31e59a38784bf3ab731_720w.webp)



可以看到输出为test，验证局部变量优先级更改。

测试结束。

### 3.job 作业默认配置

定义一个作业的时候，一般定义哪些关键字呢？ 作业在哪个runner运行？ 作业属于流水线中的哪个阶段？ 这个作业要做什么？

```text
variables:
  BUILD_RUNNER: k8s

## job名称
cibuild:
  tags:   
    - build
    - ${BUILD_RUNNER} #注意：在14版本以前，tags这里是不支持使用环境变量的！
    - devops
  stage: build
  before_script: #像这个before_script既可以在pipeline里定义，也可以在job里定义！
    - echo "job before_script......."
  script:
    - echo "job script....."
  after_script:
    - echo "job after_script......."
```

参数解析：

| 语法关键字    | 作用                             | 备注                                                         |
| ------------- | -------------------------------- | ------------------------------------------------------------ |
| variables     | 定义作业中的环境变量；           |                                                              |
| tags          | 根据标签选择运行作业的构建节点； | 多个标签， 则匹配具有所有标签的构建节点；GitLab14.1版本后， 标签的值可以使用变量；GitLab14.3版本后， 标签数量必须小于50； |
| stage         | 指定当前作业所属的阶段名称；     |                                                              |
| before_script | 作业在运行前执行的Shell命令行；  |                                                              |
| script        | 作业在运行中执行的Shell命令行；  | 每个作业至少要包含一个script；                               |
| after_script  | 作业在运行后执行的Shell命令行；  |                                                              |

实践：

```text
将全部作业里的tags内容用变量替换 & before_script和after_script 测试
```



- 先用环境变量替换掉每个作业里的runner tags：

```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
```

提交运行：



![img](https://pic3.zhimg.com/80/v2-9151b863cbfb3770e444ef4c61534ec2_720w.webp)



需要注意：如果

```text
before_script和after_script
```

定义在pipeline里，则每个作业里都会运行这2个脚本；如果是定义在某个job里，则只会在其job里运行！





![img](https://pic1.zhimg.com/80/v2-0c54c2d9432c12fe086731614bb1adec_720w.webp)





![img](https://pic2.zhimg.com/80/v2-17bcf6bf34e85ea46e7b4d5fe5343e69_720w.webp)



测试结束。

### 4.job 作业运行控制

| 语法关键字    | 作用                                                         | 备注                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| allow_failure | 控制作业状态，是否允许作业失败，默认值为false 。启用后，如果作业运行失败，该作业将在用户界面中显示橙色警告。 | 管道将认为作业成功/通过，不会被阻塞。 假设所有其他作业均成功，则该作业的阶段及其管道将显示相同的橙色警告。但是，关联的提交将被标记为"通过"，而不会发出警告。 |
| when          | 根据状态控制作业运行, 当前面作业成功或者失败时运行。         | on_success 前面阶段成功时执行(默认值);on_failure 前面阶段失败时执行；always 总是执行；manual 手动执行；delayed 延迟执行；start_in'5'5 seconds30 minutes1 day1 weeknever 永不执行； |
| retry         | 作业重新运行，遇到错误重新运行的次数。                       | 值为整数等于或大于0，但小于或等于2异常分类 (一般就是3次)     |
| timeout       | 作业运行超时时间；                                           |                                                              |
| rules         | 根据特定的变量或文件变更来控制作业运行；                     | ifchangesexists                                              |
| needs         | 作业依赖控制；                                               | needs: ["作业名称"]                                          |
| parallel      | 生成多个作业，并行运行                                       | parallel:5值 2-50之间                                        |

### 1、allow_failure 允许作业失败

源文档

```text
cibuild:
  tags:
    - build
  stage: build
  script:
    - echo1 "job script....."    ## 报错：命令找不到
```



![img](https://pic3.zhimg.com/80/v2-f30a5815f441f9c0be7f624a710e8092_720w.webp)



添加后：

```text
cibuild:
  tags:
    - build
  stage: build
  script:
    - echo1 "job script....."
  allow_failure: true
```



![img](https://pic1.zhimg.com/80/v2-c5c20c134a67059983ef0c33f17d8734_720w.webp)



测试：

```text
allow_failure
```

-2022.5.6



- 编写代码



![img](https://pic2.zhimg.com/80/v2-0d712f3c3cfccb7a397e04f32d9f0799_720w.webp)



```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - ech
    - sleep 3
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
```

- 提交运行，观察效果：



![img](https://pic3.zhimg.com/80/v2-81b3ed56aa8f44b8b2489deb4590dae2_720w.webp)





![img](https://pic2.zhimg.com/80/v2-0d7cf46a1e51fd76cbfb573b37467ddd_720w.webp)





![img](https://pic3.zhimg.com/80/v2-bca6be33becc0802c2f17dd499be5a4a_720w.webp)



符合预期效果，测试结束。

### 2、when 状态控制和运行方式

源文档

- 根据上游作业的状态决定

- - 当前作业是否运行？

- 运行的方式？（手动/自动/定时）

```text
cibuild:
  tags:
    - build
  stage: build
  before_script:
    - echo1 "job before_script......."
  script:
    - echo "job script....."
  after_script:
    - echo "job after_script......."
  allow_failure: false

citest1:
  tags:
    - build
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
  when: on_success   #只有上面作业成功后才会运行。
```

测试

```text
when的手动执行
```

-20222.5.6



- 编写代码

```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - ech
    - sleep 3
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

- 提交并验证

可以看到这里需要手动执行操作：(审批环节)



![img](https://pic4.zhimg.com/80/v2-4cd8d3f2a24a6146a27987df26d24f03_720w.webp)





![img](https://pic1.zhimg.com/80/v2-39b9e0efd8a0b959d74dadb96ad50338_720w.webp)



测试结束。

测试

```text
when 延迟执行
```

-2022.5.6



- 编写代码：



![img](https://pic3.zhimg.com/80/v2-b0c139d596030a4d1a51c100a979797e_720w.webp)



完整代码如下：

```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - ech
    - sleep 3
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
  when: delayed
  start_in: '5'



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

- 提交并验证：



![img](https://pic2.zhimg.com/80/v2-aea0d814cc8ab274f7b28a6fedb136e1_720w.webp)



测试结束。

### 3、timeout作业运行超时时间

```text
build:
  script: build.sh
  timeout: 3 hours 30 minutes

test:
  script: rspec
  timeout: 3h 30m
```

### 4、retry 作业失败后重试次数



![img](https://pic2.zhimg.com/80/v2-dcc78d9474302fcc6f80f4f7e786b4b5_720w.webp)



```text
cibuild:
  tags:
    - build
  stage: build
  retry: 2
  before_script:
    - echo1 "job before_script......."
  script:
    - echo "job script....."
  after_script:
    - echo "job after_script......."
  allow_failure: false
```

根据特定的错误匹配：

```text
always ：在发生任何故障时重试（默认）。
unknown_failure ：当失败原因未知时。
script_failure ：脚本失败时重试。
api_failure ：API失败重试。
stuck_or_timeout_failure ：作业卡住或超时时。
runner_system_failure ：构建节点的系统发生故障。
missing_dependency_failure: 依赖丢失。
runner_unsupported ：Runner不受支持。
stale_schedule ：无法执行延迟的作业。
job_execution_timeout：作业运行超时。
archived_failure ：作业已存档且无法运行。
unmet_prerequisites ：作业未能完成先决条件任务。
scheduler_failure ：调度失败。
data_integrity_failure ：结构完整性问题。


####
max ：最大重试次数  when ：重试失败的错误类型

cibuild:
  tags:
    - build
  stage: build
  retry:
    max: 2
    when:
      - script_failure
  before_script:
    - echo1 "job before_script......."
  script:
    - echo "job script....."
  after_script:
    - echo "job after_script......."
  allow_failure: false
```

### 5、needs 作业关联运行

源文档

- needs 指定要依赖的作业名称

```text
cibuild:
  tags:
    - build
  stage: build
  script:
    - echo "mvn clean package"
    - sleep 30 

citest1:
  tags:
    - build
  stage: test
  needs: ["cibuild"]
  script:
    - echo "mvn test"
```



![img](https://pic4.zhimg.com/80/v2-7a5abb1a15c34349e7c089d27747b7bf_720w.webp)



测试

```text
needs
```

-2022.5.6



- 编写代码：



![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='637' height='472'></svg>)



完整代码如下：

```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - ech
    - sleep 10
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
  when: delayed
  start_in: '5'
  needs: #注意：当needs和上面延迟在一起时，needs优先级更高！
    - test



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

- 提交并验证：



![img](https://pic4.zhimg.com/80/v2-59402a5fb17d25e17c71c5f61ddf49ef_720w.webp)



测试结束。

### 6、parallel 并行运行

源文档

```text
demojobs:
  stage: test
  parallel: 3
  script: mvn test
```



![img](https://pic3.zhimg.com/80/v2-9790dbdb78ff3a3067c83a75b8a4de82_720w.webp)



测试

```text
parallel
```



- 编写代码：



![img](https://pic4.zhimg.com/80/v2-a3c32d2f312e2ff6c7d8b1794a2b5983_720w.webp)



```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
  parallel: 5

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - ech
    - sleep 10
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
  when: delayed
  start_in: '5'
  needs:
    - test



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

- 提交并验证：



![img](https://pic2.zhimg.com/80/v2-3842115e0c5d50ae3bf291e425f52971_720w.webp)



测试结束。

### 7、rules根据变量/文件控制

源文档

citest1 config key may not be used with

```text
rules
```

: when.



根据条件（变量）判断： IF



![img](https://pic4.zhimg.com/80/v2-dbc6047ea0ab051a4daedaa09a94fb4b_720w.webp)



```text
variables:
  DOMAIN: example.com

codescan:
  stage: build
  tags:
    - build
  rules:
    - if: '$DOMAIN == "example.com"'
      when: manual
    - when: on_success
  script:
    - echo "codescan"
    - sleep 5;
  #parallel: 5
```

根据文件判断:

```text
changes
exists
```





![img](https://pic1.zhimg.com/80/v2-85fb997d518b30d506f52bd6da1719e0_720w.webp)





![img](https://pic4.zhimg.com/80/v2-9c0ea139a1319d10f40c6c32e5d1fffb_720w.webp)





![img](https://pic1.zhimg.com/80/v2-52d8571fce7b292c96dd7923781e3460_720w.webp)



```text
#exists
citest1:
  tags:
    - build
  stage: test
  rules:
    - exists:
        - Dockerfile
      when: manual
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"

#variables
variables:
  ENV_TYPE: "dev"

cddeploy:
  tags:
    - build
  stage: deploy
  rules:
    - if: $CI_COMMIT_REF_NAME == "master"
      variables:
        ENV_TYPE: "prod"
  script:
    - echo "Deploy env ${ENV_TYPE}"  
```

测试

```text
根据条件（变量）判断
```



- 编写代码:



![img](https://pic1.zhimg.com/80/v2-1110afa3c2b294a7058f3407ce53ec28_720w.webp)



```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  rules:
    - if: '$DEPLOY_ENV == "dev"'
      when: manual
    - when: on_success
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV2: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
  parallel: 5

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "love you"
    - sleep 10
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
  when: delayed
  start_in: '5'
  needs:
    - test



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

- 提交并验证：



![img](https://pic1.zhimg.com/80/v2-e27dde3680299c438e22925f3ee18330_720w.webp)



测试结束。

测试

```text
根据文件判断: changes
```



- 先创建一个dockerfile文件并提交：



![img](https://pic4.zhimg.com/80/v2-a544ea6d61238085eda4cfa20d05ab3b_720w.webp)



- 编写代码：



![img](https://pic4.zhimg.com/80/v2-cb2c4920ab1f6e0fb379db8430ee2037_720w.webp)



```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  rules:
    - if: '$DEPLOY_ENV == "dev"'
      when: manual
    - when: on_success
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV2: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
  parallel: 5
  rules:
    - changes:
      - Dockerfile
      when: manual

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "love you"
    - sleep 10
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
  when: delayed
  start_in: '5'
  needs:
    - test



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

提交，然后此时再修改下刚才那个Dockerfile文件，并提交，观察下效果：

- 这里出错了：。。。



![img](https://pic3.zhimg.com/80/v2-1b4beceb842e960e65328807f00823be_720w.webp)





![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1919' height='710'></svg>)





![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1660' height='699'></svg>)





![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1404' height='837'></svg>)





![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1695' height='836'></svg>)



应该是和gitlab-runner上的git版本有关系了：。。。



![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1113' height='204'></svg>)



源码升级gi到2版本以上就可以了：



![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1920' height='1021'></svg>)



[https://blog.csdn.net/weixin_39246554/article/details/124628706?spm=1001.2014.3001.5502](https://link.zhihu.com/?target=https%3A//blog.csdn.net/weixin_39246554/article/details/124628706%3Fspm%3D1001.2014.3001.5502)



![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1919' height='824'></svg>)





![img](https://pic1.zhimg.com/80/v2-3dc9e0f5001741c10529577b8a9c84c4_720w.webp)





![img](https://pic1.zhimg.com/80/v2-073d4aab7b0f2ba9c5524cb304806520_720w.webp)





![img](https://pic1.zhimg.com/80/v2-969f89fc72a139ccca6b8d2c3e1d0268_720w.webp)



测试



![img](https://pic3.zhimg.com/80/v2-f9b1b7a8a00bbfa3f3c6747afe03e46a_720w.webp)



- 先创建一个demo目录，在底下创建一个Dockerfile并提交



![img](https://pic1.zhimg.com/80/v2-84261241e6fc4f3f4199ab7d9f9f7908_720w.webp)



- 编写代码



![img](https://pic2.zhimg.com/80/v2-89e6af54e0c9c44f4717272b21705005_720w.webp)



```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  rules:
    - if: '$DEPLOY_ENV == "dev"'
      when: manual
    - when: on_success
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV2: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
  parallel: 5
  rules:
    - changes:
      - /demo/**
      when: manual

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "love you"
    - sleep 10
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
  when: delayed
  start_in: '5'
  needs:
    - test



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

- 提交并验证：



![img](https://pic4.zhimg.com/80/v2-a02f3775bd975354856b437e04e439b7_720w.webp)



发现build阶段不见了：。。，这里再次运行下流水线看看



![img](https://pic4.zhimg.com/80/v2-dae8bef908081b151120b339c8e7fdbf_720w.webp)



可以看到此时就有效果了：



![img](https://pic3.zhimg.com/80/v2-4ede401b420d9b22227daf7a01660756_720w.webp)





![img](https://pic1.zhimg.com/80/v2-45d54a9a3e2a3ddd665e64fbf11c668c_720w.webp)



可以看到也是可以实现的。

测试结束。

### 5.Pipeline运行控制

### 1.workflow 控制流水线

源文档

控制管道是否创建和运行。根据条件（变量）判断： IF

- if 定义变量条件;

- variables 重新定义变量的值；

- when

- - always

- never

| if: '$CI_PIPELINE_SOURCE == "merge_request_event"' | 合并请求时运行流水线； |
| -------------------------------------------------- | ---------------------- |
| if: '$CI_PIPELINE_SOURCE == "push"'                | 提交代码运行流水线；   |

```text
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "push"
      when: never
```

自己测试过程

- 编写代码：



![img](https://pic4.zhimg.com/80/v2-ba91912f6feb9e0c911de66a9e7286d3_720w.webp)



```text
stages: #对stages的编排
  - build
  - test
  - deploy

workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "push"
      when: never
    - when: always

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  rules:
    - if: '$DEPLOY_ENV == "dev"'
      when: manual
    - when: on_success
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV2: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
  parallel: 5
  rules:
    - changes:
      - /demo/**
      when: manual

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "love you"
    - sleep 10
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
  when: delayed
  start_in: '5'
  needs:
    - test



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

- 提交并观察效果：



![img](https://pic1.zhimg.com/80/v2-f9c8d8781a75eae354be2a83346885b8_720w.webp)



测试结束。

### 2.Git 选项跳过流水线

- 提交信息中添加关键字 [ci skip]
  或者 [skip ci]
  ；
- Git 2.10 更高版本，可以通过以下配置设置CI/CD；

```text
## 跳过
git push -o ci.skip

## 传递变量
git push -o ci.variable="MAX_RETRIES=10" -o ci.variable="MAX_TIME=600"
```

### 3.trigger 触发下游管道

源文档

- 触发项目管道
- 触发子管道

strategy： 默认情况下，一旦创建了下游管道，trigger作业就会以success状态完成。要强制等待下游管道完成，使用

```text
strategy: depend
```

。



触发项目管道：

```text
triggers:
  stage: deploy
  trigger:
    project: devops/devops-maven-service
    branch: main
    strategy: depend    ## 状态同步
```

触发子管道：

```text
#这种场景适合一个项目里有多个子模块
triggers:
  stage: deploy
  trigger:
    include:
      - local: /ci/java-pipeline.yml
    strategy: depend   ## 状态同步

## 不同项目
triggers:
  stage: deploy
  trigger:
    include:
      - project: 'devops/my-pipeline-lib'
        ref: 'main'
        file: '/cd/java-pipeline.yml'
```

实践：

```text
触发项目管道
```



- 先创建一个Project，里面创建.gitlab-ci.yml
  文件，然后写入之前的简短代码

devops4-app-ui



![img](https://pic3.zhimg.com/80/v2-c591e52c517ba7cadc91e26b06d0a642_720w.webp)





![img](https://pic3.zhimg.com/80/v2-032fa80dbad299ce65075dce8f9fe5fa_720w.webp)



```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
    - sleep 10

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 5

deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
```

提交后，可以看到正在运行了：



![img](https://pic3.zhimg.com/80/v2-809ff97f011a31c3feda02e69f596566_720w.webp)



- 来到devops4/devops4-app-service
  项目的.gitlab-ci.yml
  里编写代码：



![img](https://pic2.zhimg.com/80/v2-d2b04b1fb0dbdbf30481a2941fe0c8a9_720w.webp)



```text
# workflow: #test workflow.test1
#   rules:
#     - if: '$CI_PIPELINE_SOURCE == "push"'
#       when: never
#     - when: always


stages: #对stages的编排
  - build
  - test
  - deploy



variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  rules:
    - if: '$DEPLOY_ENV == "dev"'
      when: manual
    - when: on_success
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV2: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"


build:
  stage: build
  trigger:
    project: devops4/devops4-app-ui
    branch: main
    strategy: depend #状态同步



build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
  parallel: 5
  rules:
    - changes:
      - /demo/**
      when: manual

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "love you"
    - sleep 10
  allow_failure: true

test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
  when: delayed
  start_in: '5'
  needs:
    - test



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

- 提交后，注意观察现象

此时，我们到devops4/devops4-app-ui项目下一条管道在运行了，符合预期效果！

这里不需要进入到这个目录，直接点击这个build阶段就能调到触发项目的位置了：



![img](https://pic4.zhimg.com/80/v2-2290b6aecabc2df65f7d0ae06443eac3_720w.webp)



测试结束。

实践：

```text
仅能通过其他Pipeline触发
```

-2022.5.6





![img](https://pic1.zhimg.com/80/v2-2c9f94638af95e52176b50c0dcc8a6f0_720w.webp)



- 需求：仅能通过其他Pipeline触发
- 先测试下，每次操作的CI_PIPELINE_SOURCE
  属性值是什么

来到

```text
devops4-app-ui
```

工程下，编辑代码如下：





![img](https://pic1.zhimg.com/80/v2-025ac311d9a55b18630f300e14a71484_720w.webp)



```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
    - echo $CI_PIPELINE_SOURCE    
    - sleep 3

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3


deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
```

此时，直接在自己Editor进行提交，并观察现象：



![img](https://pic1.zhimg.com/80/v2-e09a4ffd77c7bd9b6d8ea3c0de0b42a8_720w.webp)



可以看到

```text
$CI_PIPELINE_SOURCE
```

值为

```text
push
```

。



此时，来到

```text
devops4-app-serice
```

工程下，进行提交，并再次观察

```text
devops4-app-ui
```

里变量的值：



```text
# workflow: #test workflow.test1
#   rules:
#     - if: '$CI_PIPELINE_SOURCE == "push"'
#       when: never
#     - when: always


stages: #对stages的编排
  - build
  - test
  - deploy



variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  rules:
    - if: '$DEPLOY_ENV == "dev"'
      when: manual
    - when: on_success
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV2: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"


build:
  stage: build
  trigger:
    project: devops4/devops4-app-ui
    branch: main
    strategy: depend #状态同步



build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
  parallel: 5
  rules:
    - changes:
      - /demo/**
      when: manual

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "love you"
    - sleep 10
  allow_failure: true


test1:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3
  when: delayed
  start_in: '5'
  needs:
    - test




deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
  when: manual
```

我们可以看到是

```text
pipeline
```

：





![img](https://pic2.zhimg.com/80/v2-7ba82886f3e5e7d649fa0ab7812405bd_720w.webp)



- 此时，再来编辑devops4-app-ui
  的代码



![img](https://pic4.zhimg.com/80/v2-f357beb2f3a7388e765d617445a70c87_720w.webp)



```text
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "pipeline"'
      when: always
    - when: never

stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
    - echo $CI_PIPELINE_SOURCE    
    - sleep 3

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
```

- 写完代码后，直接提交，观察效果：



![img](https://pic2.zhimg.com/80/v2-0dcda0473cc9995dc785d1734cf8d991_720w.webp)



发现，提交并不能运行这条流水线。

- 我们再运行一次devops4-app-serice
  流水线，发现就可以正常触发devops4-app-ui
  管道了：



![img](https://pic4.zhimg.com/80/v2-76852f3e9e94e047f599789735dce55b_720w.webp)





![img](https://pic4.zhimg.com/80/v2-0cd2145e180954bee276321b9d4d83b7_720w.webp)



测试结束。

实践：

```text
触发子管道
```

-2022.5.6



- 新建一个项目：devops4-monrepo-service



![img](https://pic2.zhimg.com/80/v2-79e5e69e77dc2366ce8842ece880d3dd_720w.webp)



- 在项目下分别创建如下文件

```text
devops4-monrepo-service工程
    module1/ci.yml
    module2/ci.yml
    .gitlab-ci.yml
```

ci.yml文件内容为简单的一个流水线：

```text
stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
    - sleep 3

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3

deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"
```



```text
.gitlab-ci.yml
```

文件内容如下：



注意下：这里的job下是没有tags的！

```text
stages: #对stages的编排
  - build

build-module1:
  stage: build
  trigger:
    include:
      - local: module1/ci.yml
    strategy: depend

build-module2:
  stage: build
  trigger:
    include:
      - local: module2/ci.yml
    strategy: depend
```

然后提交触发构建，查看效果：

会看到同时触发

```text
build-module1
```

和

```text
build-module2
```

作业的运行，及Downstream的运行：





![img](https://pic2.zhimg.com/80/v2-b906a49354a247212a55e001ec88c621_720w.webp)





![img](https://pic3.zhimg.com/80/v2-a874687f345f21530ba377b4cc11a80a_720w.webp)





![img](https://pic2.zhimg.com/80/v2-5f692590404afbdb2ff267d7d8ddd74d_720w.webp)



- 此时更改.gitlab-ci.yml
  内容，添加rules，如果该工程下的那个module下的文件发生了改变，就只会触发自己的module运行：



![img](https://pic3.zhimg.com/80/v2-67a9e59a71e098ce3a8a30a547fcd2da_720w.webp)



```text
stages: #对stages的编排
  - build

build-module1:
  stage: build
  trigger:
    include:
      - local: module1/ci.yml
    strategy: depend
  rules:
    - changes:
        - module1/*
      when: always

build-module2:
  stage: build
  trigger:
    include:
      - local: module2/ci.yml
    strategy: depend
  rules:
    - changes:
        - module2/*
      when: always  
```

保存提交代码后，发现并不会触发流水线的运行！

- 现在，我们先后修改下module1/ci.yml内容和module2/ci.yml内容，提交并观察，是否只会触发自己模块的流水线？



![img](https://pic1.zhimg.com/80/v2-ff1f55c7f1830eaaddc6b856c5db1bb0_720w.webp)





![img](https://pic2.zhimg.com/80/v2-31ddabd620d139d60dfd76fd30128bad_720w.webp)



符合预期。

测试结束。

### 4、Pipeline环境变量

预定义变量信息：[https://docs.gitlab.com/ee/ci/variables/predefined_variables.html](https://link.zhihu.com/?target=https%3A//docs.gitlab.com/ee/ci/variables/predefined_variables.html)

代码类

- CI_COMMIT_AUTHOR 提交人
- CI_COMMIT_BRANCH 提交分支
- CI_COMMIT_MESSAGE
- CI_COMMIT_REF_NAME
- CI_COMMIT_SHORT_SHA

作业类：

- CI_JOB_ID
- CI_JOB_NAME
- CI_JOB_STAGE
- CI_JOB_URL

流水线类：

- CI_PIPELINE_ID
- CI_PIPELINE_SOURCE
- CI_PIPELINE_TRIGGERED
- CI_PIPELINE_URL

### 5、PipelineTemplates实践

[https://gitlab.com/gitlab-org/gitlab/-/tree/master/lib/gitlab/ci/templates](https://link.zhihu.com/?target=https%3A//gitlab.com/gitlab-org/gitlab/-/tree/master/lib/gitlab/ci/templates)

### 1.extends

xtends可以指定多个模板， 相同的参数最后模板会覆盖前面的。例如： 下面两个模板中，继承关系是：

- 继承 .test1 （此时rspec的变量NAME
  的值为gitlab）
- 继承 .test2 （此时rspec的变量NAME
  的值为gitlabCI , 覆盖了.test1中的值）

```text
.test1: #模板：它不会实际地去运行！
  variables:
    NAME: "gitlab"
  tags:
    - build
  stage: test
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  script: echo "mvn test"

.test2:
  variables:
    NAME: "gitlabCI"
  tags:
    - build01
  stage: test

rspec:
  extends: 
    - .test1
    - .test2
  script: echo " DevOps"




###### 结果

rspec:
  variables:
    NAME: "gitlabCI"
  tags:
    - build
  stage: test
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  script: echo " DevOps"
```

实践：

```text
PipelineTemplate之extends
```

-2022.5.6



- 本次在devops4-app-ui
  工程里测试

在文件最后添加代码如下：



![img](https://pic4.zhimg.com/80/v2-f2e82a47bfe6e2d4ed01456b0b1bceb3_720w.webp)



```text
# workflow:
#   rules:
#     - if: '$CI_PIPELINE_SOURCE == "pipeline"'
#       when: always
#     - when: never

stages: #对stages的编排
  - build
  - test
  - deploy

variables:
  DEPLOY_ENV: "dev"
  RUNNER_TAG: "maven"

deploy_job:
  stage: deploy
  tags:
    - ${RUNNER_TAG}
  variables:
    DEPLOY_ENV: "test"
  script:
    - echo  ${DEPLOY_ENV}

ciinit:
  tags:
    - ${RUNNER_TAG}
  stage: .pre
  script:
    - echo "Pipeline init first job"

ciend:
  tags:
    - ${RUNNER_TAG}
  stage: .post
  script:
    - echo "Pipeline end  job"

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  tags:
    - ${RUNNER_TAG}
  stage: build
  script:
    - echo "Do your build here"
    - echo $CI_PIPELINE_SOURCE    
    - sleep 3

test:
  tags:
    - ${RUNNER_TAG} 
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"
    - sleep 3



deploy:
  tags:
    - ${RUNNER_TAG}
  stage: deploy
  script:
    - echo "Do your deploy here"


.test1:
  variables:
    NAME: "gitlab"
  tags:
    - build01
  stage: test
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  script: echo "mvn test"

.test2:
  variables:
    NAME: "gitlabCI"
  tags:
    - build
  stage: test

rspec:
  extends: 
    - .test1
    - .test2
  script: echo "$NAME"
```

- 提交并观察现象：



![img](https://pic1.zhimg.com/80/v2-ba9b68d86ad23d25af970bd53b12fcb4_720w.webp)





![img](https://pic4.zhimg.com/80/v2-333a957319edc76814ba89d4163aa233_720w.webp)



符合

```text
相同的参数最后模板会覆盖前面
```

概念，测试结束。



### 2.include

include用于在CI/CD 配置中引入外部 YAML 文件。可以将一个长

```text
.gitlab-ci.yml
```

文件分解为多个文件以提高可读性，或减少同一配置在多个地方的重复。



- local 导入当前仓库中的文件；
- file 导入当前项目或其他项目库中的文件；
- remote 导入一个远程的文件（例如：[http://xxx.yaml](https://link.zhihu.com/?target=http%3A//xxx.yaml)）
- template 导入GitLab官方提供的模板文件；

```text
## 本地仓库文件
include:
  - local: '/templates/.gitlab-ci-java.yml'

## 其他仓库文件
include:
  - project: 'devops/my-project'
    ref: main
    file: 
      - '/templates/.gitlab-ci-java.yml'
      - '/templates/.tests.yml'

## 远程文件
include:
  - remote: 'https://192.168.1.200//-/raw/main/.gitlab-ci.yml'
```

**综合实例：**

将模板文件，放到单独的一个仓库中；



![img](https://pic2.zhimg.com/80/v2-a6230924a65a40abc3f3867aff7d7be9_720w.webp)



build.yml

```text
## build模板
.build:
  tags:
    - build
  stage: build
  variables:
    BUILD_TOOLS: "maven"
    SKIP_TEST: "true"
  script: 
    - if [ "${SKIP_TEST}" == "true" ];then echo "mvn clean package -DskipTests";fi 
  rules:
    - if: $CI_PIPELINE_SOURCE == "push"
      when: never

.mavenBuild:
  tags:
    - build
  stage: build
  variables:
    BUILD_SHELL: "mvn clean package"
  script:
    - echo "${BUILD_SHELL}"
```

gitlab-ci.yml

```text
## 引入项目分支中的build.yml
include:
  - project: 'devops03/gitlabci-library-service'
    ref:  "main"
    file:
      - "/build.yml"

stages:
  - build

ciBuild:
  tags:
    - build 
  extends:
    - .mavenBuild

ciTriggerTest:
  tags:
    - build
  stage: build
  script:
    - echo ${BUILD_TOOL}
```

实践：

```text
练习Pipeline模板库(测试成功)
```

-2022.5.6(这个有点难度)



老师文档

1. 创建一个新的项目 “devops4/devops4-devops-service” (模板库);
2. 为项目添加一个CI模板文件（ci/ci.yml）；（参考pipeline.yml）
3. 添加cicd.yml 文件， pipeline， include导入模板库中的ci/ci.yml



![img](https://pic2.zhimg.com/80/v2-aeb4bca3eabdc2e12cac8ca1f1803c61_720w.webp)



ci/ci.yml：

```text
‘devops4/devops4-devops-service   模板库
```



```text
.build:
  variables:
    SHELL: "mvn clean build"
  tags:
    - build
  stage: build
  script: echo "${SHELL}"
```

cicd.yml

```text
## 导入仓库文件
include:
  - project: 'devops4/devops4-devops-service'
    ref: main
    file: 
      - '/ci/ci.yml'
variables:
  BUILD_SHELL: "mvn build"
stages:
  - build

## 继承模板
build:
  extends: 
    - .build
  variables:
    SHELL: $BUILD_SHELL
```

远程CI文件配置:

[http://192.168.1.200/devops4/devops4-devops-service/-/raw/main/cicd.yml](https://link.zhihu.com/?target=http%3A//192.168.1.200/devops4/devops4-devops-service/-/raw/main/cicd.yml)



![img](https://pic3.zhimg.com/80/v2-4d19b4192d2e7d363df8895a50d39c3e_720w.webp)



自己测试过程

- 创建模板库devops4-devops-service



![img](https://pic3.zhimg.com/80/v2-758a566d6e8f183add5a31b2dd37c8b6_720w.webp)



创建如下相关文件：

ci/ci.yml

```text
.build:
  variables:
    SHELL: "mvn clean build"
  tags:
    - build
  stage: build
  script: echo "${SHELL}"
```

cicd.yml

```text
## 导入仓库文件
include:
  - project: 'devops4/devops4-devops-service'
    ref: main
    file: 
      - '/ci/ci.yml'

variables:
  BUILD_SHELL: "mvn build"
stages:
  - build

## 继承模板
build:
  extends: 
    - .build
  variables:
    SHELL: $BUILD_SHELL
```



![img](https://pic3.zhimg.com/80/v2-ba4619bec37ef28792ff2eca402c6852_720w.webp)



写完后提交。

获取

```text
cicd.yml
```

文件的路径：





![img](https://pic1.zhimg.com/80/v2-e52ab68199f8c55aaf4995ca8ca5921c_720w.webp)



[http://172.29.9.101/devops4/devops4-devops-service/-/raw/main/cicd.yml](https://link.zhihu.com/?target=http%3A//172.29.9.101/devops4/devops4-devops-service/-/raw/main/cicd.yml)



![img](https://pic1.zhimg.com/80/v2-59e059f046e850faa9544cc849a75e38_720w.webp)



- 来到另一个项目里devops4-monrepo-service



![img](https://pic4.zhimg.com/80/v2-6721da953ef871c2a2014fea329b924b_720w.webp)



在项目>

```text
Settings
```

\>

```text
CI/CD
```

\>

```text
Gerneral pipelines
```

里把刚才拷贝的那个链接填在这里，保存：





![img](https://pic4.zhimg.com/80/v2-310de494f28fb94eda0e9785d28ae56f_720w.webp)



- 然后，直接运行这个流水线，观察效果：



![img](https://pic1.zhimg.com/80/v2-671cd3024bd3fc30de8ea02e5306cd44_720w.webp)





![img](https://pic1.zhimg.com/80/v2-f941f276643eef0ee52e7911de962e80_720w.webp)

