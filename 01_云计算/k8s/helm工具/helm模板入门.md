---
title: helm模板入门
description: 
published: true
date: 2023-01-06T08:42:46.582Z
tags: k8s
editor: markdown
dateCreated: 2023-01-06T08:42:46.582Z
---

# helm charts 入门学习



## chart 模版

1、chart 文件结构

```bash
mychart/
  Chart.yaml
  values.yaml
  charts/
  templates/
  ...
```

- `templates/` 目录包括了模板文件。当Helm评估chart时，会通过模板渲染引擎将所有文件发送到`templates/`目录中。 然后收集模板的结果并发送给Kubernetes。

- `values.yaml` 文件也导入到了模板。这个文件包含了chart的 *默认值* 。这些值会在用户执行`helm install` 或 `helm upgrade`时被覆盖。
- `Chart.yaml` 文件包含了该chart的描述。你可以从模板中访问它。`charts/`目录 *可以* 包含其他的chart(称之为 *子chart*)。 指南稍后我们会看到当涉及模板渲染时这些是如何工作的。

### 创建第一个charts

```bash
$ helm create <mychart>
```



查看mychart/templates 目录,存在一些模版文件:



- `NOTES.txt`: chart的"帮助文本"。这会在你的用户执行`helm install`时展示给他们。
- `deployment.yaml`: 创建Kubernetes [工作负载](https://kubernetes.io/docs/user-guide/deployments/)的基本清单
- `service.yaml`: 为你的工作负载创建一个 [service终端](https://kubernetes.io/docs/user-guide/services/)基本清单。
- `_helpers.tpl`: 放置可以通过chart复用的模板辅助对象



```latex
templates/
├── NOTES.txt
├── _helpers.tpl
├── deployment.yaml
├── hpa.yaml
├── ingress.yaml
├── service.yaml
├── serviceaccount.yaml
└── tests
    └── test-connection.yaml
```



这些文件可以直接删除掉,但是在实际工作中这些模板会特别有用



### 第一个模板

第一个模板是创建一个ConfigMap.  Kubernetes中，配置映射只是用于存储配置数据的对象。其他组件，比如pod，可以访问配置映射中的数据。

因为配置映射是基础资源，对我们来说是很好的起点。

让我们以创建一个名为 `mychart/templates/configmap.yaml`的文件开始

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
```

**提示:** 模板名称不遵循严格的命名模式。但是建议以`.yaml`作为YAML文件的后缀，以`.tpl`作为helper文件的后缀。

上述YAML文件是一个简单的配置映射，构成了最小的必需字段。因为文件在 `mychart/templates/`目录中，它会通过模板引擎传递。

像这样将一个普通YAML文件放在`mychart/templates/`目录中是没问题的。当Helm读取这个模板时会按照原样传递给Kubernetes。

有了这个简单的模板，现在有一个可安装的chart了。现在安装如下：

```bash
$ helm install full-coral ./mychart

NAME: full-coral
LAST DEPLOYED: Fri Jan  6 16:22:06 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```



我们可以使用Helm检索版本并查看实际加载的模板。

```bash
$ helm get manifest full-coral


---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
```

`helm get manifest` 命令后跟一个发布名称(`full-coral`)然后打印出了所有已经上传到server的Kubernetes资源。 每个文件以`---`开头表示YAML文件的开头，然后是自动生成的注释行，表示哪个模板文件生成了这个YAML文档。

从这个地方开始，我们看到的YAML数据确实是`configmap.yaml`文件中的内容。

现在卸载发布： `helm uninstall full-coral`。

```bash
## 卸载charts
$ helm uninstall full-coral
```



### 添加一个简单的模板调用

将`name:`硬编码到一个资源中不是很好的方式。名称应该是唯一的。因此我们可能希望通过插入发布名称来生成名称字段。

**提示:** 由于DNS系统的限制，`name:`字段长度限制为63个字符。因此发布名称限制为53个字符。 Kubernetes 1.3及更早版本限制为24个字符 (名称长度是14个字符)。

对应改变一下`configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```

大的变化是`name:`字段的值，现在是`{{ .Release.Name }}-configmap`。

> 模板命令要括在 `{{` 和 `}}` 之间。

模板命令 `{{ .Release.Name }}` 将发布名称注入了模板。值作为一个 *命名空间对象* 传给了模板，用点(`.`)分隔每个命名空间的元素。

`Release`前面的点表示从作用域最顶层的命名空间开始（稍后会谈作用域）。这样`.Release.Name`就可解读为“通顶层命名空间开始查找 Release对象，然后在其中找Name对象”。

`Release`是一个Helm的内置对象。稍后会更深入地讨论。但现在足够说明它可以显示从库中赋值的发布名称。



```bash
$ helm install clunky-serval ./mychart

NAME: clunky-serval
LAST DEPLOYED: Fri Jan  6 16:34:17 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

$ helm get manifest clunky-serval

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: clunky-serval-configmap
data:
  myvalue: "Hello World"
```

可以运行`helm get manifest clunky-serval`查看生成的完整的YAML。

注意在kubernetes内的配置映射名称是 `clunky-serval-configmap`，而不是之前的 `mychart-configmap`。

由此我们已经看到了最基本的模板：YAML文件有嵌入在`{{` 和 `}}`之间的模板命令。下一部分，会深入了解模板， 但在这之前，有个快捷的技巧可以加快模板的构建速度：当你想测试模板渲染的内容但又不想安装任何实际应用时，可以使用`helm install --debug --dry-run goodly-guppy ./mychart`。这样不会安装应用(chart)到你的kubenetes集群中，只会渲染模板内容到控制台（用于测试）。渲染后的模板如下：



```bash
$ helm install --debug --dry-run goodly-guppy ./mychart

install.go:178: [debug] Original chart version: ""
install.go:195: [debug] CHART PATH: /home/cmzhu/workspace/mychart

NAME: goodly-guppy
LAST DEPLOYED: Fri Jan  6 16:36:20 2023
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
affinity: {}
autoscaling:
  enabled: false
  maxReplicas: 100
  minReplicas: 1
  targetCPUUtilizationPercentage: 80
fullnameOverride: ""
image:
  pullPolicy: IfNotPresent
  repository: nginx
  tag: ""
imagePullSecrets: []
ingress:
  annotations: {}
  className: ""
  enabled: false
  hosts:
  - host: chart-example.local
    paths:
    - path: /
      pathType: ImplementationSpecific
  tls: []
nameOverride: ""
nodeSelector: {}
podAnnotations: {}
podSecurityContext: {}
replicaCount: 1
resources: {}
securityContext: {}
service:
  port: 80
  type: ClusterIP
serviceAccount:
  annotations: {}
  create: true
  name: ""
tolerations: []

HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: goodly-guppy-configmap
data:
  myvalue: "Hello World"
```





使用`--dry-run`会让你变得更容易测试，但不能保证Kubernetes会接受你生成的模板。 最好不要仅仅因为`--dry-run`可以正常运行就觉得chart可以安装。

在 [Chart模板指南](https://helm.sh/zh/docs/chart_template_guide/)中，我们以这里定义的chart基本模板为例详细讨论Helm模板语言。 然后开始讨论内置对象。