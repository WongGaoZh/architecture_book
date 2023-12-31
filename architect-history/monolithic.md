# 单体系统时代

> 单体架构（Monolithic）
> 
>“单体”只是表明系统中主要的过程调用都是进程内调用，不会发生进程间通信，仅此而已。

单体架构是今天绝大多数软件开发者都学习、实践过的一种软件架构，许多介绍微服务的书籍和技术资料中也常把这种架构风格的应用称作“[巨石系统](https://en.wikipedia.org/wiki/Monolithic_application)”（Monolithic Application）。“单体架构”在整个软件架构演进的历史进程里，是出现时间最早、应用范围最广、使用人数最多、统治历史最长的一种架构风格，但“单体”这个名称，却是在微服务开始流行之后才“事后追认”所形成的概念。此前，并没有多少人将“单体”视作一种架构来看待，如果你去查找软件架构的开发资料，可以轻而易举地找出大量以微服务为主题的书籍和文章，却很难找出专门教你如何开发单体系统的任何形式的材料，这一方面体现了单体架构本身的简单性，另一方面，也体现出在相当长的时间尺度里，大家都已经习惯了软件架构就应该是单体这种样子。

剖析单体架构之前，我们有必要先厘清一个概念误区，许多微服务的资料里，单体系统往往是以“反派角色”的身份登场的，譬如著名的微服务入门书《[微服务架构设计模式](https://book.douban.com/subject/33425123/)》，第一章的名字就是“逃离单体的地狱”。这些材料所讲的单体系统，其实都是有一个没有明说的隐含定语：“**大型的**单体系统”。对于小型系统——即由单台机器就足以支撑其良好运行的系统，单体不仅易于开发、易于测试、易于部署，且由于系统中各个功能、模块、方法的调用过程都是进程内调用，不会发生[进程间通信](https://zh.wikipedia.org/wiki/%E8%A1%8C%E7%A8%8B%E9%96%93%E9%80%9A%E8%A8%8A)（Inter-Process Communication，IPC。广义上讲，可以认为 RPC 属于 IPC 的一种特例，但请注意这里两个“PC”不是同个单词的缩写），因此也是运行效率最高的一种架构风格，完全不应该被贴上“反派角色”的标签，反倒是那些爱赶技术潮流却不顾需求现状的微服务吹捧者更像是个反派。单体系统的不足，必须基于软件的性能需求超过了单机，软件的开发人员规模明显超过了“[2 Pizza Team](https://wiki.mbalib.com/wiki/%E4%B8%A4%E4%B8%AA%E6%8A%AB%E8%90%A8%E5%8E%9F%E5%88%99)”范畴的前提下才有讨论的价值，因此，本文后续讨论中所说的单体，均应该是特指“大型的单体系统”，也正因如此，本节中说到“单体是出现最早的架构风格”，与上一节介绍原始分布式时代开篇提到的“使用多个独立的分布式服务共同构建一个更大型系统的设想与实际尝试，反而要比今天大家所了解的大型单体系统出现的时间更早”实际并无矛盾。

```
Monolithic Application

Monolith means composed all in one piece. The Monolithic application describes a single-tiered software application in which different components combined into a single program from a single platform.

单体意味着自包含。单体应用描述了一种由同一技术平台的不同组件构成的单层软件。
—— [Monolithic Application](https://en.wikipedia.org/wiki/Monolithic_application)，Wikipedia

```

尽管“Monolithic”这个词语本身的意思“巨石”确实带有一些“不可拆分”的隐含意味，但人们也不应该简单粗暴地把单体系统在维基百科上的定义“all in one piece”翻译成“铁板一块”，它其实更接近于“自给自足”（Self-Contained，计算机术语是“自包含”）的含义。不过，这种“铁板一块”的译法不能全算作段子，笔者相信肯定有一部分人说起单体架构、巨石系统的缺点时，在脑海中闪过的第一个特点就是它的“不可拆分”，难以扩展，因此才不能支撑越来越大的软件规模。这种想法看似合理，其实是有失偏颇的，至少不完整。

从纵向角度来看，笔者从未见过实际生产环境里有哪个大型的现代信息系统是完全不分层的。分层架构（Layered Architecture）已是现在几乎所有信息系统建设中都普遍认可、采用的软件设计方法，无论是单体还是微服务，抑或是其他架构风格，都会对代码进行纵向层次划分，收到的外部请求在各层之间以不同形式的数据结构进行流转传递，触及最末端的数据库后按相反的顺序回馈响应，如图 1-1 所示。对于这个意义上的“可拆分”，单体架构完全不会展露出丝毫的弱势，反而可能会因更容易开发、部署、测试而获得一些便捷性上的好处。


![](./images/layed-arch.png)
图 1-1 分层架构示意<br/>
图片来自 O'Reilly 的开放文档《[Software Architecture Patterns](https://www.oreilly.com/programming/free/files/software-architecture-patterns.pdf)》

从横向角度来看，单体架构也可以支持按照技术、功能、职责等维度，将软件拆分为各种模块，以便重用和管理代码。单体系统并不意味着只能有一个整体的程序封装形式，如果需要，它完全可以由多个 JAR、WAR、DLL、Assembly 或者其他模块格式来构成。即使是以横向扩展（Scale Horizontally）的角度来衡量，在负载均衡器之后同时部署若干个相同的单体系统副本，以达到分摊流量压力的效果，也是非常常见的需求。

在“拆分”这方面，单体系统的真正缺陷不在如何拆分，而在拆分之后的隔离与自治能力上的欠缺。由于所有代码都运行在同一个进程空间之内，所有模块、方法的调用都无须考虑网络分区、对象复制这些麻烦的事和性能损失。获得了进程内调用的简单、高效等好处的同时，也意味着如果任何一部分代码出现了缺陷，过度消耗了进程空间内的资源，所造成的影响也是全局性的、难以隔离的。譬如内存泄漏、线程爆炸、阻塞、死循环等问题，都将会影响整个程序，而不仅仅是影响某一个功能、模块本身的正常运作。如果消耗的是某些更高层次的公共资源，譬如端口号或者数据库连接池泄漏，影响还将会波及整台机器，甚至是集群中其他单体副本的正常工作。

同样，由于所有代码都共享着同一个进程空间，不能隔离，也就无法（其实还是有办法的，譬如使用 OSGi 这种运行时模块化框架，但是很别扭、很复杂）做到单独停止、更新、升级某一部分代码，因为不可能有“停掉半个进程，重启 1/4 个程序”这样不合逻辑的操作，所以从可维护性来说，单体系统也是不占优势的。程序升级、修改缺陷往往需要制定专门的停机更新计划，做灰度发布、A/B 测试也相对更复杂。

如果说共享同一进程获得简单、高效的代价是同时损失了各个功能模块的自治、隔离能力，那这两者孰轻孰重呢？这个问题的潜台词似乎是在比较微服务、单体架构哪种更好用、更优秀？笔者认为“好用和优秀”不会是放之四海皆准的，这点不妨举一个浅显的例子加以说明。譬如，沃尔玛将超市分为仓储部、采购部、安保部、库存管理部、巡检部、质量管理部、市场营销部等，可以划清职责，明确边界，让管理能力能支持企业的成长规模；但如果你家楼下开的小卖部，爸、妈加儿子，再算上看家的中华田园犬小黄一共也就只有四名员工，再去追求“先进管理”，划分仓储部、采购部、库存管理部……那纯粹是给自己找麻烦。单体架构下，哪怕是信息系统中两个相互毫无关联的子系统，也依然会部署在同一个进程中。当系统规模小的时候，这是优势，但当系统规模大，或程序需要修改的时候，其部署的成本、技术升级的迁移成本都会变得更为昂贵。继续以前面的例子来比喻，当公司小时，让安保部和质检部两个不相干的部门在同一栋大楼中办公是节约资源，但当公司人数增加，办公室已经拥挤不堪，最多只能在楼顶加盖新楼层（相当于增强硬件性能）来解决办公问题，而不能让安保、质检分开地方办公，这便是缺陷所在。

由于隔离能力的缺失，单体除了难以阻断错误传播、不便于动态更新程序以外，还面临难以技术异构的困难，每个模块的代码都通常需要使用一样的程序语言，乃至一样的编程框架去开发。单体系统的技术栈异构并非一定做不到，譬如 JNI 就可以让 Java 混用 C 或 C++，但这通常是迫不得已的，并不是优雅的选择。

不过，以上列举的这些问题都还不是今天以微服务取代单体系统成为潮流趋势的根本原因，笔者认为最重要的理由是：单体系统很难兼容“[Phoenix](/introduction/about-the-fenix-project.html#架构的演进)”的特性。这种架构风格潜在的观念是希望系统的每一个部件，每一处代码都尽量可靠，靠不出或少出缺陷来构建可靠系统。然而战术层面再优秀，也很难弥补战略层面的不足，单体靠高质量来保证高可靠性的思路，在小规模软件上还能运作良好，但系统规模越大，交付一个可靠的单体系统就变得越来越具有挑战性。如本书的前言开篇《[什么是"凤凰架构"](/introduction/about-the-fenix-project.html)》所说，正是随着软件架构演进，构筑可靠系统从“追求尽量不出错”，到正视“出错是必然”的观念转变，才是微服务架构得以挑战并逐步开始取代运作了数十年的单体架构的底气所在。

为了允许程序出错，为了获得隔离、自治的能力，为了可以技术异构等目标，是继为了性能与算力之后，让程序再次选择分布式的理由。然而，开发分布式程序也并不意味着一定要依靠今天的微服务架构才能实现。在新旧世纪之交，人们曾经探索过几种服务拆分方法，将一个大的单体系统拆分为若干个更小的、不运行在同一个进程的独立服务，这些服务拆分方法后来导致了[面向服务架构](https://en.wikipedia.org/wiki/Service-oriented_architecture)（Service-Oriented Architecture）的一段兴盛期，我们称其为“[SOA 时代](/architecture/architect-history/soa.html)”。
