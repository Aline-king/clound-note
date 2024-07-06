---
description: https://www.envoyproxy.io
cover: >-
  https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2817/1719826372096/ac352a926134447dacf829a2eda518ad.png
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Envoy

[Envoy](https://www.envoyproxy.io/) 是一个用 C++ 开发的高性能代理，Envoy 是一种 L7 代理和通信总线，专为大型的现代面向服务的架构而设计。

#### WHY ENVOY? <a href="#why-envoy-_5" id="why-envoy-_5"></a>

> As on the ground microservice practitioners quickly realize, the majority of operational problems that arise when moving to a distributed architecture are ultimately grounded in two areas: networking and observability. It is simply an orders of magnitude larger problem to network and debug a set of intertwined distributed services versus a single monolithic application.
>
> Originally built at **Lyft**, Envoy is a high performance C++ distributed proxy designed for single services and applications, as well as a communication bus and “universal data plane” designed for large microservice “service mesh” architectures. Built on the learnings of solutions such as NGINX, HAProxy, hardware load balancers, and cloud load balancers, Envoy runs alongside every application and abstracts the network by providing common features in a platform-agnostic manner. When all service traffic in an infrastructure flows via an Envoy mesh, it becomes easy to visualize problem areas via consistent observability, tune overall performance, and add substrate features in a single place.

译文

> 正如进行微服务实践的地面工作者迅速意识到的那样，转向分布式架构时出现的大多数运营问题最终都可以归结为两个方面：网络和可观察性。相比于单一的单体应用，对一套错综复杂的分布式服务进行网络构建和调试所面临的问题，其难度简直是数量级上的增加。
>
> Envoy最初在Lyft公司开发而成，它是一个高性能的C++分布式代理，旨在服务于单个服务和应用程序，同时也是一个通信总线和“通用数据平面”，专为大型的微服务“服务网格”架构设计。Envoy基于对NGINX、HAProxy、硬件负载均衡器及云负载均衡器等解决方案的学习而构建，它与每个应用程序并行运行，以平台无关的方式提供通用功能，从而抽象化网络。当一个基础设施中的所有服务流量都通过Envoy服务网格流动时，便能够通过一致的可观察性轻松识别问题区域，调整整体性能，并在一个统一的地方添加基础架构特性。

