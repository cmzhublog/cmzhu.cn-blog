## Service

Kubernetes 中 Service 是 将运行在一个或一组 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 上的网络应用程序公开为网络服务的方法。

Kubernetes 中 Service 的一个关键目标是让你无需修改现有应用以使用某种不熟悉的服务发现机制。 你可以在 Pod 集合中运行代码，无论该代码是为云原生环境设计的， 还是被容器化的老应用。 你可以使用 Service 让一组 Pod 可在网络上访问，这样客户端就能与之交互。

如果你使用 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 来运行你的应用， Deployment 可以动态地创建和销毁 Pod。 在任何时刻，你都不知道有多少个这样的 Pod 正在工作以及它们健康与否； 你可能甚至不知道如何辨别健康的 Pod。 Kubernetes [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 的创建和销毁是为了匹配集群的预期状态。 Pod 是临时资源（你不应该期待单个 Pod 既可靠又耐用）。

每个 Pod 会获得属于自己的 IP 地址（Kubernetes 期待网络插件来保证这一点）。 对于集群中给定的某个 Deployment，这一刻运行的 Pod 集合可能不同于下一刻运行该应用的 Pod 集合。

这就带来了一个问题：如果某组 Pod（称为“后端”）为集群内的其他 Pod（称为“前端”） 集合提供功能，前端要如何发现并跟踪要连接的 IP 地址，以便其使用负载的后端组件呢？

### Kubernetes 中的 Service

Service API 是 Kubernetes 的组成部分，它是一种抽象，帮助你将 Pod 集合在网络上公开出去。 每个 Service 对象定义端点的一个逻辑集合（通常这些端点就是 Pod）以及如何访问到这些 Pod 的策略。

### 云原生服务发现

如果你想要在自己的应用中使用 Kubernetes API 进行服务发现，可以查询 [API 服务器](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-apiserver)， 寻找匹配的 EndpointSlice 对象。 只要 Service 中的 Pod 集合发生变化，Kubernetes 就会为其更新 EndpointSlice。

对于非本地应用，Kubernetes 提供了在应用和后端 Pod 之间放置网络端口或负载均衡器的方法。

无论采用那种方式，你的负载都可以使用这里的[服务发现](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#discovering-services)机制找到希望连接的目标。

### 定义 Service

Kubernetes 中的 Service 是一个[对象](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/#kubernetes-objects) （与 Pod 或 ConfigMap 类似）。你可以使用 Kubernetes API 创建、查看或修改 Service 定义。 通常你会使用 `kubectl` 这类工具来替你发起这些 API 调用。

例如，假定有一组 Pod，每个 Pod 都在侦听 TCP 端口 9376，并且它们还被打上 `app.kubernetes.io/name=MyApp` 标签。你可以定义一个 Service 来发布该 TCP 侦听器

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

应用上述清单时，系统将创建一个名为 "my-service" 的、 [服务类型](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#publishing-services-service-types)默认为 ClusterIP 的 Service。 该 Service 指向带有标签 `app.kubernetes.io/name: MyApp` 的所有 Pod 的 TCP 端口 9376

Kubernetes 为该服务分配一个 IP 地址（称为 “集群 IP”），供虚拟 IP 地址机制使用。 有关该机制的更多详情

>**说明：**
>
>- Service 对象的名称必须是有效的 [RFC 1035 标签名称](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#rfc-1035-label-names)。
>
>- Service 能够将任意入站 `port` 映射到某个 `targetPort`。 默认情况下，出于方便考虑，`targetPort` 会被设置为与 `port` 字段相同的值。

### 端口定义

Pod 中的端口定义是有名字的，你可以在 Service 的 `targetPort` 属性中引用这些名字。 例如，我们可以通过以下方式将 Service 的 `targetPort` 绑定到 Pod 端口：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```

即使在 Service 中混合使用配置名称相同的多个 Pod，各 Pod 通过不同的端口号支持相同的网络协议， 此机制也可以工作。这一机制为 Service 的部署和演化提供了较高的灵活性。 例如，你可以在后端软件的新版本中更改 Pod 公开的端口号，但不会影响到客户端。

Service 的默认协议是 [TCP](https://kubernetes.io/zh-cn/docs/reference/networking/service-protocols/#protocol-tcp)； 你还可以使用其他[受支持的任何协议](https://kubernetes.io/zh-cn/docs/reference/networking/service-protocols/)。

由于许多 Service 需要公开多个端口，所以 Kubernetes 为同一服务定义[多个端口](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#multi-port-services)。 每个端口定义可以具有相同的 `protocol`，也可以具有不同协议。

### 没有选择算符(selector)的 Service

由于选择算符(selector)的存在，Service 的最常见用法是为 Kubernetes Pod 集合提供访问抽象， 但是当与相应的 [EndpointSlice](https://kubernetes.io/zh-cn/docs/concepts/services-networking/endpoint-slices/) 对象一起使用且没有设置选择算符时，Service 也可以为其他类型的后端提供抽象， 包括在集群外运行的后端。

例如：

- 你希望在生产环境中使用外部数据库集群，但在测试环境中使用自己的数据库。
- 你希望让你的 Service 指向另一个[名字空间（Namespace）](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/namespaces/)中或其它集群中的服务。
- 你正在将工作负载迁移到 Kubernetes 上来。在评估所采用的方法时，你仅在 Kubernetes 中运行一部分后端。

在所有这些场景中，你都可以定义**不**指定用来匹配 Pod 的选择算符的 Service。例如：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

由于此 Service 没有选择算符，因此不会自动创建对应的 EndpointSlice（和旧版的 Endpoints）对象。 你可以通过手动添加 EndpointSlice 对象，将 Service 映射到该服务运行位置的网络地址和端口：

```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: my-service-1 # 按惯例将服务的名称用作 EndpointSlice 名称的前缀
  labels:
    # 你应设置 "kubernetes.io/service-name" 标签。
    # 设置其值以匹配服务的名称
    kubernetes.io/service-name: my-service
addressType: IPv4
ports:
  - name: '' # 留空，因为 port 9376 未被 IANA 分配为已注册端口
    appProtocol: http
    protocol: TCP
    port: 9376
endpoints:  # 此列表中的 IP 地址可以按任何顺序显示
  - addresses:
      - "10.4.5.6"
  - addresses:
      - "10.1.2.3"
```

#### 自定义 EndpointSlices

当为 Service 创建 [EndpointSlice](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#endpointslices) 对象时，可以为 EndpointSlice 使用任何名称。 一个名字空间中的各个 EndpointSlice 都必须具有一个唯一的名称。通过在 EndpointSlice 上设置 `kubernetes.io/service-name` [标签](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)可以将 EndpointSlice 链接到 Service。

> **说明：**
>
> 端点 IP 地址**必须不是**：本地回路地址（IPv4 的 127.0.0.0/8、IPv6 的 ::1/128） 或链路本地地址（IPv4 的 169.254.0.0/16 和 224.0.0.0/24、IPv6 的 fe80::/64）。
>
> 端点 IP 地址不能是其他 Kubernetes 服务的集群 IP，因为 [kube-proxy](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-proxy/) 不支持将虚拟 IP 作为目标地址。

对于你自己或在你自己代码中创建的 EndpointSlice，你还应该为 [`endpointslice.kubernetes.io/managed-by`](https://kubernetes.io/zh-cn/docs/reference/labels-annotations-taints/#endpointslicekubernetesiomanaged-by) 标签设置一个值。如果你创建自己的控制器代码来管理 EndpointSlice， 请考虑使用类似于 `"my-domain.example/name-of-controller"` 的值。 如果你使用的是第三方工具，请使用全小写的工具名称，并将空格和其他标点符号更改为短划线 (`-`)。 如果直接使用 `kubectl` 之类的工具来管理 EndpointSlice 对象，请使用用来描述这种手动管理的名称， 例如 `"staff"` 或 `"cluster-admins"`。你要避免使用保留值 `"controller"`； 该值标识由 Kubernetes 自己的控制平面管理的 EndpointSlice。

#### 访问没有选择算符的 Service

访问没有选择算符的 Service 与有选择算符的 Service 的原理相同。 在没有选择算符的 Service [示例](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#services-without-selectors)中， 流量被路由到 EndpointSlice 清单中定义的两个端点之一： 通过 TCP 协议连接到 10.1.2.3 或 10.4.5.6 的端口 9376。

> **说明：**
>
> Kubernetes API 服务器不允许将流量代理到未被映射至 Pod 上的端点。由于此约束，当 Service 没有选择算符时，诸如 `kubectl proxy <service-name>` 之类的操作将会失败。这可以防止 Kubernetes API 服务器被用作调用者可能无权访问的端点的代理。

`ExternalName` Service 是 Service 的特例，它没有选择算符，而是使用 DNS 名称。 更多的相关信息，请参阅 [ExternalName](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#externalname) 一节。

### EndpointSlices

[EndpointSlice](https://kubernetes.io/zh-cn/docs/concepts/services-networking/endpoint-slices/) 对象表示某个服务的后端网络端点的子集（**切片**）。

你的 Kubernetes 集群会跟踪每个 EndpointSlice 所表示的端点数量。 如果 Service 的端点太多以至于达到阈值，Kubernetes 会添加另一个空的 EndpointSlice 并在其中存储新的端点信息。 默认情况下，一旦现有 EndpointSlice 都包含至少 100 个端点，Kubernetes 就会创建一个新的 EndpointSlice。 在需要添加额外的端点之前，Kubernetes 不会创建新的 EndpointSlice。

### Endpoints

在 Kubernetes API 中，[Endpoints](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/service-resources/endpoints-v1/) （该资源类别为复数形式）定义的是网络端点的列表，通常由 Service 引用， 以定义可以将流量发送到哪些 Pod。

推荐使用 EndpointSlice API 替换 Endpoints。

#### 超出容量的端点

Kubernetes 限制单个 Endpoints 对象中可以容纳的端点数量。 当一个 Service 拥有 1000 个以上支撑端点时，Kubernetes 会截断 Endpoints 对象中的数据。 由于一个 Service 可以链接到多个 EndpointSlice 之上，所以 1000 个支撑端点的限制仅影响旧版的 Endpoints API。

如出现端点过多的情况，Kubernetes 选择最多 1000 个可能的后端端点存储到 Endpoints 对象中， 并在 Endpoints 上设置[注解](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/annotations/) [`endpoints.kubernetes.io/over-capacity: truncated`](https://kubernetes.io/zh-cn/docs/reference/labels-annotations-taints/#endpoints-kubernetes-io-over-capacity)。 如果后端 Pod 的数量降至 1000 以下，控制平面也会移除该注解。

请求流量仍会被发送到后端，但任何依赖旧版 Endpoints API 的负载均衡机制最多只能将流量发送到 1000 个可用的支撑端点。

这一 API 限制也意味着你不能手动将 Endpoints 更新为拥有超过 1000 个端点。

### 应用协议

`appProtocol` 字段提供了一种为每个 Service 端口设置应用协议的方式。 此字段被实现代码用作一种提示信息，以便针对实现能够理解的协议提供更为丰富的行为。 此字段的取值会被映射到对应的 Endpoints 和 EndpointSlice 对象中。

此字段遵循标准的 Kubernetes 标签语法。合法的取值值可以是以下之一：

- [IANA 标准服务名称](https://www.iana.org/assignments/service-names)。

- 由具体实现所定义的、带有 `mycompany.com/my-custom-protocol` 这类前缀的名称。

- Kubernetes 定义的前缀名称：

  | 协议                | 描述                                                         |
  | ------------------- | ------------------------------------------------------------ |
  | `kubernetes.io/h2c` | 基于明文的 HTTP/2 协议，如 [RFC 7540](https://www.rfc-editor.org/rfc/rfc7540) 所述 |

### 多端口 Service

对于某些 Service，你需要公开多个端口。Kubernetes 允许你为 Service 对象配置多个端口定义。 为 Service 使用多个端口时，必须为所有端口提供名称，以使它们无歧义。 例如：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

> **说明：**
>
> 与一般的 Kubernetes 名称一样，端口名称只能包含小写字母、数字和 `-`。 端口名称还必须以字母或数字开头和结尾。
>
> 例如，名称 `123-abc` 和 `web` 是合法的，但是 `123_abc` 和 `-web` 不合法。

## 服务类型

对一些应用的某些部分（如前端），你可能希望将其公开于某外部 IP 地址， 也就是可以从集群外部访问的某个地址。

Kubernetes Service 类型允许指定你所需要的 Service 类型。

可用的 `type` 值及其行为有：

- `ClusterIP`

  通过集群的内部 IP 公开 Service，选择该值时 Service 只能够在集群内部访问。 这也是你没有为服务显式指定 `type` 时使用的默认值。 你可以使用 [Ingress](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/) 或者 [Gateway API](https://gateway-api.sigs.k8s.io/) 向公共互联网公开服务。

- [`NodePort`](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport)

  通过每个节点上的 IP 和静态端口（`NodePort`）公开 Service。 为了让 Service 可通过节点端口访问，Kubernetes 会为 Service 配置集群 IP 地址， 相当于你请求了 `type: ClusterIP` 的服务。

- [`LoadBalancer`](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#loadbalancer)

  使用云平台的负载均衡器向外部公开 Service。Kubernetes 不直接提供负载均衡组件； 你必须提供一个，或者将你的 Kubernetes 集群与某个云平台集成。

- [`ExternalName`](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#externalname)

  将服务映射到 `externalName` 字段的内容（例如，映射到主机名 `api.foo.bar.example`）。 该映射将集群的 DNS 服务器配置为返回具有该外部主机名值的 `CNAME` 记录。 集群不会为之创建任何类型代理。

服务 API 中的 `type` 字段被设计为层层递进的形式 - 每层都建立在前一层的基础上。 并不是所有云平台都作如此严格要求，但 Kubernetes 的 Service API 设计要求满足这一逻辑。

#### `type: ClusterIP`

此默认 Service 类型从你的集群中为此预留的 IP 地址池中分配一个 IP 地址。

其他几种 Service 类型在 `ClusterIP` 类型的基础上进行构建。

如果你定义的 Service 将 `.spec.clusterIP` 设置为 `"None"`，则 Kubernetes 不会为其分配 IP 地址。有关详细信息，请参阅[无头服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#headless-services)。

##### 选择自己的 IP 地址

在创建 `Service` 的请求中，你可以通过设置 `spec.clusterIP` 字段来指定自己的集群 IP 地址。 比如，希望复用一个已存在的 DNS 条目，或者遗留系统已经配置了一个固定的 IP 且很难重新配置。

你所选择的 IP 地址必须是合法的 IPv4 或者 IPv6 地址，并且这个 IP 地址在 API 服务器上所配置的 `service-cluster-ip-range` CIDR 范围内。 如果你尝试创建一个带有非法 `clusterIP` 地址值的 Service，API 服务器会返回 HTTP 状态码 422， 表示值不合法。

请阅读[避免冲突](https://kubernetes.io/zh-cn/docs/reference/networking/virtual-ips/#avoiding-collisions)节， 以了解 Kubernetes 如何协助降低两个不同的 Service 试图使用相同 IP 地址的风险和影响。

#### `type: NodePort`

如果你将 `type` 字段设置为 `NodePort`，则 Kubernetes 控制平面将在 `--service-node-port-range` 标志所指定的范围内分配端口（默认值：30000-32767）。 每个节点将该端口（每个节点上的相同端口号）上的流量代理到你的 Service。 你的 Service 在其 `.spec.ports[*].nodePort` 字段中报告已分配的端口。

使用 NodePort 可以让你自由设置自己的负载均衡解决方案， 配置 Kubernetes 不完全支持的环境， 甚至直接公开一个或多个节点的 IP 地址。

对于 NodePort 服务，Kubernetes 额外分配一个端口（TCP、UDP 或 SCTP 以匹配 Service 的协议）。 集群中的每个节点都将自己配置为监听所分配的端口，并将流量转发到与该 Service 关联的某个就绪端点。 通过使用合适的协议（例如 TCP）和适当的端口（分配给该 Service）连接到任何一个节点， 你就能够从集群外部访问 `type: NodePort` 服务。

#### 选择你自己的端口

如果需要特定的端口号，你可以在 `nodePort` 字段中指定一个值。 控制平面将或者为你分配该端口，或者报告 API 事务失败。 这意味着你需要自行注意可能发生的端口冲突。 你还必须使用有效的端口号，该端口号在配置用于 NodePort 的范围内。

以下是 `type: NodePort` 服务的一个清单示例，其中指定了 NodePort 值（在本例中为 30007）：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    # 默认情况下，为了方便起见，`targetPort` 被设置为与 `port` 字段相同的值。
    - port: 80
      targetPort: 80
      # 可选字段
      # 默认情况下，为了方便起见，Kubernetes 控制平面会从某个范围内分配一个端口号
      #（默认：30000-32767）
      nodePort: 30007
```

##### 预留 NodePort 端口范围以避免分配端口时发生冲突

**特性状态：** `Kubernetes v1.28 [beta]`

为 NodePort 服务分配端口的策略既适用于自动分配的情况，也适用于手动分配的场景。 当某个用于希望创建一个使用特定端口的 NodePort 服务时，该目标端口可能与另一个已经被分配的端口冲突。 这时，你可以启用特性门控 `ServiceNodePortStaticSubrange`，进而为 NodePort Service 使用不同的端口分配策略。用于 NodePort 服务的端口范围被分为两段。 动态端口分配默认使用较高的端口段，并且在较高的端口段耗尽时也可以使用较低的端口段。 用户可以从较低端口段中分配端口，降低端口冲突的风险。

##### 为 `type: NodePort` 服务自定义 IP 地址配置

你可以配置集群中的节点使用特定 IP 地址来支持 NodePort 服务。 如果每个节点都连接到多个网络（例如：一个网络用于应用流量，另一网络用于节点和控制平面之间的流量）， 你可能想要这样做。

如果你要指定特定的 IP 地址来为端口提供代理，可以将 kube-proxy 的 `--nodeport-addresses` 标志或 [kube-proxy 配置文件](https://kubernetes.io/zh-cn/docs/reference/config-api/kube-proxy-config.v1alpha1/)中的等效字段 `nodePortAddresses` 设置为特定的 IP 段。

此标志接受逗号分隔的 IP 段列表（例如 `10.0.0.0/8`、`192.0.2.0/25`），用来设置 IP 地址范围。 kube-proxy 应视将其视为所在节点的本机地址。

例如，如果你使用 `--nodeport-addresses=127.0.0.0/8` 标志启动 kube-proxy， 则 kube-proxy 仅选择 NodePort 服务的本地回路接口。 `--nodeport-addresses` 的默认值是一个空的列表。 这意味着 kube-proxy 将认为所有可用网络接口都可用于 NodePort 服务 （这也与早期的 Kubernetes 版本兼容。）

> **说明：**
>
> 此 Service 的可见形式为 `<NodeIP>:spec.ports[*].nodePort` 以及 `.spec.clusterIP:spec.ports[*].port`。 如果设置了 kube-proxy 的 `--nodeport-addresses` 标志或 kube-proxy 配置文件中的等效字段， 则 `<NodeIP>` 将是一个被过滤的节点 IP 地址（或可能是多个 IP 地址）。

#### `type: LoadBalancer`

在使用支持外部负载均衡器的云平台时，如果将 `type` 设置为 `"LoadBalancer"`， 则平台会为 Service 提供负载均衡器。 负载均衡器的实际创建过程是异步进行的，关于所制备的负载均衡器的信息将会通过 Service 的 `status.loadBalancer` 字段公开出来。 例如

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127

```

#### ExternalName 类型

类型为 ExternalName 的 Service 将 Service 映射到 DNS 名称，而不是典型的选择算符， 例如 `my-service` 或者 `cassandra`。你可以使用 `spec.externalName` 参数指定这些服务。

例如，以下 Service 定义将 `prod` 名字空间中的 `my-service` 服务映射到 `my.database.example.com`：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

> **说明：**
>
> `type: ExternalName` 的服务接受 IPv4 地址字符串，但将该字符串视为由数字组成的 DNS 名称， 而不是 IP 地址（然而，互联网不允许在 DNS 中使用此类名称）。 类似于 IPv4 地址的外部名称无法被 DNS 服务器解析。
>
> 如果你想要将服务直接映射到某特定 IP 地址，请考虑使用[无头服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#headless-services)。

### 无头服务（Headless Services）

有时你并不需要负载均衡，也不需要单独的 Service IP。遇到这种情况，可以通过显式设置 集群 IP（`spec.clusterIP`）的值为 `"None"` 来创建**无头服务（Headless Service）**。

你可以使用无头 Service 与其他服务发现机制交互，而不必绑定到 Kubernetes 的实现。

无头 Service 不会获得集群 IP，kube-proxy 不会处理这类 Service， 而且平台也不会为它们提供负载均衡或路由支持。 取决于 Service 是否定义了选择算符，DNS 会以不同的方式被自动配置。

#### 带选择算符(selector)的服务

对定义了选择算符的无头 Service，Kubernetes 控制平面在 Kubernetes API 中创建 EndpointSlice 对象，并且修改 DNS 配置返回 A 或 AAAA 记录（IPv4 或 IPv6 地址）， 这些记录直接指向 Service 的后端 Pod 集合。

#### 无选择算符的服务

对没有定义选择算符的无头 Service，控制平面不会创建 EndpointSlice 对象。 然而 DNS 系统会执行以下操作之一：

- 对于 [`type: ExternalName`](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#externalname) Service，查找和配置其 CNAME 记录；
- 对所有其他类型的 Service，针对 Service 的就绪端点的所有 IP 地址，查找和配置 DNS A / AAAA 记录：
  - 对于 IPv4 端点，DNS 系统创建 A 记录。
  - 对于 IPv6 端点，DNS 系统创建 AAAA 记录。

当你定义无选择算符的无头 Service 时，`port` 必须与 `targetPort` 匹配。

### 会话的黏性

如果你想确保来自特定客户端的连接每次都传递到同一个 Pod，你可以配置基于客户端 IP 地址的会话亲和性。可阅读[会话亲和性](https://kubernetes.io/zh-cn/docs/reference/networking/virtual-ips/#session-affinity) 来进一步学习。

#### 外部 IP

如果有外部 IP 能够路由到一个或多个集群节点上，则 Kubernetes Service 可以在这些 `externalIPs` 上公开出去。当网络流量进入集群时，如果外部 IP（作为目的 IP 地址）和端口都与该 Service 匹配， Kubernetes 所配置的规则和路由会确保流量被路由到该 Service 的端点之一。

定义 Service 时，你可以为任何[服务类型](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#publishing-services-service-types)指定 `externalIPs`。

在下面的例子中，名为 `my-service` 的服务可以在 "`198.51.100.32:80`" （根据 `.spec.externalIPs[]` 和 `.spec.ports[].port` 得出）上被客户端使用 TCP 协议访问。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 49152
  externalIPs:
    - 198.51.100.32
```

**说明：**

Kubernetes 不负责管理 `externalIPs` 的分配，这一工作是集群管理员的职责。