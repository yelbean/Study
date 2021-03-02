## What is Envoy
Envoy is an L7 proxy and communication bus designed for large modern service oriented architectures. The project was born out of the belief that:
    The network should be transparent to applications. When network and application problems do occur it should be easy to determine the source of the problem.

## Terminology 术语
Host: An entity capable of network communication (application on a mobile phone, server, etc.). In this documentation a host is a logical network application. A physical piece of hardware could possibly have multiple hosts running on it as long as each of them can be independently addressed.
能够进行网络通信的实体（在手机，服务器等上的应用程序）。 在本文档中，主机是逻辑网络应用程序。 一个物理硬件可以在其上运行多个主机，只要它们中的每一个都可以独立寻址即可。

Downstream: A downstream host connects to Envoy, sends requests, and receives responses.
下游主机连接envoy，发送请求并接收响应。

Upstream: An upstream host receives connections and requests from Envoy and returns responses.
上游主机接收来自envoy的连接和请求并返回响应。

Listener: A listener is a named network location (e.g., port, unix domain socket, etc.) that can be connected to by downstream clients. Envoy exposes one or more listeners that downstream hosts connect to.
监听器是一个命名的网络位置(例如，端口，unix域套接字等)，可以被下游客户端连接。Envoy暴露一个或多个侦听器给下游主机连接。

Cluster: A cluster is a group of logically similar upstream hosts that Envoy connects to. Envoy discovers the members of a cluster via service discovery. It optionally determines the health of cluster members via active health checking. The cluster member that Envoy routes a request to is determined by the load balancing policy.
集群是一组逻辑上相似的上游主机，通过envoy连接到它们。envoy通过服务发现发现集群成员。它可以选择通过主动的健康检查来确定集群成员的运行状况。Envoy将请求路由到的群集成员由负载平衡策略确定。

Mesh: A group of hosts that coordinate to provide a consistent network topology. In this documentation, an “Envoy mesh” is a group of Envoy proxies that form a message passing substrate for a distributed system comprised of many different services and application platforms.
一组主机进行协调以提供一致的网络拓扑。 在本文档中，“Envoy网格”是一组Envoy代理，它们构成了由许多不同服务和应用程序平台组成的分布式系统的消息传递基础。

Runtime configuration: Out of band realtime configuration system deployed alongside Envoy. Configuration settings can be altered that will affect operation without needing to restart Envoy or change the primary configuration.
与Envoy一起部署的带外实时配置系统。 可以更改将影响操作的配置设置，而不需重新启动Envoy或更改主配置。



