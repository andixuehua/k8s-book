```

[root@trainee monitor]# kubectl api-resources |grep -i prometheus
prometheuses                      prom             monitoring.coreos.com/v1               true         Prometheus
prometheusrules                   promrule         monitoring.coreos.com/v1               true         PrometheusRule


[root@trainee monitor]# kubectl api-resources |grep -i alert
alertmanagerconfigs               amcfg            monitoring.coreos.com/v1alpha1         true         AlertmanagerConfig
alertmanagers                     am               monitoring.coreos.com/v1               true         Alertmanager

```



一般在这里配置邮件通知等:

```
[root@trainee monitor]# kubectl get alertmanagerconfigs -o yaml -n monitoring 
apiVersion: v1
items: []
kind: List
metadata:
  resourceVersion: ""



```

或可参考这里,修改

https://stackoverflow.com/questions/60815906/helm-prometheus-operator-set-email-notifications-edit-secrets





```
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: alertmanager-config
  namespace: monitoring
  labels:
    alertmanagerConfig: "example"
spec:
  route:
    groupBy: ["alertname"]
    groupWait: 30s
    groupInterval: 5m
    repeatInterval: 12h
    receiver: "default_channel"
    routes:
      - matchers:
          - name: namespace
            value: xxx-.*
            regex: true
        receiver: "xxx_email"
  receivers:
    - name: "xxx_email"
      emailConfigs:
        - to: "xxx@gmail.com"
          from: "xxxr@gmail.com"
          smarthost: "smtp.gmail.com:587"
          authUsername: "xxx@gmail.com"
          authPassword:
            name: gmail-auth
            key: password
          authIdentity: "xxx@gmail.com"
```

