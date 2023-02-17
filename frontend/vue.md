### 与后端的API　proxy地址设定

- 使用后端的api service名,不使用地址
- 本地使用docker测试时,/etc/hosts中加入后端 svc name和ip的设置,不使用127.0.0.1



### node+nginx Dockerfile构建的两个选择

#### 本地npm/yarn构建生成dist目录,只在dockerfile中加入dist目录和nginx配置文件,构建nginx镜像

#### 使用多阶段构建Dockerfile,一个镜像用来构建静态文件,一个镜像用来构建nginx 

cd /home/qxu/PycharmProjects/tk-demo/frontend



```
cat Dockerfile 
#vue node build
FROM docker.io/library/node:16.19.0 as build-stage

WORKDIR /app
COPY package*.json ./

#在工作目录执行npm install 
RUN git config --global url."https://".insteadOf git://
RUN npm install -g npm@8.1.2
RUN npm install --registry=https://registry.npm.taobao.org

#将本本录下所有文件复制到目标目录
COPY . .
RUN npm run build:prod

#nginx image build
FROM docker.io/library/nginx:latest  as production-stage
MAINTAINER qxu
RUN rm -f /etc/nginx/conf.d/*.conf
COPY dzbird.conf /etc/nginx/conf.d/dzbird.conf

COPY --from=build-stage /app/dist/ /usr/share/nginx/html/
```

```
docker build . -t frontend:master 
```



### Dockerfile注意事项

- 与开发环境版本一致或相似 ,  go to https://hub.docker.com/_/node  to search 16.19.0

  ```
  node -v
  v16.13.1
  
  ```

  

- 不选-alpine, 缺少git或其他相关命令,构建会失败,　try this

  ```
  docker run -it docker.io/library/node:16.19.0-alpine /bin/sh
  
  docker run -it docker.io/library/node:16.19.0 /bin/bash
  ```

- 可以注释　RUN git config --global url."https://".insteadOf git:// 后尝试构建,应该会失败

- 使用国内的npm源　 npm install --registry=https://registry.npm.taobao.org

- 目录copy,不要使用/*,　使用下面的格式

  ```
  COPY --from=build-stage /app/dist/ /usr/share/nginx/html/
  COPY  /app/dist/ /usr/share/nginx/html/
  ```

- 优化构建,ＣＯＰＹ时排除本地的node_modules目录 or not

  ```
  cat .dockeringore
  node_modules
  ```

  





### nginx配置文件

两个主要的注意点:

- root的位置与Dockerfile中dist复制的位置一致
- proxy_pass使用后端的svc地址,不使用ip地址,不使用127.0.0.1

```
root@qxu frontend]# cat dzbird.conf 
server {
    listen       80 default_server;
    #listen       [::]:80 default_server;
    server_name  _;
    root         /usr/share/nginx/html/;

    location / {
        index  index.html index.htm;
	try_files $uri $uri/ @router;
    }

    location @router {
	rewrite ^.*$ /index.html last;
    }
    
    location /api {
	proxy_pass http://projectapi:8080;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }
    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}

```



deployemnt & svc & ingress

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: frontend
  replicas: 4
  template:
    metadata:
      labels:
        app: frontend
    spec:
      imagePullSecrets:
      - name: dockercred
      containers:
        - name: projectapi
          image: quay.io/qxu/frontend:master
          #image: quay.io/qxu/frontend:v2
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: frontend
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend
spec:
  ingressClassName: nginx 
  rules:
  - host: "frontend.user1.apps.taikang1.local"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: frontend
            port:
              number: 80
```



vue中的frontend更新后,使用新的pod,正在使用的用户,并不会无感知,用户的chrome中会报错

这是因为部分修改的js,css等文件会发生变更,这里要求用户强制刷新页面才行.

```
chunk-libs.90ad461e.js:46 ChunkLoadError: Loading chunk chunk-50da8ecc failed.
(missing: http://frontend.user1.apps.taikang1.local/static/js/chunk-50da8ecc.75a0a72e.js)
    at Function.i.e (http://frontend.user1.apps.taikang1.local/:1:3140)
    at component (http://frontend.user1.apps.taikang1.local/static/js/app.ceb2a8d6.js:1:80840)
    at http://frontend.user1.apps.taikang1.local/static/js/chunk-libs.90ad461e.js:46:15331
    at http://frontend.user1.apps.taikang1.local/static/js/chunk-libs.90ad461e.js:46:15581
    at Array.map (<anonymous>)
    at http://frontend.user1.apps.taikang1.local/static/js/chunk-libs.90ad461e.js:46:15557
    at Array.map (<anonymous>)
    at Rt (http://frontend.user1.apps.taikang1.local/static/js/chunk-libs.90ad461e.js:46:15507)
    at http://frontend.user1.apps.taikang1.local/static/js/chunk-libs.90ad461e.js:46:15018
    at d (http://frontend.user1.apps.taikang1.local/static/js/chunk-libs.90ad461e.js:46:18329)
```

不是所有的js, css文件都会发现变化,

**webpack.config.js**　中对构建后的文件名做了hash的处理

```
module.exports = {
  output: {
    chunkFilename: '[id].[hash].js',
    // https://reactjs.org/docs/cross-origin-errors.html
    crossOriginLoading: "anonymous",
    filename: '[name].[hash].js',
    path: path.resolve(__dirname, 'dist'),
```

可参考:

https://github.com/webpack/webpack/issues/7502

vue element admin的webpacket.config.js做了优化,在vue.config.js中

https://www.cnblogs.com/ypSharing/p/vue-webpack.html



```
module.exports = {
    configureWebpack: config => {
      if(process.env.NODE_ENV === "production") {
        config.output.filename = 'js/[name].[contenthash:8].min.js'
        config.output.chunkFilename = 'js/[name].[contenthash:8].min.js'
      } else {
        config.output.filename = 'js/[name].js'
        config.output.chunkFilename = 'js/[name].js';
      }
    }
}
```

也可以build目录中建立对应的env build文件,对所有的build env做webpacet的配置.



# vue 开发常用工具及配置五：hash 与缓存控制



https://cloud.tencent.com/developer/article/1573517



**hash** 

以前使用 JQuery 开发前端页面的时候，页面中引用的资源文件如js、css等，一般尾部加一个 *t=[时间戳]* 参数，用于防止修改不生效。现在工程化开发，使用 Webpack 编译，打包的资源文件路径里自动带有一串随机字符串，称为 hash：

```javascript
<link href=/static/css/chunk-vendors.d637be65.css rel=preload as=style>
```

复制

“d637be65”即是hash。每次 build 都会生成不同的 hash，所以每次编译部署，都不会有缓存问题。

这个 hash 是如何生成的？它的生成机制是什么？

**三种 hash**

Webpack 打包后的资源按大小分有三类，从小到大排列：

- module，即模块，每个引入的文件就是一个module，常言模块化，是开发中的物理最小代码单位
- chunk， N 个模块打包在一起形成的的一个文件（如果 chunk 有 split，则每个分开的文件都是一个独立的 chunk）
- bundle，一次工程编译打包的最终产物，有可能就是 chunk，也有可能包含多个chunk的综合体

这三类资源都可以生成 hash，粒度从低到高依次为：

- hash，根据每次编译的内容计算所得，不是针对每个具体文件的，每次编译都会有一个 hash
- chunkhash，入口级别的 hash，如果入口文件有改动，则从该入口打包引入的所有文件的hash都会变化，主要指同一个入口的js和css文件。
- contenthash，文件级别的 hash，只有文件的内容变了hash才会变

在 vue 项目中，启用的是哪一类 hash？

**在 vue.config.js 中配置 hash**

在vue.config.js配置文件中，与输出文件名有关的主要配置有：

```javascript
  outputDir: 'dist',
  assetsDir: 'static',
  filenameHashing:true,
  pages: {
    index: {
      // page 的入口
      entry: 'src/main.js',
      // 模板来源
      template: 'public/index.html',
      // 在 dist/index.html 的输出
      filename: 'index.html',
      // 当使用 title 选项时，
      // template 中的 title 标签需要是 <title><%= htmlWebpackPlugin.options.title %></title>
      title: 'Index Page',
      hash: true,
      // 在这个页面中包含的块，默认情况下会包含
      // 提取出来的通用 chunk 和 vendor chunk。
      chunks: ['chunk-vendors', 'chunk-common', 'index']
    }
  },
```

复制

这里的 pages.*.chunks 有什么含义？

这部分配置，其实会编译到webpack中的html-webpack-plugin的配置里。所以vue.config.js中的pages.chunks也就等同于html-webpack-plugin中的chunks。

再看一下 html-webpack-plugin 的 chunks，有什么含义。

chunks 选项的作用主要是针对多入口(entry)文件。当你有多个入口文件的时候，对应就会生成多个编译后的 js 文件。那么 chunks 选项就可以决定是否都使用这些生成的 js 文件。例如这个html-webpack-plugin配置：

```javascript
entry: {
    index: path.resolve(__dirname, './src/index.js'),
    index1: path.resolve(__dirname, './src/index1.js'),
    index2: path.resolve(__dirname, './src/index2.js')
}
...
plugins: [
    new HtmlWebpackPlugin({
        ...
        chunks: ['index','index2']
    })
]
```

复制

注意，而如果没有显式指定 chunks 选项，默认会全部引用。

通过编译发现，如果不修改图片或源码，生成的 hash 是不会变化的。如果修改了 vue 组件，部分文件的 hash，例如 css、js文件的，会变化，但不想干的图片不会变化。

所以，回到前面的问题，在 vue 项目中，启用的是哪一类 hash？答案当是 contenthash。

源码



vue cli filenamehashing配置:

https://segmentfault.com/a/1190000016216299



splitChunks 常用参数

name 打包的 chunks 的名字
test 匹配到的模块奖杯打进这个缓存组
chunks 代码块类型 必须三选一： “initial”（初始化） | “all”(默认就是 all) | “async”（动态加载）默认 Webpack 4 只会对按需加载的代码做分割。如果我们需要配置初始加载的代码也加入到代码分割中，可以设置为 ‘all’
priority ：缓存组打包的先后优先级，数值大的优先
minSize （默认是30000）形成一个新代码块最小的体积
minChunks （默认是1）在分割之前，这个代码块最小应该被引用的次数
maxInitialRequests （默认是3）一个入口最大的并行请求数
maxAsyncRequests（默认是5）按需加载时候最大的并行请求数
reuseExistingChunk 如果当前的 chunk 已被从 split 出来，那么将会直接复用这个 chunk 而不是重新创建一个
enforce 告诉 webpack 忽略 splitChunks.minSize, splitChunks.minChunks, splitChunks.maxAsyncRequests and splitChunks.maxInitialRequests，总是为这个缓存组创建 chunks





可以试下的solution,测试不可用,还是要强刷

https://github.com/vuejs/vue-cli/issues/5989

添加:

       config
            .output
            .filename('static/js/[name].js?_hash=[contenthash:8]')
            .chunkFilename('static/js/[name].js?_hash=[contenthash:8]')


```
config
      .when(process.env.NODE_ENV !== 'development',
        config => {
          config
            .plugin('ScriptExtHtmlWebpackPlugin')
            .after('html')
            .use('script-ext-html-webpack-plugin', [{
            // `runtime` must same as runtimeChunk name. default is `runtime`
              inline: /runtime\..*\.js$/
            }])
            .end()
          config
            .optimization.splitChunks({
              chunks: 'all',
              cacheGroups: {
                libs: {
                  name: 'chunk-libs',
                  test: /[\\/]node_modules[\\/]/,
                  priority: 10,
                  chunks: 'initial' // only package third parties that are initially dependent
                },
                elementUI: {
                  name: 'chunk-elementUI', // split elementUI into a single package
                  priority: 20, // the weight needs to be larger than libs and app or it will be packaged into libs or app
                  test: /[\\/]node_modules[\\/]_?element-ui(.*)/ // in order to adapt to cnpm
                },
                commons: {
                  name: 'chunk-commons',
                  test: resolve('src/components'), // can customize your rules
                  minChunks: 3, //  minimum common number
                  priority: 5,
                  reuseExistingChunk: true
                }
              }
            })
           
          // https:// webpack.js.org/configuration/optimization/#optimizationruntimechunk
          config.optimization.runtimeChunk('single')

          config
            .output
            .filename('static/js/[name].js?_hash=[contenthash:8]')
            .chunkFilename('static/js/[name].js?_hash=[contenthash:8]')
```





npm run build:prod 可以看到有下面的变化

```
entrypoint size limit: The following entrypoint(s) combined asset size exceeds the recommended limit (244 KiB). This can impact web performance.
Entrypoints:
  app (1.27 MiB)
      static/js/runtime.js?_hash=d4042038
      static/js/chunk-elementUI.js?_hash=6018f1a8
      static/css/chunk-libs.3dfb7769.css
      static/js/chunk-libs.js?_hash=90ad461e
      static/css/app.ca58c650.css
      static/js/app.js?_hash=b17e5d9c


```



for openshift istio config

frontend use v3 version, it add http 1.1 protocol to support istio

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: projectapi
spec:
  selector:
    matchLabels:
      app: projectapi
  replicas: 1
  template:
    metadata:
      labels:
        app: projectapi
      annotations:
        sidecar.istio.io/inject: 'true'
    spec:
      containers:
        - name: projectapi
          image: quay.io/qxu/projectapi:master
          #image: quay.io/qxu/projectapi:v2
          #image: hub.taikang1.local/vue/projectapi:master
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: projectapi
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: projectapi
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: frontend
  replicas: 4
  template:
    metadata:
      labels:
        app: frontend
      annotations:
        sidecar.istio.io/inject: 'true'
    spec:
      imagePullSecrets:
      - name: dockercred
      containers:
        - name: projectapi
          image: quay.io/qxu/frontend:v3
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    name:　http
  selector:
    app: frontend
  type: ClusterIP

```

注意,下面一定不要使用*的泛域名,不然会和bookinfo的冲突　

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata: 
  name: frontend-gw
spec:
  selector:
    #app: istio-ingressgateway
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "front.test.com"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: frontend-vs
spec:
  hosts:
  - 'front.test.com'
  gateways:
  - 'frontend-gw'
  http:
  - route:
    - destination:
        host: frontend


```



```
oc adm policy add-scc-to-user anyuid -z default
```

