# kubeconfig文件

用于访问集群的配置文件

### 位置

默认　~/.kube/config

或export KUBECONFIG=/XXX/config

或 kubectl --kubeconfig /xxx/config  执行时指定





### 生成kubeconfig

参考:　

为普通用户创建一个kubeconfig文件,而不是使用一安装时产生的管理员帐号



#### 方式１:通过token

 https://blog.csdn.net/weixin_44267608/article/details/105536454?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-105536454-blog-118467210.t5_refersearch_landing&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-105536454-blog-118467210.t5_refersearch_landing&utm_relevant_index=2

https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengaddingserviceaccttoken.htm

```
#查看 view cluserrole
kubectl describe clusterrole view

#创建一个serviceaccount
kubectl create sa readonly -n kube-system 

#将其进行绑定
kubectl create clusterrolebinding readonly --clusterrole=view --serviceaccount=kube-system:readonly

#查看rolebinding token  #v1.24不会自动创建这个secret,请见后面的操作,要手动创建一个secret
# kubectl get secret -n kube-system |grep readonly
readonly-token-swp7v                             kubernetes.io/service-account-token   3      5m22s

get secret readonly-token-swp7v -o yaml -n kube-system

apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1ESXdOakUwTVRJME1Gb1hEVE15TURJd05ERTBNVEkwTUZvd0ZURVRNQ
kVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSjdCCi84YXZkak0yMUZGNVpMVXU1Smd6dERodmlKeDNpblRrM3ozaXF1NlV3cnRwMTgyWW9ZZUl3a1ZCa2JaNzdMVzgKajJ4QmhLaWNEN3dyZTdmYmV1MFMv
cU93cWU2N0x1aGRVMERFZHRGQ2FyVjBiZW5nUm1CZU52ODZkRlZFa2JSagpoRTFYQ1g5U2M3QVJIMG5EVlNCK2xPT3ZiUkhOV0U3Ti9UMlQwdFlIcWRoUE5uc2VaWUtFaDlhODlXWmM5TUkvCnNQOXZGMndKcU9KWG9CTVZMbVhuSllmVHNMTHFHaERWS0d1UjdmUFMweHlQbG0xeGR
CelRUdEJMVFhLTzJUeW0KUWMvMk1CbnpXbVhDVWNzRDE5SWZWb0YzdEJDMWs2am55ekZlMjNwb3NiS01US2htSDhwbU8wTjFESXNjWTR0UApBL0xGMkVrOXJqT2Zsd2pCYVJjQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0
hRWURWUjBPQkJZRUZBaHFTR003UExaQXIvNzZYL01TMUdnc3BnMTVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCODRTdm51ZlpEdnN4Qml1OVRRK0VBbmQrY1dlS1pGTlpJaGhlQ2o0cERjeExaQVFkcwpBNkhHRERJbUpJY29IQUVnT3NhS21VT0J3SFFSM041R2F6cmJvQzBKR
W8vTWwydGNrazZyR3dIMExuZWZGYkxTClVxQ0tNaVgwNHYwSzRLckp2ZnZNZ24rU095SEJTM1ZRSTdUbkxCNngyeW43aHFKa3VHd2ZqSWQ1bm9LWExDcVoKM1lLdzZQVlE1OVVlWHFiSkNaTXp5WXRHeUxlN1diVkZyUDU5SjRzUnZvcjJwL0VsV1JIRXVrbWQ0K096aFVWaQpsQk8z
QU9kdklHOVQvK1k5YkIwVWNhN0toWHhkMnJ0WmgrWHV2YUlMZ01WZXgvSS9hYXo5bUcyUlFvNmtndGtGCnFSekhQeG1ZSTEzdUVmVSt2LzJ3RkUxQ1pIN3VDaGYrZFdHcAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  namespace: a3ViZS1zeXN0ZW0=
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkltRXpOa2h2VVZNMU0zRnVVVEJ2YWpjeE9FVnpjVEZ4ZUZCaFFXSmtSbkZrVkVKa1FWQnRlRzk1VDAwaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbG
N5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUpyZFdKbExYTjVjM1JsYlNJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZqY21WMExtNWhiV1VpT2lKeVpXRmtiMjVzZVMxMGIydGxiaTF6ZDNBM2RpSXNJbXQxWW1WeWJtV
jBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMbTVoYldVaU9pSnlaV0ZrYjI1c2VTSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMblZwWkNJNklqWTJPVGRoT1RaaUxXUTVO
VFV0TkdReFl5MWhORGs1TFRsaE4yTTNNREl5TWpKa1pDSXNJbk4xWWlJNkluTjVjM1JsYlRwelpYSjJhV05sWVdOamIzVnVkRHByZFdKbExYTjVjM1JsYlRweVpXRmtiMjVzZVNKOS5SeDZjUElKUVZkUEFadE43QW9SZVMtRXVQLXEtLWdIT2tVcTBvLXB1ZW9WZGh4MDBKeGVseDB
GMEtzVkhSLU9seXJYX1BOWXBEUzBabXZKU2h1eTF0blJuWTYtcEpjeHhZSGh0NngycjZYODVQcGZQNEYtbkMwNFRGUmxTM21lbU5qTlA4bVZaSkR5M1lfOXhpNWY0aGtBV1M0S082akRRZ2dKM04tTGMyNnRmc25ob3NoNnU2VTZlTmt3ODNRc1lBRTgwRXF0cWctODdQN05zUXlQNk
UwUTZrNjlJekZiUzBDWVFkSGc0OU9ZdWxBOFppbDJsUWtyRXFXanhBQ1dERHZuOHUycTN0Q3VhRHprSlRkSUlwdU5iS2JKYVRuWEIycEFqSFFmZU04QUxkaWxxWkF2Ylpfclh1akhJZG9uSWFZWnlTRkVyNGNvNmZsNEw4NHIzc3c=
kind: Secret
metadata:


```

手工建立一个kubeconfig,***将以上的token　base64 -d解密后放在下面的kubeconfig中***

```
# cat token.kubeconfig 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1ESXdOakUwTVRJME1Gb1hEVE15TURJd05
ERTBNVEkwTUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSjdCCi84YXZkak0yMUZGNVpMVXU1Smd6dERodmlKeDNpblRrM3ozaXF1NlV3cnRwMTgyWW9ZZUl3a1ZCa2JaNzdMVzgKajJ4Qm
hLaWNEN3dyZTdmYmV1MFMvcU93cWU2N0x1aGRVMERFZHRGQ2FyVjBiZW5nUm1CZU52ODZkRlZFa2JSagpoRTFYQ1g5U2M3QVJIMG5EVlNCK2xPT3ZiUkhOV0U3Ti9UMlQwdFlIcWRoUE5uc2VaWUtFaDlhODlXWmM5TUkvCnNQOXZGMndKcU9KWG9CTVZMbVhuSllmVHNMTHFHaERWS
0d1UjdmUFMweHlQbG0xeGRCelRUdEJMVFhLTzJUeW0KUWMvMk1CbnpXbVhDVWNzRDE5SWZWb0YzdEJDMWs2am55ekZlMjNwb3NiS01US2htSDhwbU8wTjFESXNjWTR0UApBL0xGMkVrOXJqT2Zsd2pCYVJjQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRF
d0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZBaHFTR003UExaQXIvNzZYL01TMUdnc3BnMTVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCODRTdm51ZlpEdnN4Qml1OVRRK0VBbmQrY1dlS1pGTlpJaGhlQ2o0cERjeExaQVFkcwpBNkhHRERJbUpJY29IQUVnT3NhS21VT0J
3SFFSM041R2F6cmJvQzBKRW8vTWwydGNrazZyR3dIMExuZWZGYkxTClVxQ0tNaVgwNHYwSzRLckp2ZnZNZ24rU095SEJTM1ZRSTdUbkxCNngyeW43aHFKa3VHd2ZqSWQ1bm9LWExDcVoKM1lLdzZQVlE1OVVlWHFiSkNaTXp5WXRHeUxlN1diVkZyUDU5SjRzUnZvcjJwL0VsV1JIRX
VrbWQ0K096aFVWaQpsQk8zQU9kdklHOVQvK1k5YkIwVWNhN0toWHhkMnJ0WmgrWHV2YUlMZ01WZXgvSS9hYXo5bUcyUlFvNmtndGtGCnFSekhQeG1ZSTEzdUVmVSt2LzJ3RkUxQ1pIN3VDaGYrZFdHcAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.146.91:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: default
    user: my-service
  name: test
current-context: test
kind: Config
preferences: {}
users:
- name: my-service
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6ImEzNkhvUVM1M3FuUTBvajcxOEVzcTFxeFBhQWJkRnFkVEJkQVBteG95T00ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlL
XN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJyZWFkb25seS10b2tlbi1zd3A3diIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJyZWFkb25seSIsImt1YmVybmV0ZXMuaW8vc2Vydmlj
ZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjY2OTdhOTZiLWQ5NTUtNGQxYy1hNDk5LTlhN2M3MDIyMjJkZCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpyZWFkb25seSJ9.Rx6cPIJQVdPAZtN7AoReS-EuP-q--gHOkUq0o-pueoVdhx00Jx
elx0F0KsVHR-OlyrX_PNYpDS0ZmvJShuy1tnRnY6-pJcxxYHht6x2r6X85PpfP4F-nC04TFRlS3memNjNP8mVZJDy3Y_9xi5f4hkAWS4KO6jDQggJ3N-Lc26tfsnhosh6u6U6eNkw83QsYAE80Eqtqg-87P7NsQyP6E0Q6k69IzFbS0CYQdHg49OYulA8Zil2lQkrEqWjxACWDDvn8u
2q3tCuaDzkJTdIIpuNbKbJaTnXB2pAjHQfeM8ALdilqZAvbZ_rXujHIdonIaYZySFEr4co6fl4L84r3sw
```

测试



```
# kubectl --kubeconfig=./token.kubeconfig get ns
NAME                   STATUS   AGE
0509                   Active   140d
default                Active   232d
ingress-nginx          Active   231d
kube-node-lease        Active   232d
kube-public            Active   232d
kube-system            Active   232d
kubernetes-dashboard   Active   230d
logging                Active   225d
monitoring             Active   230d
test                   Active   231d
test1                  Active   224d
test2                  Active   224d
test3                  Active   224d
[root@kubernetes-master ~/practice/user] ens33 = 192.168.146.91
# kubectl --kubeconfig=./token.kubeconfig get node
Error from server (Forbidden): nodes is forbidden: User "system:serviceaccount:kube-system:readonly" cannot list resource "nodes" in API group "" at the cluster scope

```



#### 方式2,通过用户证书

,在master上执行以下脚本:



```
#创建用户证书
cat user.sh

openssl genrsa -out $1.key 2048
openssl req -new -key $1.key -out $1.csr -subj "/CN=$1/O=MGM"
openssl x509 -req -in $1.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out $1.crt -days 3650


#从~/.kube/config 复制一个andy.kubeconfig,删除掉其中的users, contexts信息,保留cluster下的信息

clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1ESXdOakUwTVRJME1Gb1hEVE15TURJd05
ERTBNVEkwTUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSjdCCi84YXZkak0yMUZGNVpMVXU1Smd6dERodmlKeDNpblRrM3ozaXF1NlV3cnRwMTgyWW9ZZUl3a1ZCa2JaNzdMVzgKajJ4Qm
hLaWNEN3dyZTdmYmV1MFMvcU93cWU2N0x1aGRVMERFZHRGQ2FyVjBiZW5nUm1CZU52ODZkRlZFa2JSagpoRTFYQ1g5U2M3QVJIMG5EVlNCK2xPT3ZiUkhOV0U3Ti9UMlQwdFlIcWRoUE5uc2VaWUtFaDlhODlXWmM5TUkvCnNQOXZGMndKcU9KWG9CTVZMbVhuSllmVHNMTHFHaERWS
0d1UjdmUFMweHlQbG0xeGRCelRUdEJMVFhLTzJUeW0KUWMvMk1CbnpXbVhDVWNzRDE5SWZWb0YzdEJDMWs2am55ekZlMjNwb3NiS01US2htSDhwbU8wTjFESXNjWTR0UApBL0xGMkVrOXJqT2Zsd2pCYVJjQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRF
d0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZBaHFTR003UExaQXIvNzZYL01TMUdnc3BnMTVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCODRTdm51ZlpEdnN4Qml1OVRRK0VBbmQrY1dlS1pGTlpJaGhlQ2o0cERjeExaQVFkcwpBNkhHRERJbUpJY29IQUVnT3NhS21VT0J
3SFFSM041R2F6cmJvQzBKRW8vTWwydGNrazZyR3dIMExuZWZGYkxTClVxQ0tNaVgwNHYwSzRLckp2ZnZNZ24rU095SEJTM1ZRSTdUbkxCNngyeW43aHFKa3VHd2ZqSWQ1bm9LWExDcVoKM1lLdzZQVlE1OVVlWHFiSkNaTXp5WXRHeUxlN1diVkZyUDU5SjRzUnZvcjJwL0VsV1JIRX
VrbWQ0K096aFVWaQpsQk8zQU9kdklHOVQvK1k5YkIwVWNhN0toWHhkMnJ0WmgrWHV2YUlMZ01WZXgvSS9hYXo5bUcyUlFvNmtndGtGCnFSekhQeG1ZSTEzdUVmVSt2LzJ3RkUxQ1pIN3VDaGYrZFdHcAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.146.91:6443
  name: kubernetes

以上部分的certificate-authority-data来自　master节点上的:cat /etc/kubernetes/pki/ca.crt |base64
https://cloud.tencent.com/document/product/1159/47752




sh user.sh andy

cat user.sh
kubectl config set-credentials $1 --client-certificate=/root/practice/user/$1.crt --client-key=/root/practice/user/$1.key --embed-certs=true --kubeconfig=$1.kubeconfig

kubectl config set-context andy-test --cluster=kubernetes --namespace=default --user=andy --kubeconfig=$1.kubeconfig


```

 最终的内容如下所示:

cat andy.kubeconfig 

```

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1ESXdOakUwTVRJME1Gb1hEVE15TURJd05
ERTBNVEkwTUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSjdCCi84YXZkak0yMUZGNVpMVXU1Smd6dERodmlKeDNpblRrM3ozaXF1NlV3cnRwMTgyWW9ZZUl3a1ZCa2JaNzdMVzgKajJ4Qm
hLaWNEN3dyZTdmYmV1MFMvcU93cWU2N0x1aGRVMERFZHRGQ2FyVjBiZW5nUm1CZU52ODZkRlZFa2JSagpoRTFYQ1g5U2M3QVJIMG5EVlNCK2xPT3ZiUkhOV0U3Ti9UMlQwdFlIcWRoUE5uc2VaWUtFaDlhODlXWmM5TUkvCnNQOXZGMndKcU9KWG9CTVZMbVhuSllmVHNMTHFHaERWS
0d1UjdmUFMweHlQbG0xeGRCelRUdEJMVFhLTzJUeW0KUWMvMk1CbnpXbVhDVWNzRDE5SWZWb0YzdEJDMWs2am55ekZlMjNwb3NiS01US2htSDhwbU8wTjFESXNjWTR0UApBL0xGMkVrOXJqT2Zsd2pCYVJjQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRF
d0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZBaHFTR003UExaQXIvNzZYL01TMUdnc3BnMTVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCODRTdm51ZlpEdnN4Qml1OVRRK0VBbmQrY1dlS1pGTlpJaGhlQ2o0cERjeExaQVFkcwpBNkhHRERJbUpJY29IQUVnT3NhS21VT0J
3SFFSM041R2F6cmJvQzBKRW8vTWwydGNrazZyR3dIMExuZWZGYkxTClVxQ0tNaVgwNHYwSzRLckp2ZnZNZ24rU095SEJTM1ZRSTdUbkxCNngyeW43aHFKa3VHd2ZqSWQ1bm9LWExDcVoKM1lLdzZQVlE1OVVlWHFiSkNaTXp5WXRHeUxlN1diVkZyUDU5SjRzUnZvcjJwL0VsV1JIRX
VrbWQ0K096aFVWaQpsQk8zQU9kdklHOVQvK1k5YkIwVWNhN0toWHhkMnJ0WmgrWHV2YUlMZ01WZXgvSS9hYXo5bUcyUlFvNmtndGtGCnFSekhQeG1ZSTEzdUVmVSt2LzJ3RkUxQ1pIN3VDaGYrZFdHcAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.146.91:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: default
    user: andy
  name: andy-test
current-context: andy-test
kind: Config
preferences: {}
users:
- name: andy
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNyakNDQVpZQ0NRRDYrNE82aHdlMGhUQU5CZ2txaGtpRzl3MEJBUXNGQURBVk1STXdFUVlEVlFRREV3cHIKZFdKbGNtNWxkR1Z6TUI0WERUSXlNRGt5TnpBd01qUTBNVm9YRFRNeU1Ea3
lOREF3TWpRME1Wb3dIVEVOTUFzRwpBMVVFQXd3RVlXNWtlVEVNTUFvR0ExVUVDZ3dEVFVkTk1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBCk1JSUJDZ0tDQVFFQXU2MlZyN050cUkvTnJoUVdLaDZYQ3FCOG5MRWZSYzB4MW54akZJNlNwcVdZbzlLbmlzdzEKT3hGaUw5R
3lHR3V5ZGk4cnArK3UvRHdxSHlpUis5Vi9qRjdMMWx6Unkyd1duWXRzYVZKeW5ldkdza2F1OURlbAp5TnBiSm9na2xFcTM1dGtTR1krYTdLSmtxcnlObWF6UTNUeGlVNHBnbHQ5cmZZdXd6RFoyREovWTVSbWR2UFBTCmpNMkttUFVwSjJQc2ZkdzZxajBOa2hVVDJiNlNhbUVUaEFQ
aDMxWnpCTmMzYjFxRy9IMUY0SExSVi80dzR1MngKMFZRbzhVajZFTVlBa3c2czMvcE0xN25odUtkb1UrVWpCbjQ4YmZLTldOL3lyanlBQW8wVmRQZElKYWIyZjJ6RApOaW1MOEdEUW1tWFowTStCNGVpVGxYZzdUUW1NODBpWHZRSURBUUFCTUEwR0NTcUdTSWIzRFFFQkN3VUFBNEl
CCkFRQTcwM25tTXl0N2ZFZnVFNEFzTCtXK1R0d2YxbGlJU1ZtSkdlMlpnbGRndVJuK0hmQ0ZwTkdwSzNUTGE2WW0KTkFodG94RmRwMlhERnM5OWo2M3hGMFdSTCtKSjB1a2Y0WWtiK3N0YmRxTmVEcm1DNmdOeHJTTVowOGFMakd0NAplaHN5QVNkNXdYV2tvOEFiMlZnUVNKOXJ1Vj
dxNkxxOGd0ay95U0FvcW9Ib0J2dWpnQkVuaFlPc0lwempITlByCjZaYjA5cXZ5SlZ5eGNoUDcyaGpDYm0zWE0wTFYzaTlJRHJFNFJjdnc1bGVPVFlLUWRIdnVKRzJTcDBXVUlaVGEKdE45S3VUeWpUN2RaNHFrS0hGSnFHWXR6K0c3NVo4ME54NlQ3MnFUaklxaEZtc2ZKZHovY2tES
3lSbXBNdlNEZQpzbkorbjlRQjRKRHBtQmhSSTFkQmtEMXkKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBdTYyVnI3TnRxSS9OcmhRV0toNlhDcUI4bkxFZlJjMHgxbnhqRkk2U3BxV1lvOUtuCmlzdzFPeEZpTDlHeUdHdXlkaThycCsrdS9Ed3FIeWlSKzlWL2pGN0wxbHpSeT
J3V25ZdHNhVkp5bmV2R3NrYXUKOURlbHlOcGJKb2drbEVxMzV0a1NHWSthN0tKa3FyeU5tYXpRM1R4aVU0cGdsdDlyZll1d3pEWjJESi9ZNVJtZAp2UFBTak0yS21QVXBKMlBzZmR3NnFqME5raFVUMmI2U2FtRVRoQVBoMzFaekJOYzNiMXFHL0gxRjRITFJWLzR3CjR1MngwVlFvO
FVqNkVNWUFrdzZzMy9wTTE3bmh1S2RvVStVakJuNDhiZktOV04veXJqeUFBbzBWZFBkSUphYjIKZjJ6RE5pbUw4R0RRbW1YWjBNK0I0ZWlUbFhnN1RRbU04MGlYdlFJREFRQUJBb0lCQUUxSkVsY2tZSWdGa0FHYgpxL1QwVythNGFCaHVxQjRxZmRlQnFadVJpcnF0ZnNvWHVYN2kw
UmpkODcwVmNXMjFDK3kzU0JjRUVOODJOM0pWClZxaUtKdGc3UVYycEk0dk5teEtOazd0YmhHK2I1RnNOMklZaFZGZjk4NE5PbFNHc0UwY3hKTTc1NENhS1NVSTIKRzJtcFRPbU9NRCtPd0cvZzJYYjl5M1NOQ05meFZvbVpVVDFKMGxxTXZHZ2lIR1dqekhxNGcyUFZxTHA0WGZsYwp
0Qy9LQm0wYkRORXg0U3pxeVJVcUxpdDdpLytLb0lRWWllZkh1c2hXb3cyNzNJN1Jla0FOcHJCUmY3N0psU3VVCnZNUDViaVNpNVBKOWZaVHBrbWdjcnBGQ0IyOHpabGhieklxM0M5RlF3aXJpRy9UdTFpZ3BJckJsc1E5SFI2Y0YKR2ZjTzNnRUNnWUVBNmdWN1N5bTVFRllTZjVRM3
lNcEp2U25Ha1ArckVWSFFwcWEvSXRnc0dFdUdZL28xSHA2dAprTEd2RjV6OEEyNnJqajRjYSsyUzdDb0YzbVRvZnJkSklJdnFsNWVVR3grcmtmWmFHQVFkT05BQjFsN2hyYlhBCk8vOUxRTkZPYzBTV0dDMFJkK25LeVVSN1ZaNzhOTUYzakQ0M3VyY3VEM2kwd2sxbXRISmVGV0VDZ
1lFQXpVM2gKcnBsL3ZmWHUyRWZmNWpwcnRDNGg1ZlEreFgrVldyUmxMd2FZTGtNUDdzWERqUEg3ZEx1Vy9YMnc2NjJOSWhwZAplMWlzWE5yYlN0a3U4ZHQvbU10MU4zMFAzNjhXdkRuMG1hWDNVc040dEFTbGZxdXJSZzJlUjBzeEhvM0VsVFJtClAwV25PZk03RDRzM0ZxT3JyeUlQ
c0NpMnlLcHdUK3d6eHgvUUE5MENnWUVBcmppQ2Y0U0NTQTBzSEZxbDVRL2sKTWJXbUpGQ1NkSVJxUjRjZ0NMclNxUXVnODFGVXRvVC9SaGRoK0x4Z1NkWGNWanNiUWFaT21RNzM2NGRJSnY0ZwpyT0E5TjdvYkFzNWpGbmxHdlZaaXd1Ym9WMFd2MFhqYnNrN3ZJVVF4bVZMWVF3dGF
VK0Z6bkw4azZxcy9xNjdVClowM3lnMGpZTzZCclFiUUo0QkM4eGtFQ2dZQjljL3pYUTJjaXZoaHdReU5YUFJXNWFZTS9VRXZYUllvUGZqSmkKVlFaREpxbWl2MmdxUldmaHdndVc1T3BxYVlmWGNnTHpyNURMd05URjNRYnB0YlkzdVFQc24xaEcyR2Z0SndFSApycm45OGdKZEJvWF
diTEpoUVVzWng2SEJTT0g2UnFYQVBpRGNzWHZDbU5CVjNqZTM4ZmxpTE03Y3VnR2RaUG1TCjBTYThaUUtCZ1FDa0ZKckhvWVNNZVR3OFFSRnkrbzhnWG13Q0tRN2prajIza2dCN09Oc21uaUFENnRFaDJ0S0wKY2U1cDk4K3RpbFN3RllCQzdLb2Y3L3NPYWZvRVZYdWdzT0NoOFJiW
k81SDlpTWovdWhaSzFLREp5akhCUXQyQworQWZJSE92ZWtabFVCNUhURmVOVUUwVlMxME9nK1RmTzZNMzdrelNDcXA0V1JtY1crdjRkb3c9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

```



还要给用户andy加权限,先创建一个只读的cluster-role,然后授权给andy

```
# cat clusterrole-readonly.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
    name: cluster-reader
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "watch", "list"]


kubectl apply -f  clusterrole-readonly.yaml

#授权
kubectl create clusterrolebinding andy-reader --clusterrole=cluster-reader --user=andy

#测试
# kubectl --kubeconfig=./andy.kubeconfig get node -n default
NAME                 STATUS     ROLES    AGE    VERSION
kubernetes-master    Ready      master   232d   v1.19.5
kubernetes-worker1   Ready      worker   232d   v1.19.5
kubernetes-worker2   NotReady   worker   232d   v1.19.5
[root@kubernetes-master ~/practice/user] ens33 = 192.168.146.91
# kubectl --kubeconfig=./andy.kubeconfig get pod -n default
NAME                                      READY   STATUS    RESTARTS   AGE
nfs-client-provisioner-57df8659f9-shpf4   1/1     Running   15         207d
[root@kubernetes-master ~/practice/user] ens33 = 192.168.146.91

```



## kubeconfig结构

包括三部分

clusters

users

contexts  (使用context把cluster和 user组织起来)



### 练习,添加一个用户,是某个namespace的管理员



将用户andy添加到test namespace的管理员权限上,　参考:https://www.cnblogs.com/roy2220/p/14772838.html

```
# cat ns-admin.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: test
  name: admin
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: test
  name: andy-test-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: andy

```



测试:

```
# kubectl --kubeconfig=./andy.kubeconfig scale deployment whoami --replicas=0 -n test
deployment.apps/whoami scaled

```







-----------

注意在1.24以后,serice acount的secret不会自动创建了,要手工创建

https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md#urgent-upgrade-notes

https://stackoverflow.com/questions/72256006/service-account-secret-is-not-listed-how-to-fix-it

```
apiVersion: v1
kind: Secret
metadata:
  name: user1-token
  annotations:
    kubernetes.io/service-account.name: sa-user1
type: kubernetes.io/service-account-token


```



kubeconfig中的certificate-authority-data的数据来源

```
发现
echo "xxx"|base６４ -d
是集群master中　/etc/kubernetes/ssl/ca.crt的内容
```

