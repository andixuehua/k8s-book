#### 常见的Pipeline用法

- 前提,外部jenkins已经安装完成,可以构建maven项目

- #### 基本的pipeline 新建一个job: pipe-basic, to run

```
def mvnCmd = "mvn -s configuration/aliyun-settings.xml "

pipeline {
  agent any
  stages {

    stage('CheckStyle') {
      steps {
        git branch: 'master', url: 'https://gitee.com/xuqw123/log-collection' 
        //git branch: 'master', credentialsId: 'gitee', url: "https://gitee.com/xuqw123/log-collection"
        sh "${mvnCmd} clean checkstyle:checkstyle"
      }
    }  
    stage('Build App') {
      steps {
        sh "${mvnCmd} package -DskipTests=true"
      }
    }
    stage('Test') {
      steps {
        sh "${mvnCmd} test"
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
      }
    }
    stage('ArchiveFile') {
      steps{
        archiveArtifacts '**/target/*.jar'
      }
    }

  }
}
```

#### 以上，如果 git仓库有认证，使用以下方式：



```
新建一个credential, gitee, usrename/password方式


使用Sample Step中的，git: Git插件生成访问代码
git branch: 'master', credentialsId: 'gitee', url: "https://gitee.com/xuqw123/log-collection"

```



- #### 可以确认的pipeline

```
def mvnCmd = "mvn -s configuration/aliyun-settings.xml"

pipeline {
  agent any

  stages {

    stage('CheckStyle') {
      steps {
        git branch: 'master', url: 'https://gitee.com/xuqw123/log-collection'  
        sh "${mvnCmd} clean checkstyle:checkstyle"
      }
    }  
    stage('Build App') {
      steps {
        sh "${mvnCmd} package -DskipTests=true"
      }
    }
    stage('Test') {
      steps {
        sh "${mvnCmd} test"
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
      }
    }
    stage('ArchiveFile') {
      steps{
        archiveArtifacts '**/target/*.jar'
      }
    }

     stage('TestConfirm ') {
            options {
                timeout(time: 1, unit: 'MINUTES') 
            }
            steps {
                input "Does the Test environment look ok?"
            }
        }

        stage('Deploy') {
            steps {
                //sh 'scp spring-boot-log4j-2-demo/target/log4j2-demo-0.0.1-SNAPSHOT.jar root@192.168.1.42:/tmp/'
                sh 'echo "skip to deploy"'
            }
        }

  }
}
```



#### 带参数的pipeline

```
def mvnCmd = "mvn -s configuration/aliyun-settings.xml"

pipeline {
  agent any
  parameters {
        choice("name": 'BRANCH',choices: 'master\ntest' )
    }

  stages {

    stage('CheckStyle') {
      steps {
        git branch: "${params.BRANCH}", url: 'https://gitee.com/xuqw123/log-collection'  
        sh "${mvnCmd} clean checkstyle:checkstyle"
      }
    }  
    stage('Build App') {
      steps {
        sh "${mvnCmd} package -DskipTests=true"
      }
    }

  }
}
```

参见： https://plugins.jenkins.io/git-parameter/



以上也可以使用Sample Step中的  **properties**: set job properties生成示例代码



#### push to docker registry with credientials

credientials的用法：

https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#string-interpolation

```
steps{
        withCredentials([usernamePassword(credentialsId: 'local-docker-registry', passwordVariable: 'password', usernameVariable: 'username')]) {
        sh 'podman login -u $username -p $password registry.miketemp.com:5000'
        
        }
    }

```



#### docker build pipeline

```
pipeline {
    agent any
    environment {
        REPO_URL = "https://gitee.com/xuqw123/log-collection"
        TAG="registry.crc.test:5000/abc/log-collection:$BUILD_NUMBER"
    }
    
    stages {
        stage ('Sync Code') {
            steps {
                 git branch: 'master', url: "$REPO_URL"
            }    
        }
        stage ('Build') {
            steps {
                echo "Build..."
                sh 'mvn -s configuration/aliyun-settings.xml package ' 
            }    
        }
        
        stage('Arcive Artifact') {
            steps {
              echo "save artifact"
              archiveArtifacts '**/target/*.jar'
            }
        }
        
        stage('build and tag image') {
            steps {
                echo "begin build images"
                sh """
          			docker build -t $TAG .
                """
            }
        }
        
        stage('push image') {
            steps {
                echo "push images"
                 sh "docker push $TAG"
            }
        }
    }    
}
```



#### 例子：本地： cd-k8s-pipe-docker-cred

```
pipeline {
    agent any
    environment {
        REPO_URL = "https://gitee.com/xuqw123/log-collection"
        TAG="registry.crc.test:5000/abc/log-collection:$BUILD_NUMBER"
        KUBECONFIG="/root/.kube/config" 
    }
    
    stages {
        stage ('Sync Code') {
            steps {
                 git branch: 'master', url: "$REPO_URL"
            }    
        }
        stage ('Build') {
            steps {
                echo "Build..."
                sh 'mvn -s configuration/aliyun-settings.xml package ' 
            }    
        }
        
        stage('Arcive Artifact') {
            steps {
              echo "save artifact"
              archiveArtifacts '**/target/*.jar'
            }
        }
        
        stage('build and tag image') {
            steps {
                echo "begin build images"
                sh """
          			docker build -t $TAG .
                """
            }
        }
        
        stage('push image') {
            steps {
        
                echo "push images"
                //withCredentials([usernamePassword(credentialsId: 'local-docker-registry', passwordVariable: 'password', usernameVariable: 'username')]) {
                //  sh """
                //  podman login -u $username -p $password registry.miketemp.com:5000
                //  podman push  $TAG
                 // """
               // }
               sh "docker push $TAG"
            }
        }
        
        stage("deploy app") {
            steps {
                echo "rolling app"
                sh """
                sed -i 's|quay.io/qxu/log-collection|$TAG|' deployment/deployment.yaml
                kubectl apply -f deployment/deployment.yaml -n test
                """
            }
        }
        
    }    
}
```





两种语法：

https://www.cnblogs.com/zhaobowen/p/13514656.html#%E4%B8%A4%E7%A7%8Dpipeline%E8%AF%AD%E6%B3%95%E7%9A%84%E9%80%89%E6%8B%A9

```
Jenkinsfile可以使用两种语法编写 - Declarative和Scripted。声明式和脚本化管道的构造从根本上不同

在Declarative Pipeline语法中，pipeline块定义了整个管道中完成的所有工作。表达式与Groovy语法基本相同，区别在于：
Pipeline的顶级必须是一个块，特别是：pipeline {}
没有分号作为语句分隔符。 每个声明都必须独立

推荐使用Declarative方法。难度较低

```

