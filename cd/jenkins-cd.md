- #### 用Jenkins调用k8s,实现蓝绿和金丝雀部署的切换

- 前提条件，蓝绿和金丝雀的基本部署已经实现

- 外部的Jenkins 实例



- ### 切换蓝绿部署的pipeline
  
  in shared jenkins instace , create a pipeline job: switch-bluegreen, and add the choice DEP_TYPE parameter: blue, green

```
pipeline {
    agent any
    parameters {
        choice("name": 'DEP_TYPE',choices: 'blue\ngreen' )
    }
    
    stages {
        stage("switch versin") {
            steps {
                echo "rolling app"
                sh """
                kubectl patch ing/log-collection-bluegreen  --type=json \
  -p='[{"op": "replace", "path": "/spec/rules/0/http/paths/0/backend/service/name", "value":"log-collection-${DEP_TYPE}"}]' -n demo-icbc-bluegreen-project
                """
            }
        }
    }    
}
```

测试



- ### 切换金丝雀部署的pipeline

in shared jenkins instace , create a pipeline job: switch-ab, 

 WEIGHT parameter with default value: 50

```
pipeline {
    agent any
    parameters {
        string("name": 'WEIGHT','defaultValue': "50" )
    }
    
    stages {
        stage("switch versin") {
            steps {
                echo "rolling app"
                sh """
                  kubectl patch ing/log-collection-b  --type=json \
  -p='[{"op": "replace", "path": "/metadata/annotations/nginx.ingress.kubernetes.io~1canary-weight", "value":"${WEIGHT}"}]' -n demo-icbc-ab-project
                """
            }
        }
    }    
}

```

测试

for i in {1..10}; do curl  http://ab-icbc.apps.test.crc.test; done;



- ### 升级部署的image版本的pipeline实现

- add a string paramater: TAG  (不要对蓝绿，AB进行切换，只能对yaml部署的进行image切换)

- 前提： [此部署已经存在](../basic/deployment.md)

```

pipeline {
    agent any
    parameters {
        string("name": 'TAG','defaultValue': "latest" )
    }
    
    stages {
        stage("switch versin") {
            steps {
                echo "rolling app"
                sh """
                  kubectl set image deployment/log-collection log-collection=quay.io/qxu/log-collection:"${TAG}" -n test
                """
            }
        }
    }    
}

```

检查

kubectl get deployment -o yaml log-collection -n test |grep image



可用的版本如下：

main

latest

test

v2

v3
