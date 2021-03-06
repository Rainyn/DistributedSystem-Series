![分布式计算概念图](https://i.postimg.cc/dtd6t4MP/image.png)

# 分布式计算

现实世界中的数据系统往往颇为复杂。大型应用程序经常需要以多种方式访问和处理数据，没有一个数据库可以同时满足所有这些不同的需求。因此应用程序通常组合使用多种组件：数据存储，索引，缓存，分析系统，等等，并实现在这些组件中移动数据的机制。

# 存储节点与计算节点

其实所谓分布式运算，核心的思路就是系统架构无单点，让整个系统可扩展。一般来说，分布式计算环境下的节点会分为有状态存储节点和无状态运算节点。

那么针对无状态节点，因为不存储数据，请求分发可以采取很简单的随机算法或者是轮询的算法就可以了，如果需要增加机器，那只需要把对应的运算代码部署到一些机器上，然后启动起来，引导流量到那些机器上就可以实现动态的扩展了，所以一般来说在无状态的节点的扩展是相对的容易的，唯一需要做的事情就是在某个机器承担了某种角色以后，能够快速的广播给需要这个角色提供服务的人说：“我目前可以做这个活儿啦，你们有需要我做事儿的人，可以来找我。”

而针对有状态节点，扩容的难度就相对的大一些，因为每台 Server 中都有数据，所以请求分发的算法不能够用随机或者是轮询了，一般来说常见算法就是哈希或者是使用 Tree 来做一层映射，而如果需要增加机器，那么需要一个比较复杂的数据迁移的过程，而迁移数据本身所需要的成本是非常高的，这也就直接导致有状态节点的扩容难度比无状态节点更大。

针对有状态节点的难题，我们提供了一套数据自动扩容和迁移的工具来满足用户的自动扩容缩容中所产生的数据迁移类的需求。于是，无论是有状态的数据节点的扩容，还是无状态的数据节点的自动扩容，我们都可以使用自动化工具来完成了。

Google 在 03-06 年发布了关于 GFS、BigTable、MapReduce 的三篇论文，开启了大数据时代。在发展的早期，就诞生了以 HDFS/HBase/MapReduce 为主的 Hadoop 技术栈，并一直延续到今天。

最开始大数据的处理大多是离线处理，MapReduce 理念虽然好，但性能捉急，新出现的 Spark 抓住了这个机会，依靠其强大而高性能的批处理技术，顺利取代了 MapReduce，成为主流的大数据处理引擎。
随着时代的发展，实时处理的需求越来越多，虽然 Spark 推出了 Spark Streaming 以微批处理来模拟准实时的情况，但在延时上还是不尽如人意。2011 年，Twitter 的 Storm 吹响了真正流处理的号角，而 Flink 则将之发扬光大。
到现在，Flink 的目光也不再将自己仅仅视为流计算引擎，而是更为通用的处理引擎，开始正面挑战 Spark 的地位。

# 批处理（Batch Processing）与流处理（Stream Processing）

基本的区别在于每一条新数据在到达时是被处理的，还是作为一组新数据的一部分稍后处理。这种区分将处理分为两类：批处理和流处理。

## 批处理

在批处理中，新到达的数据元素被收集到一个组中。整个组在未来的时间进行处理（作为批处理，因此称为“批处理”）。确切地说，何时处理每个组可以用多种方式来确定，它可以基于预定的时间间隔（例如，每五分钟，处理任何新的数据已被收集）或在某些触发的条件下（例如，处理只要它包含五个数据元素或一旦它拥有超过 1MB 的数据）。

![基于时间的批处理间歇过程](https://s2.ax1x.com/2019/10/03/uwHkQJ.png)

通过类比的方式，批处理就像你的朋友（你当然知道这样的人）从干衣机中取出一大堆衣物，并简单地把所有东西都扔进一个抽屉里，只有当它很难找到东西时才分类和组织它。这个人避免每次洗衣时都要进行分拣工作，但是他们需要花费大量时间在抽屉里搜索抽屉，并最终需要花费大量时间分离衣服，匹配袜子等。当它变得很难找到东西的时候。历史上，绝大多数数据处理技术都是为批处理而设计的。传统的数据仓库和 Hadoop 是专注于批处理的系统的两个常见示例。

术语 MicroBatch 经常用于描述批次小和/或以小间隔处理的情况。即使处理可能每隔几分钟发生一次，数据仍然一次处理一批。Spark Streaming 是设计用于支持微批处理的系统的一个例子。

## 流处理

流处理系统的设计是为了在数据到达时对其进行响应。这就要求它们实现一个由事件驱动的体系结构, 即系统的内部工作流设计为在接收到数据后立即连续监视新数据和调度处理。另一方面, 批处理系统中的内部工作流只定期检查新数据, 并且只在下一个批处理窗口发生时处理该数据。

流处理和批处理之间的差异对于应用程序来说也是非常重要的。为批处理而构建的应用程序，通过定义处理数据，具有延迟性。在具有多个步骤的数据管道中，这些延迟会累积。此外，新数据的到达与该数据的处理之间的延迟将取决于直到下一批处理窗口的时间--从在某些情况下完全没有时间到批处理窗口之间的全部时间不等，这些数据是在批处理开始后到达的。因此，批处理应用程序(及其用户)不能依赖一致的响应时间，需要相应地调整以适应这种不一致性和更大的延迟。
