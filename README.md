![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg) ![](https://parg.co/bDm)

![封面图](https://i.postimg.cc/8kvMhcZL/image.png)

# Cloud Series

# Preface（前言）

![Cloud Native Computing Foundation](https://s1.ax1x.com/2020/09/07/wKC9Tx.md.png)

## 虚拟化

在计算机科学中，虚拟化技术（Virtualization）是一种资源管理（优化）技术，将计算机的各种物理资源（e.g. CPU、内存以及磁盘空间、网络适配器等 IO 设备）予以抽象、转换，然后呈现出来的一个可供分割并任意组合为一个或多个（虚拟）计算机的配置环境。虚拟化技术打破了计算机内部实体结构间不可切割的障碍，使用户能够以比原本更好的配置方式来应用这些计算机硬件资源。而这些资源的虚拟形式将不受现有架设方式，地域或物理配置所限制。虚拟化技术是一个广义的术语，根据不同的对象类型可以细分为：

- 平台虚拟化（Platform Virtualization）：针对计算机和操作系统的虚拟化。

- 资源虚拟化（Resource Virtualization）：针对特定的系统资源的虚拟化，如内存、存储、网络资源等。

- 应用程序虚拟化（Application Virtualization）：包括仿真、模拟、解释技术等，如 Java 虚拟机（JVM）。

![虚拟化技术发展](https://s2.ax1x.com/2019/08/31/mx0hXn.jpg)

从虚拟化技术诞生以来，IaaS / PaaS / SaaS 概念陆续被提了出来，各种容器技术层出不穷。

## Kubernetes 与编排

随着虚拟化技术的成熟和分布式架构的普及，用来部署、管理和运行应用的云平台被越来越多地提及。IaaS、PaaS 和 SaaS 是云计算的三种基本服务类型，分别表示关注硬件基础设施的基础设施即服务、关注软件和中间件平台的平台即服务，以及关注业务应用的软件即服务。容器的出现，使原有的基于虚拟机的云主机应用，彻底转变为更加灵活和轻量的容器与编排调度的云平台应用。

![Kubernetes Master](https://i.postimg.cc/6qxTzd39/image.png)

然而容器单元越来越散落使得管理成本逐渐上升，大家对容器编排工具的需求前所未有的强烈，Kubernetes、Mesos、Swarm 等为云原生应用提供了强有力的编排和调度能力，它们是云平台上的分布式操作系统。容器编排是通常可以部署多个容器以通过自动化实现应用程序的过程。像 Kubernetes 和 Docker Swarm 这样的容器管理和容器编排引擎，使用户能够指导容器部署并自动执行更新，运行状况监视和故障转移过程。

Kubernetes 是目前世界范围内关注度最高的开源项目，它是一个出色的容器编排系统，用于提供一站式服务。Kubernetes 出身于互联网行业巨头 Google，它借鉴了由上百位工程师花费十多年时间打造的 Borg 系统的理念，安装极其简易，网络层对接方式十分灵活。Kubernetes 和 Mesos 的出色表现给行业中各类工程师的工作模式带来了颠覆性的改变。他们再也不用关注每一台服务器，当服务器出现问题时，只要将其换掉即可。业务开发工程师不必再过分关注非功能需求，只需专注自己的业务领域即可。而中间件开发工程师则需要开发出健壮的云原生中间件，用来连接业务应用与云平台。

# About

## Copyright & More | 延伸阅读

您还可以前往 [NGTE Books](https://ng-tech.icu/books/) 主页浏览包含知识体系、编程语言、软件工程、模式与架构、Web 与大前端、服务端开发实践与工程架构、分布式基础架构、人工智能与深度学习、产品运营与创业等多类目的书籍列表：

[![NGTE Books](https://s2.ax1x.com/2020/01/18/19uXtI.png)](https://ng-tech.icu/books/)

## Links

- [2018-kubernetes-tutorial #Series#](https://github.com/KeKe-Li/kubernetes-tutorial): Running Kubernetes cluster Locally tutorial.
