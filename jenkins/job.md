### build job free style:

```
 代码：
https://gitee.com/xuqw123/log-collection

使用阿里云的maven,构建 配置

mvn -s configuration/aliyun-settings.xml package 

TAG=registry.crc.test:5000/abc/log-collection:$BUILD_NUMBER
docker build -t $TAG .
docker push $TAG


#docker login的口令和用户在这里保存：
$HOME/.docker/config.json

```



### ci/cd job free style: deploy without plugin

  ,本地build-free job, git: https://gitee.com/xuqw123/log-collection

```
mvn -s configuration/aliyun-settings.xml package 

TAG=registry.miketemp.com:5000/abc/log-collection:$BUILD_NUMBER
podman build -t $TAG .
#docker push $TAG

export KUBECONFIG=/root/kube-config   #也可加入 jenkins的Global properties

sed -i s/quay.io/qxu/log-collection/$TAG/ deployment/deployment.yaml
kubectl apply -f deployment/deployment.yaml

```



### cicd pipeline 版本：

```
pipeline {
    agent any
    environment {
        REPO_URL = "https://gitee.com/xuqw123/log-collection"
        TAG= "registry.miketemp.com:5000/abc/log-collection:$BUILD_NUMBER"
        KUBECONFIG="/root/kube-config"
    }
    
    stages {
        stage ('Sync Code') {
            steps {
                 git branch: 'master', url: "$REPO_URL"
                 //git branch: 'master', credentialsId: 'gitee', url: "$REPO_URL"
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
          			podman build -t $TAG .
                """
            }
        }
        
        stage('push image') {
            steps {
        
                echo "push images"
                sh """
                #podman login -u admin -p admin registry.miketemp.com:5000
                podman push  $TAG
                """
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



#add replace tag steps

sed -i "s|quay.io/qxu/log-collection|$TAG|" deployment/deployment.yaml
