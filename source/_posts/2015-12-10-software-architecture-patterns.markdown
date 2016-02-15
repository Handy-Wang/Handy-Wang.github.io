---
layout: post
title: "软件架构模式（译）"
date: 2015-12-10 16:10:37 +0800
comments: true
categories: 
---
原文：[《Software Architecture Patterns》](http://www.oreilly.com/programming/free/files/software-architecture-patterns.pdf)

<br/>

#前言

对于很多开发人员来说，在着手开发时并没有一套有条理的架构体系来指导他们，这简直是一件太正常的事情了。在没有一套清晰的、经过推敲过的架构体系时，许多开发人员和架构师都遵守所谓的行业标准－分层架构模式（也称之为多层架构），即创建隐式的多层结构，即把只是从源代码角度，把模块的源代码都放在不同的源代码路径下，看似像分层了。但，实际上这种层次架构有诸多问题：模块角色没有清晰地划分，模块职责不明确，模块间的关系不清楚。这种架构通常称之为：[大泥球](http://www.cin.ufpe.br/~sugarloafplop/mud.pdf)-架构反模式

缺少有条理的架构的应用通常都是紧耦合的、不健壮的、难以维护的，而且没有清晰的视野感和方向感。所以，非常难以确定这个应用的架构特点，以至于不能全面地理解这个应该系统中各模块的内部逻辑。很难找到与开发和维护相关的诸多基本问题的答案：这个应用具有伸缩性吗？这个应用有什么性能特点？这个应用是否很容易维护？这个应用有什么技术特性？

诸多架构模式可以辅助我们定义应用程序的基本特性和行为。例如，某些架构模式偏向于高可伸缩性，某些架构模式偏向于高度敏捷性。所以，在选择一种适合于某个项目的架构之前，了解不同架构模式的特点、优点和缺点是非常有必要的。

作为一名架构师，你需要经常检验你的架构决策，尤其是在选择某种特别的架构或方案的时候。

<!--more-->
<br/>
#第一章 分层架构模式

最常见的架构模式是分层架构模式，也称之为多层架构模式。它是一种适用于大多数`Jave EE`应用的行业标准，被很多架构师、设计师、攻城师所熟知；它与众多公司传统方式的技术交流和组织结构很对口，从而成为众多业务开发工作的首选。

##模式介绍
分层架构模式采用横向划分的方式把的各组件划分到相应的层级里面，每层结构在应用里都担任着特定的角色（比如，视图层负责UI展示、业务逻辑层负责业务逻辑处理）。虽然在分层架构模式的理念里没有强制要求分几层，但是一般分为四层：**展现层、业务层、持久化层和数据库层**（`如图Figure 1-1.`）。在某些情况下，业务层和持久化层统一合并到业务层，尤其是在持久化逻辑代码（比如：SQL或HSQL）被直接写在业务层的各组件中的时候。所以，一般小型应用只有三层架构，而大型的或更复杂的应用则会有五层或更多层架构。

每层架构在应用中都有着特定的角色和职责。例如，展现层是负责处理处所有界面显示和用户浏览时的逻辑，而业务层是充当处理业务请求相关事宜的角色。每层架构都是围绕它需要处理的特定业务逻辑的本职工作进行抽象的。例如，展现层不需要也不用担心将要显示的数据是从哪儿来的，它只需要负责把得到的数据按一定样式显示正确就行。同理，当业务层从某个地方获取到业务数据后，不需要关心这些数据以什么样的样式或形态显示到屏幕后，它只需要关注从持久层获取数据并对这些按一定的逻辑规则加工就好（比如，数据计算或数据整理工作），然后把最终的数据传递给展示层。

![Figure 1-1](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/software_architecture_patterns_figure1_1.png "Figure 1-1")

其中，分层架构的重要特性之一是各层架构中的组件分工明确。即，每个组件只会处理它所属那层架构的工作。例如，展现层中的组件只会处理展示逻辑，业务层中的组件只会处理业务逻辑。这种对组件的分门别类就构建成了角色明确、职责分明的分层架构，以及易于开发、测试、管理和维护的应用。所以，各层架构中各个组件的接口功能和作用域的良好定义就显示尤其重要了。

##关键理念

**`封闭性`**－注意`图Figure 1-2`中，每层架构的右侧都有一个“CLOSED”标签。这是分层架构中其中一个非常重要的理念－封闭。封闭的分层表示：当请求在多层间传递时，它要经过多层传递才会传递到目标层。例如，从展现层发起一个请求，这个请求首先需要经过业务层，再经过持久化层，才能最终到达数据库层。

![Figure 1-2](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/software_architecture_patterns_figure1_2.png "Figure 1-2")

然而，为什么不允许展现层直接访问其它的各层（持久层或数据库层）呢？因为按常规思维，展现层直接访问数据库层要比经过多层传递后才能查询或保存数据的效率要高得多呀，为什么呢？这个答案就存在于另一个关键理念之中－每层架构的隔离性。

**`隔离性`**－每层架构的隔离性是指每层内部的修改不会对其它层造成任何影响：层内的任何改变都被隔离于其它层及相关层(比如，含有SQL的持久层)之外。如果允许展现层直接访问持久层，那么对持久层SQL的修改就会同时影响到业务层和展现层，这样做显示使得应用的各层之间的组件都产生了各种复杂依赖、紧耦合。这样的架构非常难于维护。

隔离还意味着分层架构的每层都是独立的，因此每层都不感知其它层内的逻辑细节。为了能很好的理解隔离理念的强大和重要，那么试想一下这种情况：

	在花九牛二虎之力把展现层的技术方案由JSP重构为JSF时，只要展示层与业务层之间的对接协议保持不变(例如，使用某个Model数据结构),
	那么业务层是不会受展现层技术方案重构影响的，而且其它层更是完全独立于展现层的界面技术方案的。

****相关知识点：****JSP[^1]、JSF[^2]<br/><br/>
[^1]: [JSP详细资料](https://docs.oracle.com/javaee/5/tutorial/doc/bnagx.html)
[^2]: [JSF详细资料](http://docs.oracle.com/javaee/6/tutorial/doc/bnaph.html)

按以上介绍，当具备封闭性的架构层促成了隔离性的架构层以及层与层之间的修改能达到隔离时，那么是时候可以合理地使一些架构层开放了。例如：你可以给业务层增加一个其内部组件可以访问的且包含了很多通用组件的共享服务层（其内包含了数据、字符串工具类或辅助类和日志类）。其实，创建一个服务层一直是一个非常好的设计，因为这个设计从架构上限定：业务层可以访问共享层，而不是展现层可以访问共享层。如果没有这种分层的架构设计，那么将完全不能从架构上来阻止展现层对共享服务层的访问，而且很难管理这种访问受限的情况。

在上面这个为业务层开放了一个共享服务层的例子中，新的服务层应该处于业务层的下面以表明展现层是不能直接访问共享层的。但是，这样的层次结构有一个问题，即：业务层必须要经过共享服务层才能访问到持久化层，如果真是这样的话，就不合理了。这是分层架构一直存在的一个老问题，不过已经通过创建**具有开放特性的架构层**的方式解决了。

如`图Figure 1-3`所示，图中的服务层右侧有一个“开放”的绿色标签，说明这一层是具有有开放性的，即上层来的请求可以绕开它，从而请求可以直接被传递到开放层的下一层。参考下图示意，就可以解决刚才的那个问题了，即：业务层专属的共享层可以设计为开放层，然后业务层在需要访问持久化层时可以绕开共享，这样就相当合理了吧，嘿嘿。

![Figure 1-3](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/software_architecture_patterns_figure1_3.png "Figure 1-3")

基于具有开、闭特性的分层架构很好地诠释了各个层与请求流程的关系，同时也为架构师、工程师很好地理解各层间的访问约束提供了必要的信息。要是不能与架构中具有开、闭特性的各层合理对接，那么肯定会使得整个架构内部是紧耦合的、不健壮的，而且非常难以测试、维护和开发。

##模式实例

图`Figure 1-4`呈现了分层架构的工作原理，即，一个业务员查询客户数据的流程。图中的黑色箭头指示了由上至下获取客户数据的过程，红色箭头指示了数据返回并最终展示的过程。其中，客户数据除了包含客户资料还包含了客户自己产生的订单数据。

**详细分析**

* `Customer Screen`模块负责接收业务员的请求以及显示查询到的客户数据。它并不感知需要查询几个数据库才能最终获取到客户数据，而是只需要负责接收业务员的查询请求，并把请求传递给`Customer Delegate`模块。
* `Customer Delegate`模块负责感知业务层中哪些模块可以处理来自于`Customer Screen`的请求，以及如何与业务层中的相应模块(`Customer Object`)对接（接口协议）。
* `Customer Object`模块负责整合展现层需要的客户数据。它不但要请求持久层里客户资料对应的模块(`Customer DAO`)，还要请求订单数据对应的模块(`Order DAO`)。
* `Customer DAO`模块和`Order DAO`依次执行SQL语句来对数据库进行查询，获取到相应数据后回传给业务层的`Customer Object`模块。一旦`Customer Object`把客户资料和订单数据都获取到，就会整合两部分数据并把整合后的数据回传给`Customer Object`，从而进一步回传给`Customer Screen`模块，最终把查询到的数据展现给业务员。

![Figure 1-4](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/software_architecture_patterns_figure1__4.png "Figure 1-4")

从技术实现的角度来看，这些模块有很多不同的技术实现方案。

比如，
	
	在Java平台，Customer Screen可以采用JSF(Java Server Faces)方案，并由Customer Delegate来管理JSF的数据对象。
	业务层的Customer Object可以采用Spring[^]管理的数据对象或EJB3的数据对象来实现。
	持久化层的DAO层(Data Access Objects)可以采用POJO（Plain Old Java Objects）与MyBatis
	或JDBC或Hibernate组合实现。
	
	从微软平台的角度来制定技术实现方案时，Customer Screen模块可以使用.NET框架中的ASP(Active Server Pages)
	来访问业务层里以C#语言开发的子模块，而持久化层的子模块则采用ADO方案（ActiveX Data Objects）。
	
****相关知识点：****Spring[^3]、EJB[^4]<br/><br/>
[^3]: [Spring详细资料](http://www.oracle.com/technetwork/developer-tools/eclipse/springtutorial-087561.html)
[^4]: [EJB详细资料](https://docs.oracle.com/cd/E24329_01/web.1211/e24446/ejbs.htm#INTRO255)

##架构考量

分层架构是一个很稳定的通用架构模式，它可以作为很多应用的一个初始加构，尤其是在你不确定什么的加构适合你的项目的时候。尽管如此，当你在选择此架构模式时，仍然需要注意几点：

**首先**，需要注意的是`污水池反模式`。它描述了这样一种场景，即一个请求经过多层传递却没有执行任何逻辑操作。例如，展现层接受了用户查询客户数据的请求后，然后传递给业务层，再传递到持久化层，持久化层组装相应的查询SQL并传递到数据库层去获取数据。最后，从数据库查询到的数据没有经过任何处理（如整合、运算、数据转换）就一层一层地又回传给展现层。

采用分层架构的应用或多或少地有被沦陷为`污水池反模式`的模块。但是，重要的还是分析应用中沦陷为`污水池反模式`的请求的占比。`28原则`[^5]一直是一个用于检验你的项目是否遭遇了`污水池反模式`的很好的实践，即大概20%的请求只在层与层之间被简单传递，大概80%的请求不仅在层与层被传递同时还有一些相应的逻辑处理。但是，如果你发现这个比率是反过来的，即20%的请求在层间传递且逻辑处理，80%的请求只是在层间传递，那么，这时你就应该考虑做一些开放某些层的工作了，这个工作比管理有隔离缺陷的层的工作更难。

[^5]:[28原则资料](http://baike.baidu.com/view/40591.htm?fromtitle=%E4%BA%8C%E5%85%AB%E5%8E%9F%E5%88%99&fromid=3689905&type=syn)

**其次**，需要考虑的是分层架构模式有把自己大一统的趋势。尽管你已把展现层和业务层拆分成了不同的模块。分层架构的各层代码都归类到了不同的项目开发目录。对于某些应用来说，它并不关注这点，但是这个问题确实会导致开发阶段的很多潜在的问题、健壮性问题、可靠性问题、性能问题以及扩展性问题。

##模式分析
下面的表格中是对分层架构的一些常规特点的评级和分析。每个特性的评级高低取决于它在架构的典型实现方案中能否作为自然趋势的能力，以及它能否能作为分层架构的主要强项。关于几种架构模式的整体比较结果，请参看附录A。

**整体的敏捷性**

	等级：低
	理由：应用程序整体的敏捷性是一种对不断变化的需求快速响应的能力。尽管分层架构的设计会把模块的变化隔离在它对应的层里，但是
	分层架构对这种变化的响应还是显得迟钝和耗时，这是由于大多分层架构的具体实现都比较庞大，况且层里的组件间时常有紧耦合的情况。
	
**部署的简易性**

	等级：低
	理由：由于这种架构模式的分层实现方式，所以应用（尤其是大型应用）的部署成了一个问题。即，应用中的一个小修改就需要重新部署
	整个应用（或重新部署应用的大部分功能），从而导致对应用的部署工作就需要在非工作时间或非工作日来计划、安排和实施。正因为如
	此，这种架构模式不太适合持续集成，从而进一步降低了部署便利性的整体评级。
	
**可测性**

	等级：高
	理由：在这一架构模式中，各功能组件都属于相应的架构层，每层之间相对独立，每层都可以被模拟，从而便于测试。如，可以模拟展现层
	来测试业务层，也可以模拟业务层来测试展现层。
	
**性能**

	等级：低
	理由：虽然很多采用分层架构的应用都能正常运转，但是由于业务请求时常需要跨层访问才能获取到期望的结果，从而使得它不在高性能
	应用的行列。
	
**可伸缩性**

	等级：低
	理由：由于这一架构模式的紧耦合倾向性和庞大性，那么一旦采用了这一架构模式的应用就难于扩展。尽管你可以把逻辑架构层进行拆分
	成物理架构层或把整应用复制到多个结点，但是一旦在应用开发中这样做了，那要干的活就多了。
	
**开发的简易性**

	等级：高
	理由：开发的简易性之所以能获得了一个相对较高的评级，主要是由于这一架构模式被大家所熟知而且从开发层面上来看也不复杂。由于大
	多数公司在进行应用开发时，不同专业技能的人负责不同的模块（前端、服务器、数据库等），所以分层架构模式自然也成为了很多应用开
	发的选择。诸如这种公司内部的沟通方式和组织结构与软件开发模式的联系被概括为康威定律。

相关知识点：康威定律[^6]

[^6]:[康威定律](http://www.kankanews.com/a/2013-03-26/004892183.shtml)

<br/>

#第二章 事件驱动型架构模式
模型是一种流行的分布式异步架构，用于高度可扩展的程序。他同样具有高度适应的能力，可用于小型应用程序以及大型，复杂的系统。它是由高度解耦的，功能独立的事件处理组件组成，由此接受和处理事件。

事件驱动架构包含中介和代理两种主要的模式。中介者模式通常应用于当需要通过一个中央控制器来协调多个模块之间的运作，而代理模式通常用于需要链式的传递和处理事件。由于这两者的实现方式和特性的不同，理解每一种模式并如何正确地运用于当前项目环境显得尤为重要。

##中介者模式

在需要分层处理多个事件的时候中介者模式就显得非常有用。例如，一个简单的股票交易系统，首先需要验证这个交易,然后检查这个交易是否违反了各种规定，然后计算佣金，最终分配给一位代理人。所有这些步骤都需要一些步骤的编排来决定串行还是并行发生。中介者拓扑有四个主要的组成部分，事件队列，事件调度者，事件通道和事件处理器。事件流从客户端发送一个事件到事件队列，事件队列用于将事件传输给事件调度器，事件调度器接收到原始的事件队列，通过异步的方式分配每一个事件给事件通道来执行每一个步骤，事件处理器监听事件通道，接收从事件调度者分配来的任务，执行特定的业务逻辑来完成事件处理。图2-1说明了事件驱动架构中的中介者模式拓扑图。

![Figure 2-1](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/software_architecture_patterns_figure2_1.png "Figure 2-1")

在事件驱动架构中通常会有几十到上百个事件队列，该模式并没有指定实现事件队列组件的方式，它可以是一个消息队列,一个web服务端点,或任何组合。

这种模式下有两种类型的事件，初始事件和处理事件。初始事件是调度器接收到的原始事件，处理事件是调度器处理过并被事件处理器接收的事件。

待续

#第三章 微内核架构模式
微内核模式是一种为基于产品的应用程序而生的架构模式，有时也称它为插件模式。那么，什么是基于产品的应用程序呢？基于产品的应用程序是一种分版本打包和分版本下载的第三方产品。很多公司也开发和发布类似产品级的内部业务程序，有：版本控制、版本日志和可插拔的功能特性。这种情况就适合采用微内核架构模式。微内核架构模式可以让开发者在自己的核心程序基础上开发一些带有扩展性和隔离性的插件。

##模式介绍
微内核架构模式由两大组件构成：分别为**系统内核**和**插件模块集**。即，应用程序的逻辑被分为独立的插件模块集合和基本的核心系统，以提供扩展性、灵活性以及扩展特性和和逻辑的隔离性。`图Figure 3-1`以图说明了一个基本的微内核架构模式结构。

通常，微内核架构模式中的**系统内核部分**是指能使系统正常运转所需要的最少功能的集合。很多操作系统都实现了微内核架构模式，这也是微内核名字的由来。从业务程序的角度讲，**系统内核**通常是指按一定场景、规则或复杂的条件来进行逻辑处理的通用业务逻辑。

![Figure 3-1](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/software_architecture_patterns_figure3_1.png "Figure 3-1")

围绕着系统内核的插件模块都是独立于系统内核的，这些插件模块集包含了各种对系统核心功能进行扩展的定制功能、附加特性等等。通常来讲，插件之间也是独立的，不过你也可以设计多个有依赖关系的插件。换句话说，你在设计插件集时，需要保证插件间有很好的通信机制从而使得插件间的依赖最小。

同时，系统内核需要知道围绕它开发的插件哪些是与它可以正常通信的且如何通信。通常，一个常用的方式就是在系统内核依次注册这些插件，可以给每个插件注册诸如插件名称、数据交换协议、通信的数据结构在内的等等信息。比如，给一个税务系统开发一个可以标识出缴纳了高税额的纳税人的插件，那么注册信息就需要插件名称、上行下行的数据结构以及数据传输协议。这个例子中，假如你使用SOAP来做远程调用，那么还需要WSDL这个文件来对调用的服务进行描述。

相关知识点：SOAP[^7], WSDL[^8]

[^7]:[SOAP](https://docs.oracle.com/cd/E23943_01/doc.1111/e10807/c25_wsdl_and_soap.htm)
[^8]:[WSDL](https://docs.oracle.com/cd/E23943_01/doc.1111/e10807/c25_wsdl_and_soap.htm)

插件与系统内核之间可以有用多种方式进行通信，比如：OSGi[^9]、消息机制、远程服务(Web Services)或系统内核直接实例化插件。至于采用哪种方式，这完全取决于你的项目类型(是小型项目还是大型项目)以及项目的部署的方式(是单一服务器部署还是分布式部署)。在此应该明白插件间要必须要保持相对独立。

[^9]:[OSGi](https://www.osgi.org/)

系统内核与插件之间的通信协议规范可以是标准协议也可以是自定义协议。通常在第三方团队开发的插件中可以发现自定义协议，这些自定义协议对于系统内核开发商来说是完全不可控的。在这种不可控的案例中，通常需要在插件的自定义协议与系统内核的标准协议之间建立一个适配器来对两种协议进行相互转换，这样操作后，系统内核的协议就不需要与每个插件协议耦合，只需要建立一个适配器即可。同时，请记住在一开始定义通信协议时就要做好协议的版本控制。

##模式实例

微内核架构模式的最好实践可以说是Eclipse IDE。基础版Eclipse本身是一个很NB的编辑器。不过，当你为它加上各种插件后，它就变得更实用，更NB了。

Web浏览器是另一个很不错的微内核架构模式实践：内容显示和一些扩展功能都不是浏览器的系统内核。

以上两个例子是微内核架构模式基于产品型应用的实践，不过它还可以应用于大量商业程序。在这里咱们可以举个保险公司的索赔处理流程。索赔处理是一个非常复杂的流程，不同的情况都有允许的和禁止的业务逻辑规则。比如某些情况下挡风玻璃被石头子儿砸坏了可以申请免费更换，但是也有不能免费更换的情况。正是这些不同的情况构成了标准的索赔流程。

看到这里不要感到奇怪，因为大多数的理赔应用正是依靠大量的复杂的规则引擎集才能处理各种各样的理赔情况。不过，随着逻辑规则的修改或新增一个简单的逻辑规则都会影响其它现有的规则，这就需要更多的分析人员、开发人员和测试人员参与其中。庆幸地是，微内核模式就能解决刚才提到的大部分问题。

如图`Figure 3-2`中的那一摞文件夹，它描述了索赔流程的系统内核部分。它包含了保险公司处理理赔申请的基本的业务逻辑，当然了，没有包括各公司自己定制一些逻辑规则。从图中可以看到，每个插件的逻辑规则都对应到了系统内核的不同业务场景。这些插件可以采用完全自行开发或引用现成的规则引擎来实现。无论采用什么方案来实现这些插件，这些插件都是独立于系统内核的，即，添加、移除、维护插件时都不会对系统内核和其它插件产生一丁点儿的影响。

![Figure 3-2](https://github.com/Handy-Wang/Handy-Wang.github.io/blob/source/source/_posts/img/software_architecture_patterns_figure3_2.png?raw=true "Figure 3-2")

##模式考量

微内核架构模式有一个优点，就是它能被内嵌到或作为其它架构模式的一部分。比如，当系统中某个模块的功能不太稳定时，你会发现只能用微内核架构模式来解决这种特定的问题，而不能用这个模式来替换原架构模式。在形如这种情况时，你就可以在原有的架构模式(分层架构)里内嵌微服务架构模式。类似地，上一章讲到的事件驱动架构里的事件处理器组件也可以采用微服务架构模式来实现。

刚才提到的微服务架构模式为不断演进的设计和逐渐增加的开发需求提供了大力支持。首先好好地设计一个坚实的系统内核，然后在系统迭代过程中的功能增加和改进行都不需要对系统内核进行改造。

对于那些基于产品需求开发的应用程序，微内核架构模式应该作为架构的首选，特别是对于那些需要发布用户真正需要的新需求、新特性的产品。如果你发现这个架构不适合你的项目需求，你可以把架构方案修改成为更适合项目实际需求的架构方案。

##模式分析
下面的表格是对微内核架构模式各个特点的评级和分析描述。对于以下每个特点的评级是基于它是否能作为行业趋势或是否能作为本架构的代表性来评估的。至于本篇文章的几种架构的详细对比，请参见附录A。

**整体的敏捷性**

	等级：高
	理由：整体敏捷性是指对不断修改的需求的适应能力。对采用微内核架构方案的系统的修改，都是可以采用插件方式来隔离和解耦的。通常，
	大多采用微内核架构的应用程序的系统内核都是趋于稳定的，而且相当强劲、基本不会再修改。
	
**部署的简易性**

	等级：高
	理由：鉴于此架构模式的整体架构，那么插件部分是可以在动行时被动态地追加到系统内核上的（热部署），从而最大程度上减少了服务器
	down机的时长。

**可测性**

	等级：高
	理由：插件模块集可以很方便地被隔离测试，而且系统内核可以在不作任何改动的情况下就可以很容易地模拟出被扩展的插件特性功能。

**性能**

	等级：高
	理由：尽管微内核模式不能被自荐给高性能应用程序，但是，大多用到了微内核模式的应用都能正常地动转，因为你可以定制和简化需要扩展
	的插件功能。JBoss应用服务器就是一个很好的采用了微内核架构的例子：通过插件扩展能力，你可以按需移除那些高消耗的或不需要的插件
	功能，比如：远程访问功能、消息功能以及缓存功能等很消耗内存、高CPU的会拖慢应用程序的插件功能。

**可伸缩怀**

	等极：低
	理由：由于大多基于微内核架构的实现方案都是基于成形的产品应用而且规模较小，这些架构实现通常只是作为产品中的一个单元或模块，
	所以不利于扩展。尽管，在实际的插件模块开发中能做到插件级的伸缩，但，微内核架构并不是已具备整体高伸缩应用程序的首选。

**开发的简易性**

	等级：低
	理由：基于微内核架构模式开发的应用，需要深思熟虑的设计和对接口协议的合理管理，这无疑增加了开发的复杂性。接口协议的版本管理、
	插件的注册机制、插件功能的粒度以及插件与系统内核的通信机制等等内容导致了增加实现微内核架构的复杂度。

相关知识点：JBoss[^10]

[^10]:[JBoss](http://www.jboss.org/)

#第四章 微服务架构模式
微服务架构作为一种应用程序架构的可选方案以及它面向服务的特点，在行业中得到了快速的发展。由于它仍处于不断地演进中，所以关于它到底是什么以及如何具体实施的问题也一直存在。那么本章会就一些关键概念和基础知识进行讲解，以给大家提供一些是否选择此架构模式的参考依据。

##模式描述
无论你选择什么拓扑结构还是架构的实现方式，总有一些适用于这几种架构模式的通用核心概念。其中，第一个就是分块部署的概念。如图Figure 4-1所示，微服务架构中每个组件都是被单独部署的，这样更利于流水线部署、提高扩展性以及对应用和组件的高度解耦。

也许，要能很好地理解此架构模式的关键之一就是理解什么是服务组件。至于服务，与其去了解微服务架构里提供的各种服务，还不如去了解提供服务的各个组件，这些组件的粒度大小从一个小模块到一个应用的大的组成部分不等。这些服务组件包含了一个或多个负责某个小功能或大型业务逻辑的子模块。所以，在微服务器架构中合理地设计服务组件的粒度层次的确是一件非常有挑战的事情。关于这一挑战更详细的讨论，我们会在“服务组件编排”一节中介绍。

![Figure 4-1](https://github.com/Handy-Wang/Handy-Wang.github.io/blob/source/source/_posts/img/software_architecture_patterns_figure4_1.png?raw=true "Figure 4-1")

理解微服务架构的另一个关键是要明白它是一种分布式架构，即：此架构中的所有服务组件都是解耦而且可以通过一些远程访问协议(JMS[^11]、AMQP[^12]、REST[^13]、SOAP、RM[^14]I等等)来访问的。这也就不难理解为什么此架构具备很好的伸缩性和部署性。

[^11]:[JMS](http://docs.oracle.com/javaee/6/tutorial/doc/bncdq.html)
[^12]:[AMQP](https://www.amqp.org/)
[^13]:[REST](http://www.oracle.com/technetwork/articles/javase/index-137171.html)
[^14]:RM

微服务架构中令人兴奋的是它自身是从解决别的一些普通的架构模式中相关的问题演进而来，而不是为了守株待兔那些还未发生的问题而设计出来。微服务架构风格主要从两方面演进而来：采用分层架构开发的大型应用，以及采用面向服务的架构开发的分布式应用。

此架构从大型应用到微服务架构风格的演进过程，主要经历了持续交付的发展历程，从研发到产品的持续部署的概念使得应用程序的部署更简单了。那些典型的由存在紧耦合关系的组件集构成的大型应用，非常难于维护、测试以及部署（所以，在很多大型IT企业里出现了非常常见的“按月迭代开发”现象）。这几个因素综合起来往往导致了应用的不稳定、不健壮，每次新部署的内容都会被下一次推翻。微服务架构模式通过把应用拆分为几个可单独部署的单元来解决刚才说的那几个问题，从而使得这些单元可以独立于别的服务组件进行单独开发、测试以及部署。

此架构的另一条演变路径是为了解决面向服务的架构的应用(SOA)中的问题。尽管SOA模式很强大，强大到提出了无与伦比的抽象层级、异构系统间的连接性、服务的组织结构以及要与商业目标看齐的承诺，但由于它的复杂性、高成本、普遍性而难于被理解和实现，而且也常常导致了很多现有应用的失控。微服务架构的风格通过简化服务的概念、消除服务编排的需要以及简化服务组件间的连接性和访问来解决刚才聊到的SOA的复杂性。

##微服务架构的多种拓扑结构
尽管我们可以列出很多实现微服务架构的方式，但是这里我们只聊聊三种主流的微服务架构拓扑结构：**基于REST风格的API的拓扑结构**、**基于REST风格的应用的拓扑结构**以及**集中式消息型的拓扑结构**。基于REST风格的API拓扑结构适用于通过一些API接口来提供一些独立的、小型服务的网站。如图Figure 4-2，这种拓扑结构由非常小细粒度的服务组件构成(所以称之为微服务)，每个组件内都包含了一个或两个既完成特定业务又独立于其它服务的模块。这些细粒度的服务组件可以通过基于REST风格的而且是基于Web部署的接口来访问。雅虎、谷歌和亚马逊公司的一些常见的且用途单一的云服务就是实际例子。

![Figure 4-2](https://github.com/Handy-Wang/Handy-Wang.github.io/blob/source/source/_posts/img/software_architecture_patterns_figure4_2.png?raw=true "Figure 4-2")

基于REST风格的应用的拓扑结构与基于REST风格的API的拓扑结构的差异在于：基于REST风格的应用前端接收的是传统的网页请求或传统PC客户端的请求，而不是接收简单的API请求。如图Figure 4-3所示，应用的用户界面被部署为一个单独的Web应用，这个Web应用通过REST风格的接口来远程访问已部署好的服务组件(即业务逻辑)。基于应用的拓扑结构中的服务组件有别于基于API的拓扑结构中服务组件，它里面的服务组件规模更大、粒度更糙而且这些组件只是整个应用中的冰山一角。所以，这种拓扑结构在中小型的复杂度较低的商业程序中应用较为普遍。

![Figure 4-3](https://github.com/Handy-Wang/Handy-Wang.github.io/blob/source/source/_posts/img/software_architecture_patterns_figure4_3.png?raw=true "Figure 4-3")

微服务架构模式的另一种常见拓扑架构是集中式消息型的拓扑结构。如图Figure 4-4 所示，这种拓扑架构除了访问方式不是采用远程调用的方式而是采用REST方式外，和前面提到的基于应用的拓扑结构相似，它用到了轻量级的集中式消息代理（如：ActiveMQ[^15]、HornetQ[^16]等等）。在理解这一拓扑结构时不要与SOA架构模式混淆，也不要把它当成是SOA的轻量版。这种拓扑结构中的轻量级消息代理不会做任务服务组件的编排转换工作或复杂的路由工作，它只是负责转发对远程服务组件的访问。

[^15]: [ActiveMQ](http://activemq.apache.org/)
[^16]: [HornetQ](http://hornetq.jboss.org/)

这种集中式消息型拓扑结构尤其是在大型商业项目或需要在用户界面层与服务组件层之间做复杂传输控制的项目中会用到。这一基于简单REST风格的拓扑结构的特点是强大的排队机制，异步消息机制、监听机制、错误处理机制以及更好的负载平衡和伸缩性。单点故障问题和架构瓶颈问题往往与一个集中式代理相关，这些问题可以通过代理集群和代理互联来查找（可以把一个代理实例切分为多个代理实例，从而可以分散基于系统功能区的消息吞吐量的加载）。

![Figure 4-4](https://github.com/Handy-Wang/Handy-Wang.github.io/blob/source/source/_posts/img/software_architecture_patterns_figure4_4.png?raw=true "Figure 4-4")

##避免依赖和编排

微服务器架构模式中最具挑战的问题之一就是给众多服务组件定一个合理粒度的层级结构。如果这个粒度太糙，那么我们将不能从微服务架构模式中获益(可部署性、伸缩性、可测性、松耦合)。尽管粒度太糙不好，但粒度太细就需要对众多组件进行编排，那么一个很精简的微服务架构就会变为一个重量级的面向服务的架构，这将会遇到与基于SOA架构的应用程序相同的问题：复杂、混乱、开销大以及一些其它问题。

如果你发现你需要在用户界面层或API层来编排、组织管理你的众多服务组件，那么说明这些组件的粒度太细了。同样的，如果在处理一个请求时需要在服务组件间通信，那么说明要么是这些服务组件是粒度太小，要么是这些服务组件的切割没有站在业务功能的角度来实施。

服务间通信会迫使组件间产生耦合，而不是采用共享数据的方式来处理。比如，一个处理网络定单的服务组件需要客户信息时，可以从数据库中获取一些必要的数据而不是通过调用其它服务组件来获取数据。共享数据库可以解决数据层面的问题，但是功能共享的问题如何来解决呢？比如，当一个服务组件需要另一个被包含在其它服务组件里的功能时，有时你是可以拷贝这部分你需要的功能代码的(但是打破了DRY原则：不要出现重复的代码)。在大多实现微服务架构的商业应用程序中这是一种非常常用的共享功能代码的方式，牺牲一小部分重复冗余的逻辑来换取组件间的独立性和可部署性。那些小型的工具类功能应该就是属于这一类。

如果你发现无论服务组件的粒度粗细，你仍然需要对这些服务组件进行编排，那么说明你的项目不适合微服务架构模式。由于微服务架构的分布式特性，所以对于那种跨服务的事务操作是很难维护的。这种操作需要一些事务补救措施来回滚事务，但是这样就增加了微服务架构的复杂性。

##模式考量
微服务架构模式解决了大型应用程序和面向服务架构的应用程序中的诸多常见问题。由于主要的应用程序组件被拆分为小的且可单独部署的单元，这样基于微服务架构的应用程序就更健壮、扩展性更强、能更好的支持持续交付。

此架构模式的另一优点在于它提供了实时的生产环境部署，因此很大程度上减少了每月或每周末需要的生产环境来一次大部署的次数。这是因为代码的修改已经与特定的服务组件被隔离开，即只需部署修改的服务组件。如果你只有一个服务组件实例，你可以写一些用户界面来检测热部署以及重定向用户的请求到错误页面或等待页面。或者，你可以在实时部署期间置换服务组件的多个实例，以使得在部署期间服务仍然可用（然而，分层架构很难做到这一点）。

最后一个需要考量的点是：由于微服务架构模式是一个分布式架构，所以它也会遇到一些与事件驱动架构模式相同的问题，包括对接协议的建立、维护成本、管理成本、远程系统可用性以及远程访问的鉴权和授权。

##模式分析

下面的表格从常用架构特点角度出发对微服务架构进行了评级和分析。对于每一个架构特点的评级都是基于此架构在这一特点上的应用以及是否被大家所熟知。至于几种架构模式的并行对比，请参看附录A。


**整体的敏捷性**

	等级：高
	理由：整体敏捷性是指对不断修改的需求的适应能力。对采用微内核架构方案的系统的修改，都是可以采用插件方式来隔离和解耦的。由于
	存在单独部署单元的概念，所以任何修改都是被隔离到单独的服务组件里，这样就可以更快更简单地进行部署。同理，采用此架构搭建的
	应用程序的内部组件间都是松耦合的，从而也能促进变化的拥抱。
	
**部署的简易性**

	等级：高
	理由：由于事件处理器组件的解耦特性，所以整体上此架构模式是相对容易部署的。代理拓扑结构之所以比中间者拓扑结构更易于部署，
	主要是因为事件中间者组件与事件处理器之间是紧耦合的，即：对事件处理器的修改会级联到事件中间者的修改，最终两者都需要重新
	部署。

**可测性**

	等级：高
	理由：由于业务功能被分离和隔离到独立的应和程序中，所以测试范围是可控的，而且还可以进行一些针对性测试。回归测试特定服务组件
	比回归测试整个大型应用程序会更容易、更可行。 由于组件是解耦的，所以，从开发的角度来说对某一点的修改不会对应用程序的其它
	部分超成影响，减轻了由于一个小小的修改而需要测试整个应用的压力。

**性能**

	等级：低
	理由：尽管你可以使用此架构模式来搭建一个应用程序并且运行正常，但由于此架构的分布式特性，所以搭建出来的应用程序并不属于
	高性能的应用程序。

**可伸缩怀**

	等极：高
	理由：由于应用程序被切分为独立的可部署单元，所以每个服务组件都可以单独进行自调整式扩展。比如，股票交易的管理操作区是
	不需要进行扩展的，这是因为使用此功能的用户量很少；然而交易区服务组件需要被扩展，这是因为大多股票交易程序都要很高的吞吐量。

**开发的简易性**

	等级：高
	理由：由于所有功能都被隔离为单独的不同的服务组件，所以开发工作很容易在一个小的已被隔离的区域中进行。开发人员都没有机会
	由于修改一个服务组件而影响其它服务组件，所以减少了开发人员间或开发团队间协作的工作量。
	
#第五章 基于空间的架构模式

大多数web应用的请求流转是按照下面的流程来进行的：从浏览器发起一个请求，此请求先到达静态资源服务器(Web Server, 如Apache、nginx)，然后是应用服务器(Application Server, 如Tomcat等)，最终到达数据库服务器。这个流程对于用户量少的应用来说是没有问题的，但是一旦用户负载增加就会出现瓶颈。首先出现瓶颈的是静态资源服务器，然后是应用服务器，最后是数据库服务器。通常，对于用户负载上升导致的瓶颈都是通过对资源服务器扩展来解决，这个方案相对简单且成本低，确实有时候可以解决问题。但是，大多数时候简单对的静态资源服务器进行扩展只会把瓶颈问题转移到应用服务器。然而，对应用服务器扩展就相对较难且成本较高，同时也把瓶颈问题又转移到了数据库服务器，这就使得扩展更难，成本更高了。尽管可以对数据库服务器进行扩展，但最终你会看到一个三角形的拓扑结构，即静态资源服务器所处夹角最大，因为它容易扩展，而数据库服务器所处的夹角最小，因为它最难扩展。

在任何一个大型的、有着用户高并发负载的应用中，最终决定并发信息处理能力的因素往往是数据库。虽然各种缓存技术和数据库产品可以解决上述问题，但是靠扩展的来解决极限负载仍是一个难题。

所以，基于空间的架构模式主要是用于定位和解决扩展性和并发性问题，此架构模式同样适用于用户并发量不稳定的系统。那么，从架构层面来解决扩展性问题比通过扩展数据库或改进缓存技术来解决要更好些。

##模式描述
空间架构模式（也被称为云架构模式）最小化了影响应用扩展的诸多因素。这种架构模式得名于元组空间的概念，分式式共享内存空间的概念。它通过去除中央数据库约束转而使用复制型内存数据网格来达到高扩展性。应用程序的数据被保存在内存中并被复制到所有活跃的处理单元中。当用户负载量上升或下降时，这些处理单元可以被动态的开启和关闭，从而解决了变化的扩展性问题。由于没有了中央数据库，那么数据库瓶颈就没有了，从而给应用程序提供了近乎无限扩展的能力。

大多适合这种模式的应用都是标准的网站，接收浏览器的请求并执行一些操作。投标拍卖网站就可以作为一个案例来讲讲，这个网站就不断地接收互联网用户的投标。这个应用会接收某些条目的投标，并记录下时间信息，然后更新此条目的最新投标信息，最后把相关信息反馈给用户。

此架构主要包含两大组件：处理单元和虚拟中间件。图5-1中描述了一个基本的空间架构模式以及它主要的组件。

![Figure 5-1](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/software_architecture_patterns_figure5_1.png "Figure 5-1")

从上图可以看到，处理单元组件包含了应用组件集（或应用组件集的一部分），它包含了基于web的组件以及后端的业务逻辑。处理单元的内容随着应用程序的类型而改变－如，基于Web的小型应用程序很有可能被部署到某一个处理单元里，而大型应用程序则很有可能根据功能模块的不同划分而被拆解到多个的处理单元里。所以，处理单元通常包含应用程序功能模块、内存级数据网格以及一个为故障恢复而准备的可选的异步持久化存储。另外，它还包含了一个复制引擎(replication engine)，是虚拟中间件用于复制处理单元间差异化的变更数据，即用于同步处理单元间的数据。

虚拟中间件组件会处理一些清理和通信工作，它包含了很多控制数据同步和处理请求的组件。虚拟中间件包含了消息网格、数据网格、处理网格以及部署管理器。在接下来的章节中会详细介绍它们，而且它们可以被作为第三方产品进行定制化的开发或购买。

##模式结构
空间架构模式的神奇之处就在于虚拟中间件组件集以及被包含在处理单元中的内存级数据网格。如图5-2所示，它展示了一个典型的处理单元架构，包含了程序模块、内存数据网格、用于为故障恢复而准备的可选的异步持久化存储以及数据复制引擎。

![Figure 5-2](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/software_architecture_patterns_figure5__2.png "Figure 5-2")

本质上说，虚拟中间件是此架构的控制器，负责管理请求、会话、数据复制、分布式请求处理以及部署处理单元。在虚拟中间件内部有四个主要的组件：消息网格、数据网格、处理网格以及部署管理器。

###消息网格
如图5-3所示，消息网格的消息输入是用户请求和会话信息。当一个用户的请求进入虚拟中间件内后，消息网格组件来决定哪一个处于活跃状态的处理单元可以处理这个请求并把这个请求转发给相应的处理单元进行处理。消息网格的复杂度范围从一个简单的“循环算法”到一个更复杂的“下一个可用算法”，这些算法用于计算应该哪个处理单元来处理哪个请求。

###数据网格

The data-grid component is perhaps the most important and crucial component in this pattern. The data grid interacts with the data- replication engine in each processing unit to manage the data repli‐ cation between processing units when data updates occur. Since the messaging grid can forward a request to any of the processing units available, it is essential that each processing unit contains exactly the same data in its in-memory data grid. Although Figure 5-4 shows a synchronous data replication between processing units, in reality this is done in parallel asynchronously and very quickly, sometimes completing the data synchronization in a matter of microseconds (one millionth of a second).

###处理网格

The processing grid, illustrated in Figure 5-5, is an optional compo‐ nent within the virtualized middleware that manages distributed request processing when there are multiple processing units, each handling a portion of the application. If a request comes in that requires coordination between processing unit types (e.g., an order processing unit and a customer processing unit), it is the processing grid that mediates and orchestrates the request between those two processing units.

###部署管理器

The deployment-manager component manages the dynamic startup and shutdown of processing units based on load conditions. This component continually monitors response times and user loads, and starts up new processing units when load increases, and shuts down processing units when the load decreases. It is a critical component to achieving variable scalability needs within an application.

##模式考量

The space-based architecture pattern is a complex and expensive pattern to implement. It is a good architecture choice for smaller web-based applications with variable load (e.g., social media sites, bidding and auction sites). However, it is not well suited for tradi‐ tional large-scale relational database applications with large amounts of operational data.
Although the space-based architecture pattern does not require a centralized datastore, one is commonly included to perform the ini‐ tial in-memory data grid load and asynchronously persist data updates made by the processing units. It is also a common practice to create separate partitions that isolate volatile and widely used transactional data from non-active data, in order to reduce the memory footprint of the in-memory data grid within each process‐ ing unit.
It is important to note that while the alternative name of this pattern is the cloud-based architecture, the processing units (as well as the virtualized middleware) do not have to reside on cloud-based hos‐ ted services or PaaS (platform as a service). It can just as easily reside on local servers, which is one of the reasons I prefer the name “space-based architecture.”
From a product implementation perspective, you can implement many of the architecture components in this pattern through third- party products such as GemFire, JavaSpaces, GigaSpaces, IBM Object Grid, nCache, and Oracle Coherence. Because the imple‐ mentation of this pattern varies greatly in terms of cost and capabili‐ ties (particularly data replication times), as an architect, you should first establish what your specific goals and needs are before making any product selections.

##模式分析
The following table contains a rating and analysis of the common architecture characteristics for the space-based architecture pattern. The rating for each characteristic is based on the natural tendency for that characteristic as a capability based on a typical implementa‐ tion of the pattern, as well as what the pattern is generally known for. For a side-by-side comparison of how this pattern relates to other patterns in this report, please refer to Appendix A at the end of this report.

#附录A-各大架构模式的分析总结
Figure A-1 summarizes the pattern-analysis scoring for each of the architecture patterns described in this report. This summary will help you determine which pattern might be best for your situation. For example, if your primary architectural concern is scalability, you can look across this chart and see that the event-driven pattern, microservices pattern, and space-based pattern are probably good architecture pattern choices. Similarly, if you choose the layered architecture pattern for your application, you can refer to the chart to see that deployment, performance, and scalability might be risk areas in your architecture.

While this chart will help guide you in choosing the right pattern, there is much more to consider when choosing an architecture pat‐ tern. You must analyze all aspects of your environment, including infrastructure support, developer skill set, project budget, project deadlines, and application size (to name a few). Choosing the right architecture pattern is critical, because once an architecture is in place, it is very hard (and expensive) to change.

感谢 @磊哥 贡献了第二章的翻译。