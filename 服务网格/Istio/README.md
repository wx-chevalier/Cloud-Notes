# Istio

Istio 是由 Google、IBM、Lyft 等共同开源的 Service Mesh（服务网格）框架，于 2017 年初开始进入大众视野，作为云原生时代下承 Kubernetes、上接 Serverless 架构的重要基础设施层，地位至关重要。

使用 Istio 可以很简单的创建具有负载均衡、服务间认证、监控等功能的服务网络，而不需要对服务的代码进行任何修改。你只需要在部署环境中，例如 Kubernetes 的 pod 里注入一个特别的 sidecar proxy 来增加对 Istio 的支持，用来截获微服务之间的网络流量。

![](https://res.cloudinary.com/jimmysong/image/upload/images/istio-arch.jpg)

# 背景分析

在 Kubernetes 之后，Istio 是最受欢迎的云原生技术。它就是一种服务网格，能够安全的连接一个应用程序之间的多个微服务。你也可以将它视为内部和外部的负载均衡器，具有策略驱动的防火墙，支持各种全面指标。开发者和使用者倾向于 Istio 的原因是因为它具有无侵入式的部署模式，而且任何 Kubernetes 的服务都能够在不需要改动代码和配置的情况下和 Istio 进行无缝连接。

如果说 Kubernetes 是新型的操作系统的话，Helm 就是应用程序安装程序。根据 Debian 安装包和 Red Hat Linux RMPS 设计，Helm 通过执行单个命令，提供了更简洁和更强大的部署云原生工作负载能力。

Kubernetes 应用暴露了大量的像 deployments（部署），services（服务），ingress controllers（入口控制器），persistant volumes（持久化挂载目录）等更多的元素。Helm 则通过提供统一安装工具，将云原生应用程序所有依赖关系聚合到称之为图表的部署单元中。

1. Spinnaker
   云原生技术最值得关注之一的是软件的交付速度。Spinnaker 是一个最初在 Netflix 上构建的开源项目，它实现了这一承诺。Spinnaker 是一个版本管理工具，它添加部署云原生应用的模板。通过对比传统的 Iass 环境（像 Amazon EC2 和当代运行在 Kubernetes 上的 Cass 平台），无缝填补了传统虚拟机和容器之间的空白。其多云功能使得其成为跨不同云平台部署应用程序的理想平台。

Spinnaker 可作为当前所有主流的云环境自托管平台，像 Armory 这样的初创公司目前正在提供 SLA 下的商业级，企业级 Spinnaker。

5. KubeLess
   事件驱动计算目前已成为当代应用程序结构不可或缺的一部分。功能即服务（Faas）是当前无服务计算交付模型之一，它通过基于事件的调用来填补容器。现代的应用程序会被当做服务并打包成容器或者是作为方法运行在相同的环境下，随着 Kubernetes 成为云原生计算的首选平台，运行功能时必须在容器中进行运行。

在云原生生态系统中，来自于 Bitnami 的 Kubeless 项目是当前最流行的无服务项目。它与 AWS lambda 的兼容性与对主流语言的支持使得它成为理想的选择。

# Architecture Overview | 架构概述

Istio 架构分为控制层和数据层。

- 数据层：由一组智能代理（Envoy）作为 sidecar 部署，协调和控制所有 microservices 之间的网络通信。
- 控制层：负责管理和配置代理路由流量，以及在运行时执行的政策。

## Envoy

Istio 使用 Envoy 代理的扩展版本，该代理是以 C++开发的高性能代理，用于调解 service mesh 中所有服务的所有入站和出站流量。Istio 利用了 Envoy 的许多内置功能，例如动态服务发现，负载平衡，TLS 终止，HTTP/2＆gRPC 代理，断路器，运行状况检查，基于百分比的流量拆分分阶段上线，故障注入和丰富指标。

Envoy 在 kubernetes 中作为 pod 的 sidecar 来部署。这允许 Istio 将大量关于流量行为的信号作为属性提取出来，这些属性又可以在 Mixer 中用于执行策略决策，并发送给监控系统以提供有关整个 mesh 的行为的信息。Sidecar 代理模型还允许你将 Istio 功能添加到现有部署中，无需重新构建或重写代码。更多信息参见设计目标。

## Mixer

Mixer 负责在 service mesh 上执行访问控制和使用策略，并收集 Envoy 代理和其他服务的遥测数据。代理提取请求级属性，发送到 mixer 进行评估。有关此属性提取和策略评估的更多信息，请参见 Mixer 配置。混音器包括一个灵活的插件模型，使其能够与各种主机环境和基础架构后端进行接口，从这些细节中抽象出 Envoy 代理和 Istio 管理的服务。

## Istio Manager

Istio-Manager 用作用户和 Istio 之间的接口，收集和验证配置，并将其传播到各种 Istio 组件。它从 Mixer 和 Envoy 中抽取环境特定的实现细节，为他们提供独立于底层平台的用户服务的抽象表示。此外，流量管理规则（即通用 4 层规则和七层 HTTP/gRPC 路由规则）可以在运行时通过 Istio-Manager 进行编程。

## Istio-auth

Istio-Auth 提供强大的服务间和最终用户认证，使用相互 TLS，内置身份和凭据管理。它可用于升级 service mesh 中的未加密流量，并为运营商提供基于服务身份而不是网络控制的策略的能力。Istio 的未来版本将增加细粒度的访问控制和审计，以使用各种访问控制机制（包括属性和基于角色的访问控制以及授权 hook）来控制和监控访问你服务、API 或资源的人员。

# Links

- https://www.servicemesher.com/blog/istio-kubernetes-service-mesh/
- https://www.servicemesher.com/blog/back-to-microservices-with-istio-p1/
