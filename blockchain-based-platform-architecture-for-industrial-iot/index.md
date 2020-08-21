# Blockchain-Based Platform Architecture for Industrial IoT


Author：Nikolay Teslya; Igor Ryabchikov

Published in：2017 21st Conference of Open Innovations Association(FRUCT)

Date of Conference: 6-10 Nov. **2017**

会议举办地：Helsinki, Finland

被引量：11次

## 摘要

机器人技术，物联网概念，大数据处理技术，自动化和分布式数字分类账技术的发展引发了第四次工业革命。新工业的主要问题之一是基于物联网的智能工厂内部组件以及工厂之间的互操作。这种互操作应该提供物联网参与者之间的信任;控制资源的分配（如维护时间，能源等）和成品。本文描述了集成物联网和区块链技术以解决这些问题的可能方法之一。为此，开发了一种结合了Smart-M3信息共享平台和区块链平台的架构。所提出的体系结构的一个主要特征是使用智能合约来处理和存储与智能空间组件之间的交互相关的信息。

<!--more-->

文章有用的地方并不多，引言部分和Blockchain Platforms Overview for Industrial IoT purpose有部分重复，其它基本无用，文章主体内容-结合Smart-M3和区块链的架构并不是很适合，主要是其重点在于智能合约，并且两个平台的结合依靠中间件，由此会带来各种开销，Multichain的说明文档中对此有过分析，所以，有珠玉在前，不再细看这个架构了，下面只是把文章中一些可用的资料摘录出来。

## IIoT概念的应用场景

1. 按需生产。接受并自动完成买方订单，可生产高度定制化的商品（例如3D打印或计算机数字控制）。工厂将通过区块链网络跟踪订单。
2. 通过供应链跟踪商品。将商品数字化并存储其信息，包括生产细节，谁以及何时拥有该商品，维修记录等。通过这些信息我们可以追踪商品在供应链中的位置，从一批缺陷产品中识别商品，确认商品许可等。
3. 生产产品的机器间的自动交互。将产品准备信息交付下一生产阶段，不同的企业完成不同的生产阶段。

## IIoT对平台的需求

1. 安全。保证参与者能完整的发布他想发布的信息，不被别人中途篡改。
2. 容错。参与者信息系统的中断应当只影响其自身的进程。别人的进程不受影响。
3. 持久。一旦发布，所有参与者都可访问这些信息。
4. 公共访问的可能性。一些信息应当被所有参与者平地的看到。
5. 共识的可能性。指的是智能合约。

考虑IIoT的情况，还可能提出如下附加要求：

1. 能够过滤作者所要求的信息。在IIoT中，共同智能空间的参与者可以是独立的企业，它们必须在其中共存并拥有发布信息的平等权利。例如，每个人都有可能说：“莫斯科正在下雨”，但在某些情况下，对于某些问题并非所有发言都可信。因此，智能空间的要求之一是能够过滤作者所请求的信息。例如，根据州气象服务，“现在莫斯科的天气怎么样？”
2. 将信息标记为不相关并在查询中对其进行过滤。在智能空间中发布的信息可能失去相关性（例如，关于产品的当前所有者的信息），但不应基于耐久性要求移除信息。因此，需要一种机制/协议，根据该机制/协议可以将信息标记为不相关，以及在查询中过滤此类信息的机制，允许指定诸如“谁是该产品的当前所有者？”之类的表达。

## 一些讨论

区块的不可变和交易记录的存储不仅仅是优点，也是一种缺点。不断增长的区块链需要较大的存储，但这很难由物联网中的简单设备提供。假设该问题可由更强大的设备来处理，那么弱能力设备可以通过区块链将它们的信息和功能委托给智能空间中的其它设备。

另一个问题是输入信息的延迟，区块的形成和验证都需要时间。这个问题可通过选择和配置环境来解决，使得形成新块的时间尽可能短。

