- #### 演示使用jenkinsfile构建的方法

```
#crate a pipepline job ,use https://github.com/mikerain/jenkinsfile-examples.git  

select "pipeline script from SCM"

use jenkinsfiles to demo:

#explain different declarative and scripted style

jenkinsfiles/001-stages-declarative-style.groovy
jenkinsfiles/002-stages-scripted-style.groovy

#parallel

jenkinsfiles/003-stages-parallel.groovy

#shared-library

jenkinsfiles/051-shared-library-using-global-variables.groovy


#parameterized
jenkinsfiles/070-parameterized-build-choices.groovy
```



完整的例子：,branch docker

https://gitee.com/xuqw123/log-collection

or

https://gitee.com/mikerain/log-collection.git



jenkinsfile:

注意此步中删除了，同步 代码的步骤，同步代码由jenkins的scm插件来进行。

```shell
pipeline {
    agent any
    environment {
        TAG="registry.crc.test:5000/abc/log-collection:$BUILD_NUMBER"
        KUBECONFIG="/root/kube-config" 
    }
    
    stages {
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
                  docker push  $TAG
                  """
                }
            }
        }
        
        stage("deploy app") {
            steps {
                echo "rolling app"
                sh """
                kubectl apply -f deployment/deployment.yaml
                """
            }
        }
        
    }    
}
```

