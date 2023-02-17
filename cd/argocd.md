- ### 安装ArgoCD

  https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/

```
  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  
  .暴露argocd ui
  # NodePort方式
  kubectl patch service -n argocd argocd-server -p '{"spec": {"type": "NodePort"}}'
  
  
  kubectl get svc -n argocd
  kubectl get pod -n argocd

  
  3.登陆argocd
  # 获取admin登陆密码
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
  
  from web: https://xxx:nodeport
  
  
#user ingrss, 只能使用这个,测试通过 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    kubernetes.io/tls-acme: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    # If you encounter a redirect loop or are getting a 307 response code
    # then you need to force the nginx ingress to connect to the backend using HTTPS.
    #
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx 
  rules:
  - host: argocd.apps.taikang1.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https
  tls:
  - hosts:
    - argocd.apps.taikang1.local
    secretName: argocd-secret # do not change, this is provided by Argo CD
```

  

  ### 使用ArgoCD进行应用的部署，升级，回滚

```
 https://github.com/mikerain/argocodemo.git
or

https://gitee.com/mikerain/argocddemo.git
all public repo
```

- create a test-argocd,

```
project: default
application: test
repo: https://github.com/mikerain/argocodemo.git
path: log-collection   #yaml 所在的目录
namespace: test-argo  #提前创建好
```

![image-20210219110650872](http://localhost:4000/cd/image-20210219110650872.png)

- sync application

```
from the menu to sync config
```

- modify deployment , to test sync , to test rollback

```
vi deployment.yaml  #change to image to main v2 v3 latest
git commit deployment.yaml -m "change version"
git push origin main
```

![image-20210219110136624](http://localhost:4000/cd/image-20210219110136624.png)

