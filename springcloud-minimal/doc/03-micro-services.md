# 微服务概述

### 为什么要使用微服务？

传统的 Java Web 都是采用单体架构的方式来进行开发、部署、运维的，所谓**单体架构就是将 Application 的所有业务模块全部打包在一个文件中进行部署**。但是随着互联网的发展、用户数量的激增、业务的极速扩展，传统的单体应用方式的缺点就逐渐显现出来了，给开发、部署、运维都带来了极大的难度，工作量会越来越大，难度越来越高。

因为在单体架构中，所有的业务模块的耦合性太高，耦合性过高的同时项目体量又很大势必会给各个技术环节带来挑战。项目越进行到后期，这种难度越大，只要有改动，整个应用都需要重新测试，部署，极大的限制了开发的灵活性，降低了开发效率。同时也带来了更大的安全隐患，如果某个模块发生故障无法正常运行就有可能导致整个项目崩溃，单体应用的架构如下图所示。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gginbnf5vaj311o0u00vy.jpg" alt="1" style="zoom: 35%;" />

#### 单体应用存在的问题

- 随着业务的发展，开发变得越来越复杂。
- 修改、新增某个功能，需要对整个系统进行测试，重新部署。
- 一个模块出现问题，很可能导致整个系统崩溃。
- 多团队同时对数据进行管理，容易产生安全漏洞。
- 各模块使用同一种技术框架，很难根据具体业务需求选择更合适的框架，局限性太大。
- 模块内容太复杂，如果员工离职，可能需要很长时间才能完成任务交接。

为了解决上述问题，微服务架构应运而生，简单来说，微服务就是将一个单体应用拆分成若干个小型的服务，协同完成系统功能的一种架构模式，在系统架构层面进行解耦合，将一个复杂问题拆分成若干个简单问题。这样的好处是对于每一个简单问题，开发、维护、部署的难度就降低了很多，可以实现自治，可以自主选择最合适的技术框架，提高了项目开发的灵活性。

微服务架构不仅是单纯的拆分，拆分之后的各个微服务之间还要进行通信，否则就无法协同完成需求，也就失去了拆分的意义。不同的微服务之间可以通过某种协议进行通信，相互调用、协同完成功能，并且各服务之间只需要制定统一的协议即可，至于每个微服务是用什么技术框架来实现的，统统不需要关心。这种松耦合的方式使得开发、部署都变得更加灵活，同时系统更容易扩展，降低了开发、运维的难度，单体应用拆分之后的架构如下图所示。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gginbmv3baj311l0u0td7.jpg" alt="2" style="zoom:35%;" />

### 微服务的优点

- 各个服务之间实现了松耦合，彼此之间不需要关注对方是用什么语言，什么技术开发的，只需要保证自己的接口可以正常访问，通过标准协议可以访问其他微服务的接口即可。
- 各个微服务之间是独立自治的，只需要专注于做好自己的业务，开发和维护不会影响到其他的微服务，这和单体架构中"牵一发而动全身"相比是有很大优势的。
- 微服务是一个去中心化的架构方式，相当于用零件去拼接一台机器，如果某个零件出现问题，可以随时进行替换从而保证机器的正常运转，微服务就相当于零件，整个项目就相当于零件组成的机器。

任何一种架构方式都有其优势，同时也一定会存在不足，我们需要做的是根据具体的业务场景，选择最合适的架构来进行开发。

### 微服务的不足

- 各个服务之间是通过远程调用的方式来完成协作任务的，如果因为某些原因导致远程调用出现问题，导致微服务不可用，就有可能产生级联反应，造成整个系统崩溃。
- 如果某个需求需要调用多个微服务，如何来保证数据的一致性是一个比较大的问题，这就给给分布式事务处理带来了挑战。
- 相比较于单体应用开发，微服务的学习难度会增加，对于加入团队的新员工来讲，如何快速掌握上手微服务架构，是他需要面对的问题。

即便是微服务存在这样那样的问题，也不能掩盖这种技术选型在当今互联网环境下的巨大优势，了解完微服务的基本概念、优缺点之后，我们来谈一谈微服务的设计原则。

我们说过微服务就是对应用进行拆分，在软件设计层面进行解耦合，说起来容易做起来难，如何来拆分服务也是一个有难度的挑战，如果拆分的服务粒度太小，导致微服务可完成的功能有限，这种情况反而会增加系统的复杂度。相反如果服务粒度太大，又没有实现足够程度的解耦合，那它本质上就还是单体应用的架构，同样是失去了拆分服务的意义。比较合理的拆分方式是由大到小，提炼出核心需求，搞清楚服务间的交互关系，先拆分成粒度相对较大的服务，然后再根据具体的业务需求逐步细化服务粒度，最终形成一套合理的微服务体系。

### 微服务设计原则

- **服务粒度不能过大也不能过小**，提炼核心需求，根据服务间的交互关系找到最合理的服务粒度。
- **各个微服务的功能职责尽量单一**，避免出现多个服务处理同一个需求。
- **各个微服务之间要相互独立**、自治，自主开发、自主部署、自主维护。
- **保证数据独立性**，各个微服务独立管理其业务模块下的数据，可开放出接口供其他微服务访问数据，但是其他微服务不能直接管理这些数据。
- **使用 REST 协议完成微服务之间的协作任务**，数据交互采用 JSON 格式，方便调用与整合。

综上所述，微服务的架构设计不是一次成型的，需要反复进行实践和总结，不断进行优化，最终形成一套更为合理的解决方案，微服务架构图如下所示。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gginbm1t7uj31vi0sugsh.jpg" alt="3" style="zoom:35%;" />

### 微服务架构的核心组件

- 服务治理（服务注册、服务发现）

  拆分之后的微服务首先需要完成的工作就是实现服务治理，包括服务注册和服务发现。这里我们把微服务分为两类：提供服务的叫做服务提供者，调用服务的叫做服务消费者。一个**服务消费者首先需要知道有哪些可供调用的微服务**，以及如何来调用这些微服务。所以就需要将所有的**服务提供者在注册中心完成注册，记录服务信息**，如 IP 地址、端口等，然后服务消费者可以通过服务发现获取服务提供者的这些信息，从而实现调用。

- 服务负载均衡

  微服务间的负载均衡是必须要考虑的，服务消费者在调用服务提供者的接口时，可根据配置选择某种负载均衡算法，**从服务提供者列表中选择具体要调用的实例**，从而实现服务消费者与服务提供者之间的负载均衡。

- 微服务网关

  在一个微服务架构中会包含多个微服务实例，每个微服务实例都有其特定的网络信息，如 IP 地址、端口等，很多时候完成一个需求需要多个微服务来协同工作，那么客户端就需要频繁请求不同的 URL ，不便于统一管理，可以使用微服务网关来解决这一问题，**为客户端提供统一的入口**，系统内部由网关将请求映射到不同的微服务。

- 微服务容错

  前面我们提到过，各个服务之间是通过远程调用的方式来完成协作任务的，如果因为某些原因使得远程调用出现问题，导致微服务不可用，就有可能产生级联反应，造成整个系统崩溃。这个问题我们可以使用微服务的熔断器来处理，熔断器可以**防止整个系统因某个微服务调用失败而产生级联反应导致系统崩溃**的情况。

- 分布式配置

  每个微服务都有其对应的配置文件，在一个大型项目中管理这些配置文件也是工作量很大的一件事情，为了提高效率、便于管理，我们可以对各个**微服务的配置文件进行集中统一管理**，将配置文件集中保存到本地系统或者 Git 仓库，**再由各个微服务读取自己的配置文件**。

- 服务监控

  一个分布式系统中往往会部署很多个微服务，这些服务彼此之间会相互调用，整个过程就会较为复杂，我们在进行问题排查或者优化的时候工作量就会比较大。如果我们能**准确跟踪到每一个网络请求，了解它整个运行流程，经过了哪些微服务、是否有延迟、耗费时间**等，这样的话我们**分析系统性能，排查解决问题**就会容易很多，这就是服务监控。

### Spring Cloud

微服务是一种分布式软件架构设计方式，具体的落地框架有很多，如阿里的 Dubbo、Google 的 gRPC、新浪的 Motan、Apache 的 Thrift 等，都是基于微服务实现分布式架构的技术框架，本课程我们选择的框架的是 Spring Cloud，Spring Cloud 基于 Spring Boot 使得整体的开发、配置、部署都非常方便，可快速搭建基于微服务的分布式应用，Spring Cloud 相当于微服务各组件的集大成者，下图来自 Spring 官网。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gginbl5ltlj31js0rkwl3.jpg" alt="3" style="zoom: 25%;" />

Spring Boot 和 Spring Cloud 的关系可大致理解为，**Spring Boot 快速搭建基础系统**，**Spring Cloud 在此基础上实现分布式系统中的公共组件**，如服务注册、服务发现、配置管理、熔断器、控制总线等，**服务调用方式是基于 REST API**，整合了各种成熟的产品和架构。

对于 Java 开发者而言，Spring 框架已成为事实上的行业标准，Spring 全家桶系列产品的优势在于功能齐全、简单好用、性能优越、文档规范，实际开发中就各方面综合因素来看，Spring Cloud 是微服务架构中非常优秀的实现方案，本课程就将为各位读者详细解读如何快速上手 Spring Cloud，Spring Cloud 的核心组件如下图所示，我们会按照图中分布来逐一讲解各个组件的使用，最后会结合一个实战案例将各个组件进行串联，让读者可以将 Spring Cloud 运用在实际开发中。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gginbme86mj31800o4n02.jpg" alt="4" style="zoom:35%;" />

### 总结

本节课作为 Spring Cloud 阶段的开篇内容，主要讲解了**微服务应用与传统单体应用的区别**，**微服务应用的优势**，以及我们**为什么要使用微服务**，实现微服务架构的解决方案有很多，我们选择 Spring 全家桶的 Spring Cloud 作为实现方案，Spring Cloud 也是当前最火、讨论热度最高的技术栈。