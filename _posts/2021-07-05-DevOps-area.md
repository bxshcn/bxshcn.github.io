---
layout: post
title:  "DevOps区域-DevOps实践梳理（译）"
comments: true
categories: 方法论
tags: DevOps
---

译者按：

区域area是一个抽象名词，它是以Dev和Ops，以及两者之间的协作交互为特性，所定义的工具tool和流程process的总称。
![areas](/assets/2021-07-05-DevOps-area/areas.png)

在此基础上，作者还参照定义了DevOps的成熟度模型，针对任意一个成熟度级别，从具体practice、抽象pattern和深层principle三个层面进行描述，形成了一个针对DevOps的完备立体的论述体系。 

本文译自[Devops Areas - Codifying devops practices](http://www.jedi.be/blog/2012/05/12/codifying-devops-area-practices/)，作者Patrick Debois，大概成文于2011年上半年。作为DevOps早期的关键布道者，尽管Parick Debois在2011年就写下了这篇文章，对什么是DevOps进行了非常精彩的论述，但网上很少有其他的DevOps相关文章参引这篇文献，其后的留言也异常惨淡，所以我觉得非常有必要将其译成中文，分享给有需要的人。



# 正文
在和Gene Kim、John Willis、Mike Orzen一起编写《Devops Cookbook》的时候，我们收集了很多“devops”实践案例。那一阵子我们费了很大劲才将它们编排进书中，我想这是因为当时我们还没有掌握与这些实践有关的某种观念模式。

本文将首次尝试为梳理devops各类实践提供一种结构化方法，尽管一些术语和描述还不够成熟，但我觉得，为获取大家反馈而将其尽快分享出来非常重要。

## 从正确的角度看devops
大家都知道现在有很多种devops定义。有些人还会改变devops的名称以便将概念扩展至IT的其他方面，比如star-ops，dev-qa-ops，sec-ops等等。我想这是那些首先参与到devops运动中的人，希望将类似的想法应用到dev和ops之外的其他过程中。（也许类似bus-qa-sec-net-ops的名称更让人印象深刻:）。

我先参引下面的定义：
- devops：在整个组织层面增进协作和优化，甚至超越IT（比如人力、财务）和公司边界（比如供应链）
- devops ‘lite’（缩减版的devops）：当人们仅仅关注于dev和ops之间协作的时候

正如Damon Edwards[所述](http://dev2ops.org/blog/2010/11/7/devops-is-not-a-technology-problem-devops-is-a-business-prob.html)，devops并不只是与技术有关，更与某种业务问题相关。[TOC约束理论](http://nl.wikipedia.org/wiki/Theory_of_constraints)告诉我们应该优化整体而不是部分。对我来说，整体就是如何让业务适应客户的问题，或者从精益角度说，整体是整个价值链。瓶颈和相关改进可能发生在公司的其他任何地方，并对公司的dev和ops部门产生影响。

所以即便你们的问题存在于dev或ops部门之内，或者这两个部门之间，也许仍应该从公司的其他部门着手去优化，因此提前给出应对‘devops’问题（如果有这样一类问题的话）的预定步骤是不可能的：你们在公司中面临的问题可能有天壤之别，相应的解决方案又有不同的前提，并产生不同的效果。

即便没有预定步骤，我们仍能收集整理出人们在克服相似境况下问题的实践应对之法。我总是鼓励人们分享他们的实践案例以便其他人能从中学习（这也是[devopsdays](https://devopsdays.org)存在的关键原因），这能帮助我们更好的理解这些实践，我将在稍后阐述什么是最棒的实践。

当前很多实践案例都专注于部署、dev与ops的协作、度量等方面（即Devops Lite)。这是dev和ops的名称，以及讨论相关内容人群的知识背景所决定的。我希望将来的讨论也能扩展到公司的其他部门：例如，[协同人力资源与Devops](http://www.spikelab.org/blog/devops-job-title.html)，或者将我们的度量和财务报告联系起来。

另外需要清楚了解的一点是，任何系统（公司也是一种系统）都始终处于不断的变化中：改变系统中的任何部分都会产生某种影响。所以你不能想当然的认为，刚刚解决的问题和瓶颈不会再出现，所以你需要持续关注他们。显然，系统越稳定保持这种关注就越容易。但话说回来，devops和安全类似，它是朝向某个遥远目标的长途旅程，而非某种终结状态。

![complete-devops](/assets/2021-07-05-DevOps-area/complete-devops.png)

## 超越dev和ops
让我们仔细来看某些经常被讨论的实践案例：‘dev’和‘ops’两者之间的部分。

多数情况下，‘dev’意味着实施‘项目’，而‘ops’意味着运维‘产品’。在实施项目中，我们会使用一些方法论（比如Scrum，Kanban等），而在运维中，我们会使用另外一些方法论（比如ITIL，可视化Ops等），这两块在这些年中都在一直发展：从dev的角度看，出现了‘持续交付（Continous Delivery）’，从运维的角度看，出现了应用生命周期（Application Life Cycle，ALM）。但这些努力的重点都放在优化公司的单个部门上，而在多部门的协作上投入甚少，所以这些方法论在应对其所涉范围之外的瓶颈问题时都面临很大的困难。我想这就是devops开始的地方：它寻求并推动不同部门之间的协作，以便我们能看到一个完整的系统，并作出所有必要的优化（而不只是在单个的部门内进行优化）。

![devops-lite](/assets/2021-07-05-DevOps-area/devops-lite.png)

## DevOps区域（area）
我的devops观念模型有四个关键区域：

- 区域1：把交付活动延伸到业务运营阶段（Extend delivery to production）（请参考Jez Humble的观点）：这是dev和ops协作的地方，以改进将项目产品交付到生产业务运营的整个过程。
- 区域2：把业务运营活动前推到项目开发阶段（请参考John Allspaw的观点）：所有业务运营信息都被反馈到产品项目的开发过程中。
- 区域3：将项目开发内嵌到业务运营中：生产环境中发生的任何事情，项目组都要一起承担责任。
- 区域4：将业务运营内嵌到项目开发中：软件项目一开始就充分考虑业务运营需求。

在这四个区域中，都存在dev和ops的双向交互，并产生知识的交换和反馈。

根据你所感受到的最紧迫的"当前"瓶颈的表现，你可能想解决多个区域中的不同问题。问题没有先后之分，你不必一定要先解决区域1中的问题，然后再去解决区域2中的问题。你只需将这些问题都理解为压力点，然后保持它们的平衡。

区域1和区域2更依赖工具，但并非只关注工具。区域3和区域4与人或文化的转变的关系更密切，因为它们涉及到这个“链条” 的更深层。

用一张表来呈现上述四个区域：

![devops-areas-schematic](/assets/2021-07-05-DevOps-area/devops-areas-schematic.png)

正如你所看到的：
- dev和ops这两部分都具有和它们各自工作特性相关的内部流程
- 这两个流程因为区域而融合一致：区域拓展dev到业务运营，反过来又将ops拓展到项目开发。
- 这很像一个双环：area 1和area 2形成了内环，而area 3和area 4形成了外环。

> 注：
> 1. 每个区域真的需要更好记的名字
> 2. Ben Rockwoods在“[devops的三个方面](http://cuddletech.com/blog/?p=624)”这篇文章中已经列出了3个‘方面’，但我想‘区域’的概念使得这些内容更为具体。

![devops-areas](/assets/2021-07-05-DevOps-area/devops-areas.png)

## 区域的层次
对于任一区域内部结构，我们都能按传统上的分层法（工具tools、过程process、人people）来理解。

每当我听到别人在讲述某个Devops实践场景故事（story）的时候，我总是试着将其和这四个区域中的某一个联系起来，并弄清它所解决的问题属于相关区域中的哪个层面。这些实践可能会对不同的层面都产生影响，因此我将这些层面作为某种标签（tags），以便迅速弄清整个实践场景故事。这种分层视角的另外一个好处是，每当你关注一个区域时，你能问自己可以通过什么实践活动来改进其中的每个层面：为了产生最大的效力，这个方法必须在所有三个层面都进行，也就是说， 有什么实践能改进people，什么实践能改进tools，什么实践能改进process。

终极的devops工具应该支持所有这四个区域中的人和过程方面，而不仅仅只是区域1（部署）或区域2（监控/度量）。一个包含不同工具（这些工具用来应对不同区域交互）的devops工具链会更有意义。某个工具本身并不必然使其成为一个devops工具：类似chef和puppet这样的配置管理系统非常棒，但如果只运用在Ops中的话，对解决我们的问题就不太有用。当然我们承认这样做会提升基础实施的敏捷性，但除非将其应用到交付中（比如创建测试和开发环境），否则就不是‘devops’。这表明，是人在应用工具背后的思维方式使其成为某种devops工具，而不是该工具本身。

![devops-layers](/assets/2021-07-05-DevOps-area/devops-layers.png)

## 区域的成熟度
既然我们已经弄清楚了区域及其层次，我们还想跟踪我们开始解决问题和提升改进的进度。

Adrian Cockroft建议为devops使用[CMMI级别](http://groups.google.com/group/devops/browse_frm/thread/f3de603a4cea493e?scoring=d)：

CMMI级别能帮助你量化改进过程的成熟度。但这只解决了一个层面(译注：作者的意思是说CMMI级别旨在解决process这个层面的问题)的问题（当然这也很重要）。简单来说，CMMI描述了如下五个级别：
1. 初始级（Initial）：不可预测，糟糕的过程控制和响应
2. 可管理的（managed）：专注于项目开发，响应缓慢
3. 定义的（Defined）：专注于组织，主动应变
4. 量化的管理（Quantively Managed）：度量和控制方法
5. 优化（Optimizing）：专注于提升

所有这些分级都能被应用到dev，ops，或者devops。当你正在改进优化一个区域时，它会告诉你当前过程处于哪个级别。

[持续集成成熟度模型](http://blogs.urbancode.com/continuous-integration/continuous-integration-maturity-model/)是另外一种成熟度级别的表现方法。它基于IT界的行业共识为不同的成熟级别设定了一组实践集：
1. 入门（Intro）：使用源码控制……
2. 新手（Novice）：提交触发构建……
3. 中级（Intermediate）：自动部署到测试环境……
4. 高级（Advanced）：自动的功能测试……
5. 疯狂（Insane）：持续部署到生产环境……

不同于CMMI只专注于过程（process），持续集成成熟度的实践能含括工具集（tools），过程（process）或人（people）这三种层面。总之，大家所公认的越高级的实践活动将被归置于更高的级别。

## 实践、模式和原则practices, Patterns and principles
一个具体实践可以是从传闻故事到系统性方案中的任何东西。类似的实践能被归类为模式，以便将其提升到另外一种更高的层次。和软件设计模式类似，我们能将devops实践归类整理为devops模式。

实践和模式都依赖于原则，这些底层的原则将指导你在什么时候如何应用这些实践和模式。这些原则可以从其它领域（类似精益、系统理论，人类心理学）借用，比如敏捷宣言所指出的一些原则。

这样我们就逐步从实践转向模式，最后转向原则。

> 注:
>  
> 我一直想知道，将来devops是否会诞生出新的原则，或者将现有的原则应用到新的方面。

## 一些实践案例
下面是一些按照标准模板编写的实践案例。这些实践/模式/原则描述的也许不够清晰，但它们的确能被用来澄清和整理我们的DevOps实践。

![dev-test-prod](/assets/2021-07-05-DevOps-area/practice-example-dev-test-prod.png)

![expose-information](/assets/2021-07-05-DevOps-area/practice-example-expose-information.png)

![dev-pagers](/assets/2021-07-05-DevOps-area/practice-example-dev-pagers.png)

![integrated-backlog](/assets/2021-07-05-DevOps-area/practice-example-integrated-backlog.png)

## 区域指标 Area Indicators
基本思路是将可跟踪的度量和指标（metrics/indicators）列出来。测量的数字本身可能并不重要，但数字的变化率很重要。这类似于跟踪故事点（译注：Storypoint，敏捷开发比如scrum的术语，实现一个user story所耗费工作量的理想化度量，比如某个user story是2个storypoints，另外一个user story是8个storypoint。）的完成速度，或者跟踪故障恢复的平均时间（MTTR）这些度量值。

> 注: 我担心大家误解，以为我所说的就是一些待跟踪的度量值metrics，因此我特意称其为指标indicators 。
> 
> 译注：测量Measurement is a data point at a single point in time，比如当前体温38摄氏度，它无关乎上下文信息，比如当我们说你的blog这个月有10000人次访问，我们并不知道相较于上个月，这个访问量是否有新增；度量metric is a data point in context，metric也是一个数据，但总是一个比较变化的数据，比如The change in website traffic compared to last month，又比如最近体温相对于上一个小时升高了0.5度；指标indicator往往consisting of a set of different metrics或者measurements并进行复杂的计算，以便用于strategic decision-making。作者的意思是说，针对DevOps不同Area，设计好的Indicators，对我们了解相关area的实施状况非常重要。

比如：
- 工具层：部署/天
- 过程层：变更请求次数/天
- 人员层：涉及人数/部署（即每次部署涉及到的参与人数）
- 
这里的描述还不够详实，我正在为Velocity 2011大会上关于DevOps度量的[演讲](http://www.jedi.be/blog/2011/09/06/velocityconf-devops-metrics/)而准备，届时大家可以参考。

## Devops记分卡
为展示你在实施'DevOps'过程中的进度，你可将上述所有东西放在一个矩阵中，以便对你在不同层面和区域实施优化的方面有一个概览。

显然只有在不欺骗你自己、你的老板和你的顾客的情况下，这样做才有意义。

![devops-scorecard](/assets/2021-07-05-DevOps-area/devops-scorecard.png)

## 项目团队，产品团队和NOOPS
Jez Humble经常谈到项目团队演化为产品团队的过程：一个个隔离的竖井，不是根据人员技能来分拆，而是因为交付的产品功能不同而形成。如果当然这样划分团队，就有创建新的竖井式部门的潜在风险。显然这些产品团队需要再次协作：你应该将其他产品团队视为外部依赖，就像其他竖井式部门一样(比如人力资源，财务，销售部门等) 。他们彼此之间交互的区域将非常类似。
> 译注：
> 
> 产品团队1与财务团队的交互，产品团队1与产品团队2之间的交互，这两类交互是类似的。作者的意思是说，公司中不同竖井式部门之间的交互具有共性，类似于本文所描述的dev和ops团队之间四个交互区域。

当你和公司外部的产品团队一起工作时，你还会看到NOOPS这样的术语，比如你的产品的某些功能依赖于SaaS的情况。不仅在每个区域的工具层面进行整合很重要，在人员和过程层面进行整合也非常重要，但后者常常被忽视遗忘。自动化让你走的更快，但当问题发生或者情况发生变化时，必须依赖前述整合的协作来实现同步。
> 译注：
> 
> 作者的意思是说，即便你使用了SaaS，也并不意味着没有运维（NOOPS）。你仍然需要和SaaS团队协作，而这就涉及到类似于devops中所谈到的四个区域沟通协作模型，你不仅要整合四个区域中的工具，以形成完整的工具链（这意味着自动化和效率的提高），你还要分别整合四个区域中的人和过程，使得沟通顺畅，流程无碍。

## CAMS和区域
CAMS这个缩略词（文化、自动化、测量、共享）能粗略映射到上述区域结构：

- 自动化（Automation）：类似于区域1：交付过程
- 测量（Measurement）：类似于区域2：反馈过程
- 文化（Culture）：类似于区域3：将项目开发内嵌到业务运营中embedded devs in Production
- 共享（Sharing）：类似于区域4：将业务运营内嵌到项目开发中embedded ops in Projects

当然自动化、测量、文化和共享可以发生在任意区域，但某些区域似乎更关注于这四者中的不同部分。

![devops-areas-cams](/assets/2021-07-05-DevOps-area/devops-areas-cams.png)

> 译注：
> 
> 整个软件的全生命周期，最左侧是Project，最右侧是Production。Culture和Sharing其实都是一种思维，也可以说都是一种文化，即相互尊重和协作的文化：在Project伊始除了业务功能外，还考虑运维和运营（Sharing）；在运营阶段也同步考虑将数据反馈给开发（Culture）
> 
> 不过个人觉得将CAMS映射为上述区域有一些牵强，因为CAMS是DevOps的一个高阶视角，完全不同于具体的低阶的区域划分思路，他们彼此之间几乎完全重叠，但是关系异常复杂。比如CAMS中的Culture，可以理解为流程（比如使用Scrum），组织单元之间的协作机制（制度、流程），工作思路和策略等（比如限定技术债规模，定期回顾和总结），涉及到多个area的不同层面（人、工具、流程）。简单的映射是不可能的，也没有必要。
> 
> 也就是说，我们既可以从area+layer的角度来理解DevOps，也可以从CAMS角度来理解DevOps。

## 结论
Devops区域、层次和成熟度模型，向我们提供了一个理解某个具体实践故事（Practices stories）的框架，并能被用来识别devops领域中不同区域的改进。我很盼望大家的反馈。如果有人愿意提供帮助，我将开放一个大家能按照这一结构输入实践故事的站点，以帮助大家更好的学习。尽管我的网站现在已经没有太多的硬件资源，但我非常乐意促成此事。
