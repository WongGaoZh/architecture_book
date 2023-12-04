# 链路追踪
虽然 2010 年之前就已经有了 X-Trace、Magpie 等跨服务的追踪系统了，但现代分布式链路追踪公认的起源是 Google 在 2010 年发表的论文《[Dapper : a Large-Scale Distributed Systems Tracing Infrastructure](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/papers/dapper-2010-1.pdf)》，这篇论文介绍了 Google 从 2004 年开始使用的分布式追踪系统 Dapper 的实现原理。此后，所有业界有名的追踪系统，无论是国外 Twitter 的[Zipkin](https://github.com/openzipkin/zipkin)、Naver 的[Pinpoint](https://github.com/pinpoint-apm/pinpoint)（Naver 是 Line 的母公司，Pinpoint 出现其实早于 Dapper 论文发表，在 Dapper 论文中还提到了 Pinpoint），抑或是国内阿里的鹰眼、大众点评的[CAT](https://github.com/dianping/cat)、个人开源的[SkyWalking](https://github.com/apache/skywalking)（后进入 Apache 基金会孵化毕业）都受到 Dapper 论文的直接影响。

广义上讲，一个完整的分布式追踪系统应该由数据收集、数据存储和数据展示三个相对独立的子系统构成，而狭义上讲的追踪则就只是特指链路追踪数据的收集部分。譬如[Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth)就属于狭义的追踪系统，通常会搭配 Zipkin 作为数据展示，搭配 Elasticsearch 作为数据存储来组合使用，而前面提到的那些 Dapper 的徒子徒孙们大多都属于广义的追踪系统，广义的追踪系统又常被称为“APM 系统”（Application Performance Management）。

## 追踪与跨度

追踪 : 客户端发送一次请求,经过了所有的微服务, 直到像客户端返回结果就叫做一次追踪;
跨度 : 由于每次 Trace 都可能会调用数量不定、坐标不定的多个服务，为了能够记录具体调用了哪些服务，以及调用的顺序、开始时点、执行时长等信息，每次开始调用服务前都要先埋入一个调用记录，这个记录称为一个“跨度”

链路追踪的目的是为排查故障和分析性能提供数据支持，系统对外提供服务的过程中，持续地接受请求并处理响应，同时持续地生成 Trace，按次序整理好 Trace 中每一个 Span 所记录的调用关系，便能绘制出一幅系统的服务调用拓扑图。

## 数据收集
目前，追踪系统根据数据收集方式的差异，可分为三种主流的实现方式，分别是**基于日志的追踪（Log-Based Tracing），基于服务的追踪（Service-Based Tracing）和基于边车代理的追踪（Sidecar-Based Tracing）**

+ 基于日志的追踪的思路是将 Trace、Span 等信息直接输出到应用日志中，然后随着所有节点的日志归集过程汇聚到一起. 基于日志追踪的代表产品是 Spring Cloud Sleuth
+ 基于服务的追踪是目前最为常见的追踪实现方式，被 Zipkin、SkyWalking、Pinpoint 等主流追踪系统广泛采用。服务追踪的实现思路是通过某些手段给目标应用注入追踪探针（Probe）
+ 基于边车代理的追踪是服务网格的专属方案，也是最理想的分布式追踪模型，它对应用完全透明，无论是日志还是服务本身都不会有任何变化；它与程序语言无关，无论应用采用什么编程语言实现，只要它还是通过网络（HTTP 或者 gRPC）来访问服务就可以被追踪到；它有自己独立的数据通道，追踪数据通过控制平面进行上报，避免了追踪对程序通信或者日志归集的依赖和干扰，保证了最佳的精确性

