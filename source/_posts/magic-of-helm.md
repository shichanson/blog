---
title: "helm使用技巧篇"
author: "Mayer Shi"
tags: ["helm"]
categories: ["Container Cloud"]
date: 2019-01-03T20:23:50+08:00
draft: false
---

helm 的核心优点在于 charts 一次编写到处运行以及其版本跟踪的能力。本篇博文主要讲述 helm 在本地开发 charts 时的一些技巧，通过这些技巧可以大大增加 charts 的易用性以及扩展性。

<!--more-->

**针对 helm 篇的实践落地方案分为如下几个部分：**

> - helm 基础理论篇
> - helm 使用技巧篇
> - 基础中间件服务运维篇
> - 微服务应用版本管理篇
> - 基于 jenkins + helm 的 CICD 方案
> - Helm 实践趟坑篇
> - 基于 Helm Istio Jenkins 灰度发布实践方案

本篇博文是该系列博客中的第二篇文章**《helm 的使用技巧》**。社区以及官方文档大多提供的是针对单个服务的 charts 编写指导。对于有依赖关系的多个服务时也是通过子 charts 的方式来实现多服务组件部署，但是本质上还是一个 charts 一个服务。这对于动辄十来个组件的微服务架构应用来说，显然是不可取的方案。针对微服务应用场景，我们总结一些 helm 撰写 charts 的最佳实践。

### debug 调试妙用

**使用场景：**

**helm** 应用发布工具一般很少单独使用，在企业中的应用一般都是作为 DevOps 工具链中的一环。我们在做基础服务运维的时候一般都应该遵循一个基本原则“**infra as code**”。这样可以确保基础服务的可控和可追溯性。为了避免 charts 在实际运行中出错，我们可以在本地写 charts 的时候通过 debug 的功能，在不生成具体 release 的情况下检查 charts 是否存在语法错误和内容错误。

**实践总结：**

使用 Debug 功能的前提需要一个 k8s 集群且本地 helm 能够连接上该集群。最佳实践是尽可能确保本地 k8s 环境能与测试以及生产环境保持一致，这样才能确保 charts 的兼容性。笔者就遇到过这样的问题，charts 中包含了阿里云的日志服务 yaml 模板，在本地 minikube 集群上使用 helm 工具 debug 的时候总是报错的情况。最后将本地的 helm 工具直接连接阿里云上的 k8s 集群上，才顺利 debug。

```bash
helm install --dry-run --debug --name test tomcat
[debug] Created tunnel using local port: '51358'

[debug] SERVER: "127.0.0.1:51358"

[debug] Original chart version: ""
[debug] CHART PATH: /Users/mayershi/akd/charts/stable/tomcat

NAME:   test
REVISION: 1
RELEASED: Sat Mar 16 12:49:23 2019
CHART: tomcat-0.2.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
affinity: {}
deploy:
  directory: /usr/local/tomcat/webapps
image:
  pullPolicy: IfNotPresent
  pullSecrets: []
  tomcat:
    repository: tomcat
    tag: "7.0"
  webarchive:
    repository: ananwaresystems/webarchive
    tag: "1.0"
ingress:
  annotations: {}
  enabled: false
  hosts:
  - chart-example.local
  path: /
  tls: []
livenessProbe:
  initialDelaySeconds: 60
  path: /sample
  periodSeconds: 30
nodeSelector: {}
readinessProbe:
  failureThreshold: 6
  initialDelaySeconds: 60
  path: /sample
  periodSeconds: 30
replicaCount: 1
resources: {}
service:
  externalPort: 80
  internalPort: 8080
  name: http
  type: LoadBalancer
tolerations: []

HOOKS:
MANIFEST:

---
# Source: tomcat/templates/appsrv-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: test-tomcat
  labels:
    app: tomcat
    chart: tomcat-0.2.0
    release: test
    heritage: Tiller
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
      name: http
  selector:
    app: tomcat
    release: test
---
# Source: tomcat/templates/appsrv.yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: test-tomcat
  labels:
    app: tomcat
    chart: tomcat-0.2.0
    release: test
    heritage: Tiller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat
      release: test
  template:
    metadata:
      labels:
        app: tomcat
        release: test
    spec:
      volumes:
        - name: app-volume
          emptyDir: {}
      initContainers:
        - name: war
          image: ananwaresystems/webarchive:1.0
          imagePullPolicy: IfNotPresent
          command:
            - "sh"
            - "-c"
            - "cp /*.war /app"
          volumeMounts:
            - name: app-volume
              mountPath: /app
      containers:
        - name: tomcat
          image: tomcat:7.0
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: app-volume
              mountPath: /usr/local/tomcat/webapps
          ports:
            - containerPort: 8080
              hostPort: 8009
          livenessProbe:
            httpGet:
              path: /sample
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /sample
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 30
            failureThreshold: 6
          resources:
            {}
```

### 多组件利器数组

在社区以及一些其他同行的博客教程中一般都是单个服务单个 charts 的方式，但是这种对于采用了微服务架构的应用是有问题的。有很多的缺陷，比如： 不能对应用进行统一的版本管理；需要编写大量的 charts 效率不高。那么解决这个问就需要引入 helm 的控制结构。

**helm 循环控制结构：**

values.yaml 值文件部分示例内容。

```yaml
containers:
  - name: app1
    replicaCount: 2
    image:
      name: app1:1
      repository: dockerhub.com
      pullPolicy: Always

    service:
      type: ClusterIP
      externalPort: 8080
      internalPort: 8080
      healthUrl: /token
      managementPort: 8080

    container:
      spring: false
      limitmemory: 256Mi
      env: {}
  - name: app2
    replicaCount: 2
    image:
      name: app2:1
      repository: dockerhub.com
      pullPolicy: Always

    service:
      type: ClusterIP
      externalPort: 12180
      internalPort: 12180
      healthUrl: "/manage/status"
      managementPort: 12181

    container:
      memory: 2048Mi
      limitmemory: 2048Mi
      spring: true
      hpa:
        type: memory
        value: 80
      env:
        - name: ALI_LOGSTORE
          value: app2
```

deploy 和 service 模板文件内容：

```yaml
{{- range $index, $container := .Values.containers }} // 由于helm是golang开发的所以，对于控制结构来说，他的循环控制结构和golang保持一致。通过这个循环控制结构可以将value值文件中的containers值下面的数组给遍历出来生成相应的deploy 和 service 的k8s的资源。
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ $fullname }}-{{ $container.name }}
  labels:
    app: {{ $fullname }}-{{ $container.name }}
    chart: {{ $chartname }}
    release: {{ $root.Release.Name }}
    heritage: {{ $root.Release.Service }}
spec:
  replicas: {{ $container.replicaCount }}
  selector:
    matchLabels:
      app:  "{{ $fullname }}-{{ $container.name }}"
      release: "{{ $root.Release.Name }}"
  template:
    metadata:
      labels:
        app: "{{ $fullname }}-{{ $container.name }}"
        release: "{{ $root.Release.Name }}"
    spec:
      imagePullSecrets:
        - name: aliyun-registry-secret
      containers:
      - name:  {{ $fullname }}-{{ $container.name }}
        image: {{ $container.image.repository }}/{{ $container.image.name }}
        imagePullPolicy: {{ $container.image.pullPolicy }}
        ports:
        - containerPort: {{ $container.service.internalPort }}
        {{ if $container.service.healthUrl }}
        livenessProbe:
          httpGet:
            path: {{ $container.service.healthUrl }}
            port: {{ $container.service.managementPort }}
          initialDelaySeconds: 60
          timeoutSeconds: 10
          periodSeconds: 30
          successThreshold: 1
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: {{ $container.service.healthUrl }}
            port: {{ $container.service.managementPort }}
          initialDelaySeconds: 60
          timeoutSeconds: 10
          periodSeconds: 30
          successThreshold: 1
          failureThreshold: 5
        {{ end }}
        env:
          - name: aliyun_logs_image_tags
            value: docker-image={{ $container.image.repository }}/{{ $container.image.name }}
          {{ if $container.container.spring }}
          - name: JAVA_OPTIONS
            value: >-
              -XX:+UseG1GC
              -XX:+HeapDumpOnOutOfMemoryError
              -Duser.timezone=Asia/Hong_Kong
              -Dspring.profiles.active={{ $root.Values.container.spring.profile }}
          {{ end }}
{{ if $container.container.env }}
{{ toYaml $container.container.env | indent 10 }}
{{ end }}
        resources:
        {{ if or $container.container.cpu $container.container.memory }}
          requests:
          {{ if $container.container.cpu }}
            cpu: "{{ $container.container.cpu }}"
          {{ end }}
          {{ if $container.container.memory }}
            memory: "{{ $container.container.memory }}"
          {{ end }}
        {{ end }}
        {{ if or $container.container.limitcpu $container.container.limitmemory }}
          limits:
          {{ if $container.container.limitcpu }}
            cpu: "{{ $container.container.limitcpu }}"
          {{ end }}
          {{ if $container.container.limitmemory }}
            memory: "{{ $container.container.limitmemory }}"
          {{ end }}
        {{ end }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ $serviceprefix }}-{{ $container.name }}
  labels:
    app: {{ $name }}
    chart: {{ $chartname }}
    release: {{ $root.Release.Name }}
    heritage: {{ $root.Release.Service }}
spec:
  type: {{ $container.service.type }}
  ports:
    - port: {{ $container.service.externalPort }}
      targetPort: {{ $container.service.internalPort }}
      protocol: TCP
      name: {{ $fullname }}-{{ $container.name }}-http
  selector:
    app: {{ $fullname }}-{{ $container.name }}
    release: {{ $root.Release.Name }}
{{- end }}
```

将 values.yaml 值文件 和 deploy-server.yaml 模板文件通过 helm 渲染得出真正的 deploy 的 yaml 以及 service 的 yaml 文件。然后 k8s 接受到后，会生成相应的资源。

### 组件个性化开关

每个应用的配置以及运行状态是不一样的,比如: java 启动的参数，node 应用的启动环境变量等等，所以就涉及到应用个性化参数开关的问题，那么解决这个的方案就是 helm 的条件控制结构。
**helm 条件控制结构**
上面的实例文件中 containers 的数组的每个想内部存的是单个 deploy + service 的 value 值。从文件中可以看出差异部分。
第一个服务组件的值文件 spring 的值是 false。

```yaml
```

第二个服务组件的值文件 spring 的值是 true。

```yaml
container:
  memory: 2048Mi
  limitmemory: 2048Mi
  spring: true
  hpa:
    type: memory
    value: 80
  env:
    - name: ALI_LOGSTORE
      value: app2
```

deploy-service.yaml 文件中关于这块的文件描述是这样的。

```yaml
env:
  - name: aliyun_logs_image_tags
    value: docker-image={{ $container.image.repository }}/{{ $container.image.name }}
  {{ if $container.container.spring }}
  - name: JAVA_OPTIONS
    value: >-
      -XX:+UseG1GC
      -XX:+HeapDumpOnOutOfMemoryError
      -Duser.timezone=Asia/Hong_Kong
      -Dspring.profiles.active={{ $root.Values.container.spring.profile }}
  {{ end }}
```

运行结果就是当 spring 值是 true 的时候。env 的内容就会添加 Java 启动的环境变量参数。spring 值为 false 的时候就不会添加该环境变量。
