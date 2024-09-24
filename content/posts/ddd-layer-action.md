---
title: 'DDD分层架构落地实践'
categories:
    - Tech
tags: 
    - DDD
    - Collect
link: https://mp.weixin.qq.com/s/g2xaH_mLf07fdb-4TxAMXg
author: 梁波Eric
date: '2024-09-24 11:06:38'
draft : false
---

> ## Excerpt
> 通过统一的 DDD 分层架构、规范的目录结构以及 archunit 分层框架约束，让团队成员更加清楚逻辑代码的编写位置，逐渐养成习惯。同时合理的分层以及代码逻辑隔离也能更方便地进行单元测试的编写。

---
![Image](images/d04696199bad6115ade149097896a5c1.webp)

## 前言

2021 年由本人负责的微服务项目大力推广 DDD 架构设计，推广的过程中发现徒有其形，很多内在细节做得不到位。主要是团队成员对于 DDD 的理解参差不齐，团队内的约定不全面，追求代码快速实现等因素造成。

所以决定重塑其中一个微服务，期望构建一个标准的样例供团队成员参考，并通过 Tech Huddle 的形式在团队中宣讲。

本文会做一些经验总结供大家参考，随着时间的推移在新的项目中有新的感悟，在文末提出一些问题供大家思考。

## 重塑分层架构

对项目中的分层架构进行总结和重塑，同时放入项目的 OnBoarding 文档中，便于新人上手。提取其中一张进程内架构图：

![Image](images/f99b5dc3ae7e44a476d60491122f3ed1.webp)

根据依赖倒置原则（高层模块不应该依赖于低层模块，两者都应该依赖于抽象，抽象不应该依赖于细节，细节应该依赖于抽象）。

简单来说就是基础设施层的接口定义在其他层。基础设施层只实现这些接口。

-   用户接口层：面向前端提供服务适配，面向资源层提供资源适配。这一层聚集了接口适配相关的功能。
    
-   应用层职责：实现服务组合和编排，适应业务流程快速变化的需求。这一层聚集了应用服务和事件相关的功能。
    
-   领域层：实现领域的核心业务逻辑。这一层聚集了领域模型的聚合、聚合根、实体、值对象、领域服务和事件等领域对象，以及它们组合所形成的业务能力。
    
-   基础层：为各层提供基础资源服务。这一层聚集了各种底层资源相关的服务和能力。
    

## 重塑代码结构

将项目中所有的代码进行梳理，进行整合归类，梳理出适用于当前项目合理的目录结构，并且进行代码重塑。

### 一级代码结构

<pre>
<ol><li><p><span><span><code><span>&gt;</span><span> presentation</span></code></span></span></p></li><li><p><span><span><code><span>&gt;</span><span> application</span></code></span></span></p></li><li><p><span><span><code><span>&gt;</span><span> domain</span></code></span></span></p></li><li><p><span><span><code><span>&gt;</span><span> infrastructure</span></code></span></span></p></li><li><p><span><span><code><span>&gt;</span><span> common</span></code></span></span></p></li></ol>
</pre>

这些目录的职能和代码形态是这样的。

-   **presentation（用户接口层）：**它主要存放用户接口层与前端交互、展现数据相关的代码。前端应用通过这一层的接口，向应用服务获取展现所需的数据。这一层主要用来处理用户发送的 Restful 请求，解析用户输入的配置文件，并将数据传递给 Application 层。数据的组装、数据传输格式以及 Facade 接口等代码都会放在这一层目录里。
    
-   **Application（应用层）：**它主要存放应用层服务组合和编排相关的代码。应用服务向下基于微服务内的领域服务或外部微服务的应用服务完成服务的编排和组合，向上为用户接口层提供各种应用数据展现支持服务。应用服务和事件等代码会放在这一层目录里。
    
-   **Domain（领域层）：**它主要存放领域层核心业务逻辑相关的代码。领域层可以包含多个聚合代码包，它们共同实现领域模型的核心业务逻辑。聚合以及聚合内的实体、方法、领域服务和事件等代码会放在这一层目录里。
    
-   **Infrastructure（基础设施层）：**它主要存放基础资源服务相关的代码，为其它各层提供的通用技术能力、三方软件包、数据库服务、配置和基础资源服务的代码都会放在这一层目录里。
    
-   **Common（通用层）：**它主要放一些通用的代码，如异常定义，通用的枚举类型等
    

### 各层目录结构

#### **01 用户接口层**

presentation 的代码目录结构，包括但不限于：

<pre>
<ol><li><p><span><span><code><span>&gt;</span><span> presentation</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> assembler</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> vo</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> facade</span></code></span></span></p></li></ol>
</pre>

**Assembler**：和 VO 类相关的转换，实现 VO与DTO之间的相互转换和数据交换。一般来说 Assembler 与 VO 总是一同出现。

**VO**：它是数据传输的载体，主要包含三类对象CreateXxxCommand，UpdateXxxCommand，QueryXxxRequest以及XxxResponse，分别表示创建/更新数据，请求数据，以及数据展示。

**Facade**：提供较粗粒度的调用接口Controller，将用户请求委派给一个或多个应用服务进行处理。

一般项目会同时支持多个前台应用端，所以可以**在 VO 与 Facade 层加入不同端的分层 package**。

#### **02 应用层**

Application 的代码目录结构，包括但不限于：

<pre>
<ol><li><p><span><span><code><span>&gt;</span><span> application</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> assembler</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> dto</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> </span><span>event</span></code></span></span></p></li><li><p><span><span><code><span>        </span><span>&gt;</span><span> publish</span></code></span></span></p></li><li><p><span><span><code><span>        </span><span>&gt;</span><span> subscribe</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> service</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> task</span></code></span></span></p></li></ol>
</pre>

**Assembler（对象转换）：** 和 DTO 相关的类转换，一般来说 Assembler 与 DTO 总是一同出现。

**DTO：** 用于前端与应用层或者微服务之间的数据组装和传输，是应用之间数据传输的载体。

**Event（事件）**：这层目录主要存放事件相关的代码。它包括两个子目录：publish 和 subscribe。前者主要存放事件发布相关代码，后者主要存放事件订阅相关代码（事件处理相关的核心业务逻辑在领域层实现）。

这里提示一下：虽然应用层和领域层都可以进行事件的发布和处理，但为了实现事件的统一管理，我建议你将微服务内所有事件的发布和订阅的处理都统一放到应用层，事件相关的核心业务逻辑实现放在领域层。通过应用层调用领域层服务，来实现完整的事件发布和订阅处理流程。

**Service（应用服务）**：这层的服务是应用服务。应用服务会对多个领域服务或外部应用服务进行封装、编排和组合，对外提供粗粒度的服务。应用服务主要实现服务组合和编排，是一段独立的业务逻辑。你可以将所有应用服务放在一个应用服务类里，也可以把一个应用服务设计为一个应用服务类，以防应用服务类代码量过大。

**Task（定时任务）：** 将一些定时任务相关的逻辑放到该层。

#### **03 领域层**

Domain 是由一个或多个聚合包构成，共同实现领域模型的核心业务逻辑。目录结构包括但不限于：

<pre>
<ol><li><p><span><span><code><span>&gt;</span><span> domain</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> shared</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> model</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> aggregate00</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> entity</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> </span><span>event</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> repository</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> service</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> util</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> aggregate01</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> entity</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> </span><span>event</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> repository</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> service</span></code></span></span></p></li><li><p><span><span><code><span>      </span><span>&gt;</span><span> util</span></code></span></span></p></li></ol>
</pre>

而领域层聚合内部的代码目录结构是这样的。

**Shared（共有抽象包）**：它定义了一些基础的抽象接口与实现类

**Aggregate（聚合）**：它是聚合软件包的根目录，可以根据实际项目的聚合名称命名，比如权限聚合。在聚合内定义聚合根、实体和值对象以及领域服务之间的关系和边界。聚合内实现高内聚的业务逻辑，它的代码可以独立拆分为微服务。

以聚合为单位的代码放在一个包里的主要目的是为了业务内聚，而更大的目的是为了以后微服务之间聚合的重组。聚合之间清晰的代码边界，可以让你轻松地实现以聚合为单位的微服务重组，在微服务架构演进中有着很重要的作用。

**Entity（实体**）：它存放聚合根、实体、值对象以及工厂模式（Factory）相关代码。实体类采用充血模型，同一实体相关的业务逻辑都在实体类代码中实现。跨实体的业务逻辑代码在领域服务中实现。

**Event（事件）**：它存放事件实体以及与事件活动相关的业务逻辑代码。

**Service（领域服务）**：它存放领域服务代码。一个领域服务是多个实体组合出来的一段业务逻辑。你可以将聚合内所有领域逻辑都放在一个聚合领域服务类中。如果你的业务系统非常复杂，避免由于所有领域服务代码都放在一个聚合领域服务类中，而出现代码臃肿的问题，可考虑进行适当的拆分。

**Repository（仓储）**：它存放所在聚合的查询或持久化领域对象的代码，通常包括仓储接口和仓储实现方法。为了方便聚合的拆分和组合，我们设定了一个原则：一个聚合对应一个仓储。

#### **04 基础设施层**

Infrastructure 的代码目录结构，包括但不限于：

<pre>
<ol><li><p><span><span><code><span>&gt;</span><span> infrastructure</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> config</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> persistence</span></code></span></span></p></li><li><p><span><span><code><span>        </span><span>&gt;</span><span> entity</span></code></span></span></p></li><li><p><span><span><code><span>        </span><span>&gt;</span><span> converter</span></code></span></span></p></li><li><p><span><span><code><span>        </span><span>&gt;</span><span> repository</span></code></span></span></p></li><li><p><span><span><code><span>    </span><span>&gt;</span><span> integration</span></code></span></span></p></li></ol>
</pre>

**Config：**主要存放配置相关代码。

**Persistence：**主要存放数据库持久化相关的代码，用于实现 Domain 领域层中 Repository 文件夹下的接口。

**Integration：**主要存放第三方系统的集成。

## 梳理模型转换

我们先来看一下微服务内有哪些类型的数据对象？它们是如何协作和转换的？

-   数据持久化对象 PO (Persistent Object)，与数据库结构一一映射，是数据持久化过程中的数据载体。
    
-   领域对象 DO（Domain Object），微服务运行时的实体，是核心业务的载体。
    
-   数据传输对象 DTO（Data Transfer Object），用于前端与应用层或者微服务之间的数据组装和传输，是应用之间数据传输的载体。
    
-   视图对象 VO（View Object），用于封装展示层指定页面或组件的数据。
    

![Image](images/2df1aabb0e56aab5a5304018a460c429.webp)

总结如下：

![Image](images/265c62cf7f167ebaf233726bba113d73.webp)

## 小结

通过统一的 DDD 分层架构、规范的目录结构以及 archunit 分层框架约束，让团队成员更加清楚逻辑代码的编写位置，逐渐养成习惯。同时合理的分层以及代码逻辑隔离也能更方便地进行单元测试的编写。

## 思考

在项目中落地 DDD 的同时，很多开发用起来会很不顺手，甚至抱怨，有如下几点：

1.  **数据对象转换多：**不同类型的数据对象都是相同的属性字段，需要一直拷贝以及编写转换器，比如：从 PO --> DO --> DTO --> VO。
    
2.  **领域逻辑下沉：** 过程式编程深入人心，也非常符合人性的习惯，第一步 xx，第二步 xx，最后 xx，所以很多业务代码都会写到应用层的Application Service 中，而不是领域层；就算有开发有意识的写到领域层，也分不清该写在充血 Model 中还是 Domain Service 中。
    
3.  **数据访问变扭：** 现在出现很多 ORM，比如：JPA、MyBatis等，直接定义接口就可以访问数据库。比如：使用 JPA，你需要先在 Domain 层中定义接口，然后再 infrastructure 层中再定义接口并实现 domain 层的接口。
    
4.  **多个 ORM 框架混用：**甚至有的项目采用多种 ORM 框架结合使用，没有规范和约束，新人上项目产生困惑。
    
5.  **外部集成 Client 实现规范不明确：** 有的实现在 application 层，有的实现在 infrastructure 层，无法统一管理外部集成代码
    
6.  **查询性能和编码效率低：** 一般项目中有大量的查询操作，需要编写至少 4 层代码，尤其是对象转换，导致开发人员经常质疑 DDD 分层是否合理。
    

选择合理的应用分层架构最终是为了提高开发效率，减少代码的腐化，建议不要刻板的去执行，进行适当的调整，比如 **简单读操作**（这在业务系统中非常多，包括分页列表、详情页等等），没有复杂的多个聚合或实体的整合，可以尝试直接跳过 Domain。列举一个简单流程：  

> **流程：**Controller --> Application（Application Service --> Query Executor） --> infrastructure（Mybatis Mapper）
> 
> ___
> 
> **输入：**QueryCommand --> Mybatis ExampleDO
> 
> ___
> 
> **输出：**DataObject --> DTO (DetailVO 和 Application DTO 进行合并，没有 DetailVO 对象，直接使用 DTO)

## 推荐阅读

[应用分层架构最佳实践：Alibaba COLA 4.0](https://mp.weixin.qq.com/s?__biz=Mzk0NDI1NzI2Mw==&mid=2247485055&idx=1&sn=e32bd459769022f55c9ec2f1a24a3a2c&chksm=c32627fff451aee9708373be5a9d0a808507e78659b3e2c0ce6fab7c11e783f2e87da23dc2ee&scene=21#wechat_redirect)

## 附录

完整代码结构

![Image](images/e5148e631538259b6e984226874ed54a.webp)

分层架构约束

项目中引入 archunit 依赖。

![Image](images/9f91cdba3283c3ec5e823bd94b5e6a74.webp)

### DDD基础分层图

![Image](images/da78c5159d8b787d8741206698211aa3.webp)

### 洋葱架构图

![Image](images/e88f6b75f817b8d8adee1af077dc1787.webp)

### 整洁架构图

![Image](images/aaaa9fc4a40512065ef3098928576686.webp)

### 六边形架构图

![Image](images/caa54c32375dd88eb7d8a0f3a7e5e50c.webp)

### Alibaba COLA 4.0 架构图

![Image](images/4fbfea537ec69ddfabb9e9c3e93df6c4.webp)

![Image](images/9ce79a74a6afc4ac91af55344fa4972e.webp)
