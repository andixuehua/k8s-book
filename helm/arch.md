{% raw %}

https://github.com/IBM/helm101

lab参考



###  安装

https://github.com/helm/helm/releases



### template 语法

https://helm.sh/docs/chart_template_guide/

https://pkg.go.dev/text/template



Chart.yaml中中定义

chart name, version, app version,

```
annotations:
  category: Infrastructure
  licenses: Apache-2.0
apiVersion: v2
appVersion: 1.23.3
dependencies:
- name: common
  repository: https://charts.bitnami.com/bitnami
  tags:
  - bitnami-common
  version: 2.x.x
description: NGINX Open Source is a web server that can be also used as a reverse
  proxy, load balancer, and HTTP cache. Recommended for high-demanding sites due to
  its ability to provide faster content.
home: https://github.com/bitnami/charts/tree/main/bitnami/nginx
icon: https://bitnami.com/assets/stacks/nginx/img/nginx-stack-220x234.png
keywords:
- nginx
- http
- web
- www
- reverse proxy
maintainers:
- name: Bitnami
  url: https://github.com/bitnami/charts
name: nginx
sources:
- https://github.com/bitnami/containers/tree/main/bitnami/nginx
- https://www.nginx.org
version: 13.2.24

```



```
 helm list
NAME      	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
nginx-test	user1    	6       	2023-02-16 14:48:06.672446243 +0800 CST	deployed	nginx-13.2.24	1.23.3 
```



```
apiVersion: {{ include "common.capabilities.deployment.apiVersion" . }}
kind: Deployment
metadata:
  name: {{ include "common.names.fullname" . }}
  namespace: {{ include "common.names.namespace" . | quote }}
  labels: {{- include "common.labels.standard" . | nindent 4 }}
    {{- if .Values.commonLabels }}
    {{- include "common.tplvalues.render" ( dict "value" .Values.commonLabels "context" $ ) | nindent 4 }}
    {{- end }}
  {{- if .Values.commonAnnotations }}
  annotations: {{- include "common.tplvalues.render" ( dict "value" .Values.commonAnnotations "context" $ ) | nindent 4 }}
  {{- end }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  {{- if .Values.updateStrategy }}
  strategy: {{- toYaml .Values.updateStrategy | nindent 4 }}
  {{- end }}
  selector:
    matchLabels: {{- include "common.labels.matchLabels" . | nindent 6 }}
  template:
    metadata:
      labels: {{- include "common.labels.standard" . | nindent 8 }}
        {{- if .Values.podLabels }}
        {{- include "common.tplvalues.render" (dict "value" .Values.podLabels "context" $) | nindent 8 }}
        {{- end }}
      annotations:
        {{- if .Values.podAnnotations }}
        {{- include "common.tplvalues.render" ( dict "value" .Values.podAnnotations "context" $) | nindent 8 }}
        {{- end }}
        {{- if and .Values.metrics.enabled .Values.metrics.podAnnotations }}
        {{- include "common.tplvalues.render" ( dict "value" .Values.metrics.podAnnotations "context" $) | nindent 8 }}
        {{- end }}
        {{- if and .Values.serverBlock (not .Values.existingServerBlockConfigmap) }}
        checksum/server-block-configuration: {{ include (print $.Template.BasePath "/server-block-configmap.yaml") . | sha256sum }}
        {{- end }}
    spec:
      {{- include "nginx.imagePullSecrets" . | nindent 6 }}
      automountServiceAccountToken: {{ .Values.serviceAccount.automountServiceAccountToken }}
      shareProcessNamespace: {{ .Values.sidecarSingleProcessNamespace }}
      serviceAccountName: {{ template "nginx.serviceAccountName" . }}
      {{- if .Values.hostAliases }}
      hostAliases: {{- include "common.tplvalues.render" (dict "value" .Values.hostAliases "context" $) | nindent 8 }}
      {{- end }}
      {{- if .Values.affinity }}
      affinity: {{- include "common.tplvalues.render" (dict "value" .Values.affinity "context" $) | nindent 8 }}
      {{- else }}
      affinity:
        podAffinity: {{- include "common.affinities.pods" (dict "type" .Values.podAffinityPreset "context" $) | nindent 10 }}
        podAntiAffinity: {{- include "common.affinities.pods" (dict "type" .Values.podAntiAffinityPreset "context" $) | nindent 10 }}
        nodeAffinity: {{- include "common.affinities.nodes" (dict "type" .Values.nodeAffinityPreset.type "key" .Values.nodeAffinityPreset.key "values" .Values.nodeAffinityPreset.values) | nindent 10 }}
      {{- end }}
      hostNetwork: {{ .Values.hostNetwork }}
      hostIPC: {{ .Values.hostIPC }}
      {{- if .Values.priorityClassName }}
      priorityClassName: {{ .Values.priorityClassName | quote }}
      {{- end }}
      {{- if .Values.nodeSelector }}
      nodeSelector: {{- include "common.tplvalues.render" (dict "value" .Values.nodeSelector "context" $) | nindent 8 }}
      {{- end }}
      {{- if .Values.tolerations }}
      tolerations: {{- include "common.tplvalues.render" (dict "value" .Values.tolerations "context" $) | nindent 8 }}
      {{- end }}
      {{- if .Values.schedulerName }}
      schedulerName: {{ .Values.schedulerName | quote }}
      {{- end }}
      {{- if .Values.topologySpreadConstraints }}
      topologySpreadConstraints: {{- include "common.tplvalues.render" (dict "value" .Values.topologySpreadConstraints "context" .) | nindent 8 }}
      {{- end }}
      {{- if .Values.podSecurityContext.enabled }}
      securityContext: {{- omit .Values.podSecurityContext "enabled" | toYaml | nindent 8 }}
      {{- end }}
      {{- if .Values.terminationGracePeriodSeconds }}
      terminationGracePeriodSeconds: {{ .Values.terminationGracePeriodSeconds }}
      {{- end }}
      initContainers:
        {{- if .Values.cloneStaticSiteFromGit.enabled }}
        - name: git-clone-repository
          image: {{ include "nginx.cloneStaticSiteFromGit.image" . }}
          imagePullPolicy: {{ .Values.cloneStaticSiteFromGit.image.pullPolicy | quote }}
          {{- if .Values.containerSecurityContext.enabled }}
          securityContext: {{- omit .Values.containerSecurityContext "enabled" | toYaml | nindent 12 }}
          {{- end }}
          {{- if .Values.cloneStaticSiteFromGit.gitClone.command }}
          command: {{- include "common.tplvalues.render" (dict "value" .Values.cloneStaticSiteFromGit.gitClone.command "context" $) | nindent 12 }}
          {{- else }}
          command:
            - /bin/bash
            - -ec
            - |
              [[ -f "/opt/bitnami/scripts/git/entrypoint.sh" ]] && source "/opt/bitnami/scripts/git/entrypoint.sh"
              git clone {{ .Values.cloneStaticSiteFromGit.repository }} --branch {{ .Values.cloneStaticSiteFromGit.branch }} /app
          {{- end }}
          {{- if .Values.cloneStaticSiteFromGit.gitClone.args }}
          args: {{- include "common.tplvalues.render" (dict "value" .Values.cloneStaticSiteFromGit.gitClone.args "context" $) | nindent 12 }}
          {{- end }}
          volumeMounts:
            - name: staticsite
              mountPath: /app
            {{- if .Values.cloneStaticSiteFromGit.extraVolumeMounts }}
            {{- include "common.tplvalues.render" (dict "value" .Values.cloneStaticSiteFromGit.extraVolumeMounts "context" $) | nindent 12 }}
            {{- end }}
          {{- if .Values.cloneStaticSiteFromGit.extraEnvVars }}
          env: {{- include "common.tplvalues.render" (dict "value" .Values.cloneStaticSiteFromGit.extraEnvVars "context" $) | nindent 12 }}
          {{- end }}
        {{- end }}
        {{- if .Values.initContainers }}
        {{- include "common.tplvalues.render" (dict "value" .Values.initContainers "context" $) | nindent 8 }}
        {{- end }}
      containers:
        {{- if .Values.cloneStaticSiteFromGit.enabled }}
        - name: git-repo-syncer
          image: {{ include "nginx.cloneStaticSiteFromGit.image" . }}
          imagePullPolicy: {{ .Values.cloneStaticSiteFromGit.image.pullPolicy | quote }}
          {{- if .Values.containerSecurityContext.enabled }}
          securityContext: {{- omit .Values.containerSecurityContext "enabled" | toYaml | nindent 12 }}
          {{- end }}
          {{- if .Values.cloneStaticSiteFromGit.gitSync.command }}
          command: {{- include "common.tplvalues.render" (dict "value" .Values.cloneStaticSiteFromGit.gitSync.command "context" $) | nindent 12 }}
          {{- else }}
          command:
            - /bin/bash
            - -ec
            - |
              [[ -f "/opt/bitnami/scripts/git/entrypoint.sh" ]] && source "/opt/bitnami/scripts/git/entrypoint.sh"
              while true; do
                  cd /app && git pull origin {{ .Values.cloneStaticSiteFromGit.branch }}
                  sleep {{ .Values.cloneStaticSiteFromGit.interval }}
              done
          {{- end }}
          {{- if .Values.cloneStaticSiteFromGit.gitSync.args }}
          args: {{- include "common.tplvalues.render" (dict "value" .Values.cloneStaticSiteFromGit.gitSync.args "context" $) | nindent 12 }}
          {{- end }}
          volumeMounts:
            - name: staticsite
              mountPath: /app
            {{- if .Values.cloneStaticSiteFromGit.extraVolumeMounts }}
            {{- include "common.tplvalues.render" (dict "value" .Values.cloneStaticSiteFromGit.extraVolumeMounts "context" $) | nindent 12 }}
            {{- end }}
          {{- if .Values.cloneStaticSiteFromGit.extraEnvVars }}
          env: {{- include "common.tplvalues.render" (dict "value" .Values.cloneStaticSiteFromGit.extraEnvVars "context" $) | nindent 12 }}
          {{- end }}
        {{- end }}
        - name: nginx
          image: {{ include "nginx.image" . }}
          imagePullPolicy: {{ .Values.image.pullPolicy | quote }}
          {{- if .Values.containerSecurityContext.enabled }}
          securityContext: {{- omit .Values.containerSecurityContext "enabled" | toYaml | nindent 12 }}
          {{- end }}
          {{- if .Values.diagnosticMode.enabled }}
          command: {{- include "common.tplvalues.render" (dict "value" .Values.diagnosticMode.command "context" $) | nindent 12 }}
          {{- else if .Values.command }}
          command: {{- include "common.tplvalues.render" (dict "value" .Values.command "context" $) | nindent 12 }}
          {{- end }}
          {{- if .Values.diagnosticMode.enabled }}
          args: {{- include "common.tplvalues.render" (dict "value" .Values.diagnosticMode.args "context" $) | nindent 12 }}
          {{- else if .Values.args }}
          args: {{- include "common.tplvalues.render" (dict "value" .Values.args "context" $) | nindent 12 }}
          {{- end }}
          {{- if .Values.lifecycleHooks }}
          lifecycle: {{- include "common.tplvalues.render" (dict "value" .Values.lifecycleHooks "context" $) | nindent 12 }}
          {{- end }}
          env:
            - name: BITNAMI_DEBUG
              value: {{ ternary "true" "false" .Values.image.debug | quote }}
            - name: NGINX_HTTP_PORT_NUMBER
              value: {{ .Values.containerPorts.http | quote }}
            {{- if .Values.containerPorts.https }}
            - name: NGINX_HTTPS_PORT_NUMBER
              value: {{ .Values.containerPorts.https | quote }}
            {{- end }}
            {{- if .Values.extraEnvVars }}
            {{- include "common.tplvalues.render" (dict "value" .Values.extraEnvVars "context" $) | nindent 12 }}
            {{- end }}
          envFrom:
            {{- if .Values.extraEnvVarsCM }}
            - configMapRef:
                name: {{ include "common.tplvalues.render" (dict "value" .Values.extraEnvVarsCM "context" $) }}
            {{- end }}
            {{- if .Values.extraEnvVarsSecret }}
            - secretRef:
                name: {{ include "common.tplvalues.render" (dict "value" .Values.extraEnvVarsSecret "context" $) }}
            {{- end }}
          ports:
            - name: http
              containerPort: {{ .Values.containerPorts.http }}
            {{- if .Values.containerPorts.https }}
            - name: https
              containerPort: {{ .Values.containerPorts.https }}
            {{- end }}
            {{- if .Values.extraContainerPorts }}
            {{- include "common.tplvalues.render" (dict "value" .Values.extraContainerPorts "context" $) | nindent 12 }}
            {{- end }}
          {{- if not .Values.diagnosticMode.enabled }}
          {{- if .Values.customLivenessProbe }}
          livenessProbe: {{- include "common.tplvalues.render" (dict "value" .Values.customLivenessProbe "context" $) | nindent 12 }}
          {{- else if .Values.livenessProbe.enabled }}
          livenessProbe: {{- include "common.tplvalues.render" (dict "value" (omit .Values.livenessProbe "enabled") "context" $) | nindent 12 }}
            tcpSocket:
              port: http
          {{- end }}
          {{- if .Values.customReadinessProbe }}
          readinessProbe: {{- include "common.tplvalues.render" (dict "value" .Values.customReadinessProbe "context" $) | nindent 12 }}
          {{- else if .Values.readinessProbe.enabled }}
          readinessProbe: {{- include "common.tplvalues.render" (dict "value" (omit .Values.readinessProbe "enabled") "context" $) | nindent 12 }}
            tcpSocket:
              port: http
          {{- end }}
          {{- if .Values.customStartupProbe }}
          startupProbe: {{- include "common.tplvalues.render" (dict "value" .Values.customStartupProbe "context" $) | nindent 12 }}
          {{- else if .Values.startupProbe.enabled }}
          startupProbe: {{- include "common.tplvalues.render" (dict "value" (omit .Values.startupProbe "enabled") "context" $) | nindent 12 }}
            tcpSocket:
              port: http
          {{- end }}
          {{- end }}
          {{- if .Values.resources }}
          resources: {{- toYaml .Values.resources | nindent 12 }}
          {{- end }}
          volumeMounts:
            {{- if or .Values.serverBlock .Values.existingServerBlockConfigmap }}
            - name: nginx-server-block
              mountPath: /opt/bitnami/nginx/conf/server_blocks
            {{- end }}
            {{- if (include "nginx.useStaticSite" .) }}
            - name: staticsite
              mountPath: /app
            {{- end }}
            {{- if .Values.extraVolumeMounts }}
            {{- include "common.tplvalues.render" ( dict "value" .Values.extraVolumeMounts "context" $) | nindent 12 }}
            {{- end }}
        {{- if .Values.metrics.enabled }}
        - name: metrics
          image: {{ include "nginx.metrics.image" . }}
          imagePullPolicy: {{ .Values.metrics.image.pullPolicy | quote }}
          {{- if .Values.metrics.securityContext.enabled }}
          securityContext: {{- omit .Values.metrics.securityContext "enabled" | toYaml | nindent 12 }}
          {{- end }}
          command: ['/usr/bin/exporter', '-nginx.scrape-uri', 'http://127.0.0.1:{{- default .Values.containerPorts.http .Values.metrics.port }}/status']
          ports:
            - name: metrics
              containerPort: 9113
          livenessProbe:
            httpGet:
              path: /metrics
              port: metrics
            initialDelaySeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /metrics
              port: metrics
            initialDelaySeconds: 5
            timeoutSeconds: 1
          {{- if .Values.metrics.resources }}
          resources: {{- toYaml .Values.metrics.resources | nindent 12 }}
          {{- end }}
        {{- end }}
        {{- if .Values.sidecars }}
        {{- include "common.tplvalues.render" ( dict "value" .Values.sidecars "context" $) | nindent 8 }}
        {{- end }}
      volumes:
        {{- if or .Values.serverBlock .Values.existingServerBlockConfigmap }}
        - name: nginx-server-block
          configMap:
            name: {{ include "nginx.serverBlockConfigmapName" . }}
        {{- end }}
        {{- if (include "nginx.useStaticSite" .) }}
        - name: staticsite
          {{- include "nginx.staticSiteVolume" . | nindent 10 }}
        {{- end }}
        {{- if .Values.extraVolumes }}
        {{- include "common.tplvalues.render" ( dict "value" .Values.extraVolumes "context" $) | nindent 8 }}
        {{- end }}

```





#### include

include "common.names.fullname" .

对应:

charts/common/templates/_names.tpl 中的fullname



include with dict usage

When using the `include` function, you can pass it a custom object tree built from the current context by using the `dict` function:

https://v2.helm.sh/docs/charts_tips_and_tricks/

https://itnext.io/reference-other-values-in-helm-chart-values-file-19d44d9276c7

```
{{- include "common.tplvalues.render" (dict "value" .Values.extraContainerPorts "context" $) | nindent 12 }}

MyApiOneUrl: {{ include "common.tplvalues.render" ( dict "value" .Values.apiOneUrl "context" .) }}
```



#### .Values	


  {{ .Values.image.repository }}

其中Values是Helm的内置对象，此处“.Values”可理解为工程内的values.yaml文件，image.repository表示访问其中image片段下的repository值。



#### -

-”表示去除表达式之前的空白/空行。 `{{-` (with the dash and space added) indicates that whitespace should be chomped left, while `-}}`

 {{- if eq .Values.favorite.drink "coffee" }}

通过使用在模板标识`{{`后面添加破折号和空格`{{-`来表示将左边空白删除，而在`}}`前面添加一个空格和破折号`-}}`表示应该删除右边的空格，另外需要注意的是**换行符也是空格！**

https://helm.sh/docs/chart_template_guide/control_structures/



#### .

```none
{{- $files := .Files }}        {{/* Get "Files" from . */}}
{{ . }}                        {{/* Write . as a value */}}
{{ include "common.names.fullname" . }}  {{/* Pass . as the template parameter */}},代表使用当前目录下的common.names.fullname
```

https://stackoverflow.com/questions/62472224/what-is-and-what-use-cases-have-the-dot-in-helm-charts



#### nindent

 {{- toYaml .Values.metrics.resources | nindent 12 }}

即表达式中 | nindent 12部分，nindent 12是函数作用是在内容前面增加12个空格





## 流程控制

- `if`/ `else`用于创建条件块，以下值的内容，被认为`flase`
  - 布尔值false
  - 数字零
  - 一个空字符串
  - 一个`nil`（空或空）
  - 一个空的集合（`map`，`slice`，`tuple`，`dict`，`array`）

- `with` 指定范围

  ```yaml
  {{ with PIPELINE }}
    # restricted scope
  {{ end }}
  123
  ```

- `range`，提供“针对每个”样式的循环



##### if/else的基本结构

```fallback
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

```yaml
 {{ if eq .Values.favorite.drink "coffee" }}
    mug: "true"
  {{ end }}
```



##### with使用示例

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
  
   
 #以下使用会出现错误，Release.Name不在with的作用范围，在受限范围内，将无法使用从父范围访问其他对象
 {{- with .Values.favorite }}
 drink: {{ .drink | default "tea" | quote }}
 food: {{ .food | upper | quote }}
 release: {{ .Release.Name }}
 {{- end }}

```



##### range用法示例

```
cat values.yaml

favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions

==========================================================================
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}
    
=============================================================================
# Source: my-nginx/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aaaa-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  toppings: |-
    - "Mushrooms"
    - "Cheese"
    - "Peppers"
    - "Onions"

```



#### 模板函数列表

https://helm.sh/zh/docs/chart_template_guide/function_list/



Dict　

Root context (`$`) is a dictionary that contains all [built-in objects](https://helm.sh/docs/chart_template_guide/builtin_objects/), including the `Values` object. The current context (`.`) points to the root context by default, but the user can change which variable the current context points to.

https://blog.palark.com/advanced-helm-templating/

```
{{ include "common.tplvalues.render" (dict "value" .Values.extraEnvVarsCM "context" $) }}
```

$代表将当前的contex传给 "context"


{% endraw %}





使用dry-run调试:

```
helm install nginx-test -f values.yaml --dry-run .
NAME: nginx-test
LAST DEPLOYED: Thu Feb 16 15:10:51 2023
NAMESPACE: user1
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: nginx/templates/svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-test
  namespace: "user1"
  labels:
    app.kubernetes.io/name: nginx
    helm.sh/chart: nginx-13.2.24
    app.kubernetes.io/instance: nginx-test
    app.kubernetes.io/managed-by: Helm
  annotations:
spec:
  type: ClusterIP
  sessionAffinity: None
  ports:
    - name: http
      port: 80
      targetPort: http
  selector:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/instance: nginx-test
---
# Source: nginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  namespace: "user1"
  labels:
    app.kubernetes.io/name: nginx
    helm.sh/chart: nginx-13.2.24
    app.kubernetes.io/instance: nginx-test
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  strategy:
    rollingUpdate: {}
    type: RollingUpdate
  selector:
    matchLabels:
      app.kubernetes.io/name: nginx
      app.kubernetes.io/instance: nginx-test
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nginx
        helm.sh/chart: nginx-13.2.24
        app.kubernetes.io/instance: nginx-test
        app.kubernetes.io/managed-by: Helm
      annotations:
    spec:
      
      automountServiceAccountToken: false
      shareProcessNamespace: false
      serviceAccountName: default
      affinity:
        podAffinity:
          
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app.kubernetes.io/name: nginx
                    app.kubernetes.io/instance: nginx-test
                topologyKey: kubernetes.io/hostname
              weight: 1
        nodeAffinity:
          
      hostNetwork: false
      hostIPC: false
      initContainers:
      containers:
        - name: nginx
          image: docker.io/bitnami/nginx:1.23.3-debian-11-r17
          imagePullPolicy: "IfNotPresent"
          env:
            - name: BITNAMI_DEBUG
              value: "false"
            - name: NGINX_HTTP_PORT_NUMBER
              value: "8080"
          envFrom:
          ports:
            - name: http
              containerPort: 8080
          livenessProbe:
            failureThreshold: 6
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
            tcpSocket:
              port: http
          readinessProbe:
            failureThreshold: 3
            initialDelaySeconds: 5
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 3
            tcpSocket:
              port: http
          resources:
            limits: {}
            requests: {}
          volumeMounts:
      volumes:

```



使用debug显示　变量

```
helm install nginx-test -f values.yaml --dry-run --debug .
install.go:192: [debug] Original chart version: ""
install.go:209: [debug] CHART PATH: /tmp/helpm-dir/nginx

NAME: nginx-test
LAST DEPLOYED: Thu Feb 16 15:12:16 2023
NAMESPACE: user1
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
affinity: {}
args: []
autoscaling:
  enabled: false
  maxReplicas: ""
  minReplicas: ""
  targetCPU: ""
  targetMemory: ""
cloneStaticSiteFromGit:
  branch: ""
  enabled: false
  extraEnvVars: []
  extraVolumeMounts: []
  gitClone:
    args: []
    command: []
  gitSync:
    args: []
    command: []
  image:
    digest: ""
    pullPolicy: IfNotPresent
    pullSecrets: []
    registry: docker.io
    repository: bitnami/git
    tag: 2.39.1-debian-11-r2
  interval: 60
  repository: ""
clusterDomain: cluster.local
command: []
commonAnnotations: {}
commonLabels: {}
containerPorts:
  http: 8080
  https: ""
containerSecurityContext:
  enabled: false
  runAsNonRoot: true
  runAsUser: 1001
customLivenessProbe: {}
customReadinessProbe: {}
customStartupProbe: {}
diagnosticMode:
  args:
  - infinity
  command:
  - sleep
  enabled: false
existingServerBlockConfigmap: ""
extraContainerPorts: []
extraDeploy: []
extraEnvVars: []
extraEnvVarsCM: ""
extraEnvVarsSecret: ""
extraVolumeMounts: []
extraVolumes: []
fullnameOverride: ""
global:
  imagePullSecrets: []
  imageRegistry: ""
healthIngress:
  annotations: {}
  enabled: false
  extraHosts: []
  extraPaths: []
  extraRules: []
  extraTls: []
  hostname: example.local
  ingressClassName: ""
  path: /
  pathType: ImplementationSpecific
  secrets: []
  selfSigned: false
  tls: false
hostAliases: []
hostIPC: false
hostNetwork: false
image:
  debug: false
  digest: ""
  pullPolicy: IfNotPresent
  pullSecrets: []
  registry: docker.io
  repository: bitnami/nginx
  tag: 1.23.3-debian-11-r17
ingress:
  annotations: {}
  apiVersion: ""
  enabled: false
  extraHosts: []
  extraPaths: []
  extraRules: []
  extraTls: []
  hostname: nginx.local
  ingressClassName: ""
  path: /
  pathType: ImplementationSpecific
  secrets: []
  selfSigned: false
  tls: false
initContainers: []
kubeVersion: ""
lifecycleHooks: {}
livenessProbe:
  enabled: true
  failureThreshold: 6
  initialDelaySeconds: 30
  periodSeconds: 10
  successThreshold: 1
  timeoutSeconds: 5
metrics:
  enabled: false
  image:
    digest: ""
    pullPolicy: IfNotPresent
    pullSecrets: []
    registry: docker.io
    repository: bitnami/nginx-exporter
    tag: 0.11.0-debian-11-r44
  podAnnotations: {}
  port: ""
  prometheusRule:
    additionalLabels: {}
    enabled: false
    namespace: ""
    rules: []
  resources:
    limits: {}
    requests: {}
  securityContext:
    enabled: false
    runAsUser: 1001
  service:
    annotations:
      prometheus.io/port: '{{ .Values.metrics.service.port }}'
      prometheus.io/scrape: "true"
    port: 9113
  serviceMonitor:
    enabled: false
    honorLabels: false
    interval: ""
    jobLabel: ""
    labels: {}
    metricRelabelings: []
    namespace: ""
    relabelings: []
    scrapeTimeout: ""
    selector: {}
nameOverride: ""
namespaceOverride: ""
nodeAffinityPreset:
  key: ""
  type: ""
  values: []
nodeSelector: {}
pdb:
  create: false
  maxUnavailable: 0
  minAvailable: 1
podAffinityPreset: ""
podAnnotations: {}
podAntiAffinityPreset: soft
podLabels: {}
podSecurityContext:
  enabled: false
  fsGroup: 1001
  sysctls: []
priorityClassName: ""
readinessProbe:
  enabled: true
  failureThreshold: 3
  initialDelaySeconds: 5
  periodSeconds: 5
  successThreshold: 1
  timeoutSeconds: 3
replicaCount: 1
resources:
  limits: {}
  requests: {}
schedulerName: ""
serverBlock: ""
service:
  annotations: {}
  clusterIP: ""
  externalTrafficPolicy: Cluster
  extraPorts: []
  loadBalancerIP: ""
  loadBalancerSourceRanges: []
  nodePorts:
    http: ""
    https: ""
  ports:
    http: 80
    https: 443
  sessionAffinity: None
  sessionAffinityConfig: {}
  targetPort:
    http: http
    https: https
  type: ClusterIP
serviceAccount:
  annotations: {}
  automountServiceAccountToken: false
  create: false
  name: ""
sidecarSingleProcessNamespace: false
sidecars: []
startupProbe:
  enabled: false
  failureThreshold: 6
  initialDelaySeconds: 30
  periodSeconds: 10
  successThreshold: 1
  timeoutSeconds: 5
staticSiteConfigmap: ""
staticSitePVC: ""
terminationGracePeriodSeconds: ""
tolerations: []
topologySpreadConstraints: []
updateStrategy:
  rollingUpdate: {}
  type: RollingUpdate

COMPUTED VALUES:
affinity: {}
args: []
autoscaling:
  enabled: false
  maxReplicas: ""
  minReplicas: ""
  targetCPU: ""
  targetMemory: ""
cloneStaticSiteFromGit:
  branch: ""
  enabled: false
  extraEnvVars: []
  extraVolumeMounts: []
  gitClone:
    args: []
    command: []
  gitSync:
    args: []
    command: []
  image:
    digest: ""
    pullPolicy: IfNotPresent
    pullSecrets: []
    registry: docker.io
    repository: bitnami/git
    tag: 2.39.1-debian-11-r2
  interval: 60
  repository: ""
clusterDomain: cluster.local
command: []
common:
  exampleValue: common-chart
  global:
    imagePullSecrets: []
    imageRegistry: ""
commonAnnotations: {}
commonLabels: {}
containerPorts:
  http: 8080
  https: ""
containerSecurityContext:
  enabled: false
  runAsNonRoot: true
  runAsUser: 1001
customLivenessProbe: {}
customReadinessProbe: {}
customStartupProbe: {}
diagnosticMode:
  args:
  - infinity
  command:
  - sleep
  enabled: false
existingServerBlockConfigmap: ""
extraContainerPorts: []
extraDeploy: []
extraEnvVars: []
extraEnvVarsCM: ""
extraEnvVarsSecret: ""
extraVolumeMounts: []
extraVolumes: []
fullnameOverride: ""
global:
  imagePullSecrets: []
  imageRegistry: ""
healthIngress:
  annotations: {}
  enabled: false
  extraHosts: []
  extraPaths: []
  extraRules: []
  extraTls: []
  hostname: example.local
  ingressClassName: ""
  path: /
  pathType: ImplementationSpecific
  secrets: []
  selfSigned: false
  tls: false
hostAliases: []
hostIPC: false
hostNetwork: false
image:
  debug: false
  digest: ""
  pullPolicy: IfNotPresent
  pullSecrets: []
  registry: docker.io
  repository: bitnami/nginx
  tag: 1.23.3-debian-11-r17
ingress:
  annotations: {}
  apiVersion: ""
  enabled: false
  extraHosts: []
  extraPaths: []
  extraRules: []
  extraTls: []
  hostname: nginx.local
  ingressClassName: ""
  path: /
  pathType: ImplementationSpecific
  secrets: []
  selfSigned: false
  tls: false
initContainers: []
kubeVersion: ""
lifecycleHooks: {}
livenessProbe:
  enabled: true
  failureThreshold: 6
  initialDelaySeconds: 30
  periodSeconds: 10
  successThreshold: 1
  timeoutSeconds: 5
metrics:
  enabled: false
  image:
    digest: ""
    pullPolicy: IfNotPresent
    pullSecrets: []
    registry: docker.io
    repository: bitnami/nginx-exporter
    tag: 0.11.0-debian-11-r44
  podAnnotations: {}
  port: ""
  prometheusRule:
    additionalLabels: {}
    enabled: false
    namespace: ""
    rules: []
  resources:
    limits: {}
    requests: {}
  securityContext:
    enabled: false
    runAsUser: 1001
  service:
    annotations:
      prometheus.io/port: '{{ .Values.metrics.service.port }}'
      prometheus.io/scrape: "true"
    port: 9113
  serviceMonitor:
    enabled: false
    honorLabels: false
    interval: ""
    jobLabel: ""
    labels: {}
    metricRelabelings: []
    namespace: ""
    relabelings: []
    scrapeTimeout: ""
    selector: {}
nameOverride: ""
namespaceOverride: ""
nodeAffinityPreset:
  key: ""
  type: ""
  values: []
nodeSelector: {}
pdb:
  create: false
  maxUnavailable: 0
  minAvailable: 1
podAffinityPreset: ""
podAnnotations: {}
podAntiAffinityPreset: soft
podLabels: {}
podSecurityContext:
  enabled: false
  fsGroup: 1001
  sysctls: []
priorityClassName: ""
readinessProbe:
  enabled: true
  failureThreshold: 3
  initialDelaySeconds: 5
  periodSeconds: 5
  successThreshold: 1
  timeoutSeconds: 3
replicaCount: 1
resources:
  limits: {}
  requests: {}
schedulerName: ""
serverBlock: ""
service:
  annotations: {}
  clusterIP: ""
  externalTrafficPolicy: Cluster
  extraPorts: []
  loadBalancerIP: ""
  loadBalancerSourceRanges: []
  nodePorts:
    http: ""
    https: ""
  ports:
    http: 80
    https: 443
  sessionAffinity: None
  sessionAffinityConfig: {}
  targetPort:
    http: http
    https: https
  type: ClusterIP
serviceAccount:
  annotations: {}
  automountServiceAccountToken: false
  create: false
  name: ""
sidecarSingleProcessNamespace: false
sidecars: []
startupProbe:
  enabled: false
  failureThreshold: 6
  initialDelaySeconds: 30
  periodSeconds: 10
  successThreshold: 1
  timeoutSeconds: 5
staticSiteConfigmap: ""
staticSitePVC: ""
terminationGracePeriodSeconds: ""
tolerations: []
topologySpreadConstraints: []
updateStrategy:
  rollingUpdate: {}
  type: RollingUpdate

HOOKS:
MANIFEST:
---
# Source: nginx/templates/svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-test
  namespace: "user1"
  labels:
    app.kubernetes.io/name: nginx
    helm.sh/chart: nginx-13.2.24
    app.kubernetes.io/instance: nginx-test
    app.kubernetes.io/managed-by: Helm
  annotations:
spec:
  type: ClusterIP
  sessionAffinity: None
  ports:
    - name: http
      port: 80
      targetPort: http
  selector:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/instance: nginx-test
---
# Source: nginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  namespace: "user1"
  labels:
    app.kubernetes.io/name: nginx
    helm.sh/chart: nginx-13.2.24
    app.kubernetes.io/instance: nginx-test
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  strategy:
    rollingUpdate: {}
    type: RollingUpdate
  selector:
    matchLabels:
      app.kubernetes.io/name: nginx
      app.kubernetes.io/instance: nginx-test
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nginx
        helm.sh/chart: nginx-13.2.24
        app.kubernetes.io/instance: nginx-test
        app.kubernetes.io/managed-by: Helm
      annotations:
    spec:
      
      automountServiceAccountToken: false
      shareProcessNamespace: false
      serviceAccountName: default
      affinity:
        podAffinity:
          
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app.kubernetes.io/name: nginx
                    app.kubernetes.io/instance: nginx-test
                topologyKey: kubernetes.io/hostname
              weight: 1
        nodeAffinity:
          
      hostNetwork: false
      hostIPC: false
      initContainers:
      containers:
        - name: nginx
          image: docker.io/bitnami/nginx:1.23.3-debian-11-r17
          imagePullPolicy: "IfNotPresent"
          env:
            - name: BITNAMI_DEBUG
              value: "false"
            - name: NGINX_HTTP_PORT_NUMBER
              value: "8080"
          envFrom:
          ports:
            - name: http
              containerPort: 8080
          livenessProbe:
            failureThreshold: 6
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
            tcpSocket:
              port: http
          readinessProbe:
            failureThreshold: 3
            initialDelaySeconds: 5
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 3
            tcpSocket:
              port: http
          resources:
            limits: {}
            requests: {}
          volumeMounts:
      volumes:

NOTES:
CHART NAME: nginx
CHART VERSION: 13.2.24
APP VERSION: 1.23.3

** Please be patient while the chart is being deployed **
NGINX can be accessed through the following DNS name from within your cluster:

    nginx-test.user1.svc.cluster.local (port 80)

To access NGINX from outside the cluster, follow the steps below:

1. Get the NGINX URL by running these commands:

    export SERVICE_PORT=$(kubectl get --namespace user1 -o jsonpath="{.spec.ports[0].port}" services nginx-test)
    kubectl port-forward --namespace user1 svc/nginx-test ${SERVICE_PORT}:${SERVICE_PORT} &
    echo "http://127.0.0.1:${SERVICE_PORT}"

```

