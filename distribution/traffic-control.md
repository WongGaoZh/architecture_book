# 流量控制
任何一个系统的运算、存储、网络资源都不是无限的，当系统资源不足以支撑外部超过预期的突发流量时，便应该要有取舍，建立面对超额流量自我保护的机制，这个机制就是微服务中常说的“限流”。

## 提出思考 : 限流需要解决那三个问题呢?

+ **依据什么限流？**：要不要控制流量，要控制哪些流量，控制力度要有多大，等等这些操作都没法在系统设计阶段静态地给出确定的结论，必须根据系统此前一段时间的运行状况，甚至未来一段时间的预测情况来动态决定。
+ **具体如何限流？**：解决系统具体是如何做到允许一部分请求能够通行，而另外一部分流量实行受控制的失败降级，这必须了解掌握常用的服务限流算法和设计模式。
+ **超额流量如何处理？**：超额流量可以有不同的处理策略，也许会直接返回失败（如 429 Too Many Requests），或者被迫使它们进入降级逻辑，这种被称为否决式限流。也可能让请求排队等待，暂时阻塞一段时间后继续处理，这种被称为阻塞式限流。

## 流量统计指标
要做流量控制，首先要弄清楚到底哪些指标能反映系统的流量压力大小。

+ 每秒事务数（Transactions per Second，TPS）：TPS 是衡量信息系统吞吐量的最终标准。“事务”可以理解为一个逻辑上具备原子性的业务操作。譬如你在 Fenix's Bookstore 买了一本书，将要进行支付，“支付”就是一笔业务操作，支付无论成功还是不成功，这个操作在逻辑上是原子的，即逻辑上不可能让你买本书还成功支付了前面 200 页，又失败了后面 300 页。
+ 每秒请求数（Hits per Second，HPS）：HPS 是指每秒从客户端发向服务端的请求数（请将 Hits 理解为 Requests 而不是 Clicks，国内某些翻译把它理解为“每秒点击数”多少有点望文生义的嫌疑）。如果只要一个请求就能完成一笔业务，那 HPS 与 TPS 是等价的，但在一些场景（尤其常见于网页中）里，一笔业务可能需要多次请求才能完成。譬如你在 Fenix's Bookstore 买了一本书要进行支付，尽管逻辑上它是原子的，但技术实现上，除非你是直接在银行开的商城中购物能够直接扣款，否则这个操作就很难在一次请求里完成，总要经过显示支付二维码、扫码付款、校验支付是否成功等过程，中间不可避免地会发生多次请求。
+ 每秒查询数（Queries per Second，QPS）：QPS 是指一台服务器能够响应的查询次数。如果只有一台服务器来应答请求，那 QPS 和 HPS 是等价的，但在分布式系统中，一个请求的响应往往要由后台多个服务节点共同协作来完成。譬如你在 Fenix's Bookstore 买了一本书要进行支付，尽管扫描支付二维码时客户端只发送了一个请求，但这背后服务端很可能需要向仓储服务确认库存信息避免超卖、向支付服务发送指令划转货款、向用户服务修改用户的购物积分，等等，这里面每次内部访问都要消耗掉一次或多次查询数。

以上这三个指标都是基于调用计数的指标，在整体目标上我们当然最希望能够基于 TPS 来限流, 但是之间针对TPS的限流实际是很难操作的,
所以目前主流系统大多倾向使用 HPS 作为首选的限流指标，它是相对容易观察统计的，而且能够在一定程度上反应系统当前以及接下来一段时间的压力。

## 限流设计模式

+ 流量计数器模式
+ 滑动时间窗模式
+ 漏桶模式
+ 令牌桶模式






