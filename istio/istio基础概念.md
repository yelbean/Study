## Istio 是什么？
Istio 允许您连接、保护、控制和观察服务。有助于降低部署的复杂性，并减轻开发团队的压力。它是一个完全开源的服务网格，作为透明的一层接入到现有的分布式应用程序里。它也是一个平台，拥有可以集成任何日志、遥测和策略系统的 API 接口。Istio 多样化的特性使您能够成功且高效地运行分布式微服务架构，并提供保护、连接和监控微服务的统一方法。


## 服务网格是什么？
服务网格用来描述组成这些应用程序的微服务网络以及它们之间的交互。随着服务网格的规模和复杂性不断的增长，它将会变得越来越难以理解和管理。它的需求包括服务发现、负载均衡、故障恢复、度量和监控等。服务网格通常还有更复杂的运维需求，比如 A/B 测试、金丝雀发布、速率限制、访问控制和端到端认证。

Istio 提供了对整个服务网格的行为洞察和操作控制的能力，以及一个完整的满足微服务应用各种需求的解决方案。


## 为什么使用 Istio？
通过负载均衡、服务间的身份验证、监控等方法，Istio 可以轻松地创建一个已经部署了服务的网络，而服务的代码只需很少更改甚至无需更改。通过在整个环境中部署一个特殊的 sidecar 代理为服务添加 Istio 的支持，而代理会拦截微服务之间的所有网络通信，然后使用其控制平面的功能来配置和管理 Istio，这包括：

- 为 HTTP、gRPC、WebSocket 和 TCP 流量自动负载均衡。
- 通过丰富的路由规则、重试、故障转移和故障注入对流量行为进行细粒度控制。
- 可插拔的策略层和配置 API，支持访问控制、速率限制和配额。
- 集群内（包括集群的入口和出口）所有流量的自动化度量、日志记录和追踪。
- 在具有强大的基于身份验证和授权的集群中实现安全的服务间通信。

Istio 为可扩展性而设计，可以满足不同的部署需求。


## Virtual Service
虚拟服务可以配置路由规则，在服务网格内将请求路由到服务，可以为一个或多个主机名指定流量行为
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v3
```


## Destination Rule
可以将虚拟服务视为将流量如何路由到给定目标地址，然后使用目标规则来配置该目标的流量。
在评估虚拟服务路由规则之后，目标规则将应用于流量的“真实”目标地址。
目标规则还允许您在调用整个目的地服务或特定服务子集时定制 Envoy 的流量策略，比如您喜欢的负载均衡模型、TLS 安全模式或熔断器设置。

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3
```


## Gateway
使用网关为网格来管理入站和出站流量，可以让您指定要进入或离开网格的流量。网关配置被用于运行在网格边界的独立 Envoy 代理，而不是服务工作负载的 sidecar 代理
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: ext-host-gwy
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - ext-host.example.com
    tls:
      mode: SIMPLE
      serverCertificate: /tmp/tls.crt
      privateKey: /tmp/tls.key

这个网关配置让 HTTPS 流量从 ext-host.example.com 通过 443 端口流入网格，但没有为请求指定任何路由规则。为想要工作的网关指定路由，您必须把网关绑定到虚拟服务上。正如下面的示例所示，使用虚拟服务的 gateways 字段进行设置：
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: virtual-svc
spec:
  hosts:
  - ext-host.example.com
  gateways:
    - ext-host-gwy
```

## Service Entry
使用服务入口（Service Entry） 来添加一个入口到 Istio 内部维护的服务注册中心。添加了服务入口后，Envoy 代理可以向服务发送流量，就好像它是网格内部的服务一样
配置服务入口允许您管理运行在网格外的服务的流量，它包括以下几种能力：
- 为外部目标 redirect 和转发请求，例如来自 web 端的 API 调用，或者流向遗留老系统的服务。
- 为外部目标定义重试、超时和故障注入策略。
- 添加一个运行在虚拟机的服务来扩展您的网格。
- 从逻辑上添加来自不同集群的服务到网格，在 Kubernetes 上实现一个多集群 Istio 网格。

下面示例的 mesh-external 服务入口将 ext-resource 外部依赖项添加到 Istio 的服务注册中心：
```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: svc-entry
spec:
  hosts:
  - ext-svc.example.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS
```
您指定的外部资源使用 hosts 字段。可以使用完全限定名或通配符作为前缀域名。
您可以配置虚拟服务和目标规则，以更细粒度的方式控制到服务入口的流量，这与网格中的任何其他服务配置流量的方式相同。例如，下面的目标规则配置流量路由以使用双向 TLS 来保护到 ext-svc.example.com 外部服务的连接，我们使用服务入口配置了该外部服务：
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ext-res-dr
spec:
  host: ext-svc.example.com
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /etc/certs/myclientcert.pem
      privateKey: /etc/certs/client_private_key.pem
      caCertificates: /etc/certs/rootcacerts.pem
```

## Sidecar
默认情况下，Istio 让每个 Envoy 代理都可以访问来自和它关联的工作负载的所有端口的请求，然后转发到对应的工作负载。您可以使用 sidecar 配置去做下面的事情：
- 微调 Envoy 代理接受的端口和协议集。
- 限制 Envoy 代理可以访问的服务集合。

您可能希望在较庞大的应用程序中限制这样的 sidecar 可达性，配置每个代理能访问网格中的任意服务可能会因为高内存使用量而影响网格的性能。

您可以指定将 sidecar 配置应用于特定命名空间中的所有工作负载，或者使用 workloadSelector 选择特定的工作负载。例如，下面的 sidecar 配置将 bookinfo 命名空间中的所有服务配置为仅能访问运行在相同命名空间和 Istio 控制平面中的服务（目前需要使用 Istio 的策略和遥测功能）：
```
apiVersion: networking.istio.io/v1alpha3
kind: Sidecar
metadata:
  name: default
  namespace: bookinfo
spec:
  egress:
  - hosts:
    - "./*"
    - "istio-system/*"
```

## 超时
超时是 Envoy 代理等待来自给定服务的答复的时间量，以确保服务不会因为等待答复而无限期的挂起，并在可预测的时间范围内调用成功或失败。HTTP 请求的默认超时时间是 15 秒，这意味着如果服务在 15 秒内没有响应，调用将失败。

对于某些应用程序和服务，Istio 的缺省超时可能不合适。例如，超时太长可能会由于等待失败服务的回复而导致过度的延迟；而超时过短则可能在等待涉及多个服务返回的操作时触发不必要地失败。为了找到并使用最佳超时设置，Istio 允许您使用虚拟服务按服务轻松地动态调整超时，而不必修改您的业务代码。下面的示例是一个虚拟服务，它对 ratings 服务的 v1 子集的调用指定 10 秒超时：
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    timeout: 10s
```

## 重试
重试设置指定如果初始调用失败，Envoy 代理尝试连接服务的最大次数。通过确保调用不会因为临时过载的服务或网络等问题而永久失败，重试可以提高服务可用性和应用程序的性能。重试之间的间隔（25ms+）是可变的，并由 Istio 自动确定，从而防止被调用服务被请求淹没。HTTP 请求的默认重试行为是在返回错误之前重试两次。

与超时一样，Istio 默认的重试行为在延迟方面可能不适合您的应用程序需求（对失败的服务进行过多的重试会降低速度）或可用性。您可以在虚拟服务中按服务调整重试设置，而不必修改业务代码。您还可以通过添加每次重试的超时来进一步细化重试行为，并指定每次重试都试图成功连接到服务所等待的时间量。下面的示例配置了在初始调用失败后最多重试 3 次来连接到服务子集，每个重试都有 2 秒的超时。
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    retries:
      attempts: 3
      perTryTimeout: 2s
```


## 熔断器
熔断器是 Istio 为创建具有弹性的微服务应用提供的另一个有用的机制。在熔断器中，设置一个对服务中的单个主机调用的限制，例如并发连接的数量或对该主机调用失败的次数。一旦限制被触发，熔断器就会“跳闸”并停止连接到该主机。使用熔断模式可以快速失败而不必让客户端尝试连接到过载或有故障的主机。

熔断适用于在负载均衡池中的“真实”网格目标地址，您可以在目标规则中配置熔断器阈值，让配置适用于服务中的每个主机。下面的示例将 v1 子集的reviews服务工作负载的并发连接数限制为 100：
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 100
```

## 故障注入
在配置了网络，包括故障恢复策略之后，可以使用 Istio 的故障注入机制来为整个应用程序测试故障恢复能力。故障注入是一种将错误引入系统以确保系统能够承受并从错误条件中恢复的测试方法。使用故障注入特别有用，能确保故障恢复策略不至于不兼容或者太严格，这会导致关键服务不可用。

与其他错误注入机制（如延迟数据包或在网络层杀掉 Pod）不同，Istio 允许在应用层注入错误。这使您可以注入更多相关的故障，例如 HTTP 错误码，以获得更多相关的结果。

您可以注入两种故障，它们都使用虚拟服务配置：
- 延迟：延迟是时间故障。它们模拟增加的网络延迟或一个超载的上游服务。
- 终止：终止是崩溃失败。他们模仿上游服务的失败。终止通常以 HTTP 错误码或 TCP 连接失败的形式出现。

例如，下面的虚拟服务为千分之一的访问 ratings 服务的请求配置了一个 5 秒的延迟：
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    route:
    - destination:
        host: ratings
        subset: v1
```



