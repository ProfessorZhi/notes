---
source: nowcoder
url: https://www.nowcoder.com/discuss/864821937527128064?sourceSSR=users
fetched_at: 2026-04-08T16:16:03
---

# 26年全网最全Agent学习路线，拿走不谢！

作者：程序员花海

时间：03-21 10:53 发布于上海

大家好，我是@程序员花海，眼下 26 届春招、27 届暑期实习全面开启，后端卷到没边，AI Agent的岗位占主导，很多牛友在我的评论区留言，想让我出一份Agent学习路线。

我特意去看了下，打开淘天的招聘页面，以校招为例，一眼望去全是AI相关的岗位，只能说之后绝大多数岗位都会快速推进AI的落地和实践。

之前写过 Java 后端 3 个月抢救路线https://www.nowcoder.com/discuss/824693499982315520?sourceSSR=users，也收到了牛友们的强烈好评，这次专门给后端转 Agent做一套最少必要知识路线—— 不堆概念、不啃论文，只学面试必问、项目必用、能快速写进简历的硬核内容，3 个月从 0 到能投 Agent 岗、做企业级项目。

一、转型前置：快速复用后端底子

这部分不用重学，只做对齐，把你已有后端知识映射到 Agent 体系。

必掌握（后端本来就会）

API 设计、HTTP/HTTPS、接口鉴权

异步、线程池、重试 / 降级 / 限流

MySQL/Redis、缓存策略

SpringBoot、参数校验、异常处理

快速对齐 Agent

Agent = LLM 大脑 + 工具调用 + 记忆 + 规划调度

后端服务 = Agent 的执行器，Agent 做决策，你写逻辑

推荐教程

地址：https://www.bilibili.com/video/BV11NNAz5EKn/，看完这篇教程之后能够让你对AI Agent 入门且和后端知识快速对齐。

二、第一阶段：Agent 核心基础

1. LLM 大模型基础

这部分面试必问，对于后端来说不需要像算法岗那样搞学术，只记工程点就行。

大模型能力边界：生成、理解、FunctionCalling

提示词工程：Zero-shot/Few-shot/CoT/ 结构化输出

国产模型：通义千问、文心一言、DeepSeek、豆包

接口调用：API Key、流式输出、Tokens 计费

必看视频

LLM 入门 + 提示词工程实战

地址：https://www.bilibili.com/video/BV1n9CwYoEro/?spm_id_from=333.337.search-card.all.click&vd_source=e029be45bf88fb0e108467d3b4e2f10c 这部分快速过一下能够快速理解提示词工程在实际开发中是如何用的就行。

2. Agent 核心四组件

感知：理解用户意图

记忆：短期记忆 / 长期记忆 / 向量记忆

规划：拆任务、做决策

执行：调工具、调 API、写库表

核心范式 ReAct

Reasoning → Acting → Observation → 循环

面试常考题：为什么 ReAct 比单纯 LLM 更稳？

必看视频

Agent 四组件

强推up:马克的技术工作坊https://space.bilibili.com/1815948385?spm_id_from=333.337.search-card.all.click，他的作品里面从0到1讲解了很多诸如Agent、skills的概念，对于新手小白来说特别友好。

3. 工具调用与 FunctionCalling

函数定义、入参出参、校验

工具注册、路由、异常捕获

实战：调天气、数据库、Redis、支付接口

第二步看完了之后，就可以看下ReAct和CodeAct-Agent调用外部工具的两种方法，从原理到实战了。

推荐教程：

地址：https://www.bilibili.com/video/BV1wXZfYRErX/?spm_id_from=333.337.search-card.all.click&vd_source=e029be45bf88fb0e108467d3b4e2f10c

三、第二阶段：Agent 开发框架核心

1. LangChain（必学）

这部分是Agent 开发事实标准，后端必掌握。

Chain、PromptTemplate、OutputParser

工具封装、AgentExecutor、记忆组件

接入国产大模型、本地模型

必看视频

可以去看下尚硅谷的LangChain 全套教程（2026 最新），不需要全部看完，快速跳着看就行https://www.bilibili.com/video/BV1ZppNzHEY4/?spm_id_from=333.337.search-card.all.click&vd_source=e029be45bf88fb0e108467d3b4e2f10c。

2. LangGraph（进阶必学）

LangGraph 是一个用于构建复杂 AI 应用的工作流编排框架，属于 LangChain 生态体系的一部分，但也可独立使用。

状态节点、条件跳转、循环、中断恢复

多步骤任务：报销、审批、研报生成

对比：LangChain vs LangGraph

必看视频

可以快速看下挑战19分钟搞定，LangGraph快速入门与原理剖析，这个视频：https://www.bilibili.com/video/BV1HVFLzfEeS/?spm_id_from=333.337.search-card.all.click&vd_source=e029be45bf88fb0e108467d3b4e2f10c

看完了之后会对LangGraph的概念有个初步认识，但是只是入门，如果要往深度研究可以去看下吴恩达机器学习的教程，讲的特别好！

https://www.bilibili.com/video/BV1ancBzjEWi/?spm_id_from=333.337.search-card.all.click&vd_source=e029be45bf88fb0e108467d3b4e2f10c

3. 多 Agent 框架

多Agent框架是一种用于构建和管理多个智能体（Agent）协作系统的技术架构，旨在通过分工协作解决复杂问题。

CrewAI：角色分工、任务调度

https://www.bilibili.com/video/BV1JXawzqEtA/?spm_id_from=333.337.search-card.all.click

AutoGen：多轮对话、人机协同

https://www.bilibili.com/video/BV1mK41147Wx/?spm_id_from=333.337.search-card.all.click

场景：代码生成、舆情分析、智能

四、第三阶段：Agent 关键技术

1. RAG 检索增强生成

把LLM基础和核心组件吃透之后，接下来就要攻克Agent工程落地最核心、面试必考的关键技术——RAG检索增强生成，这部分也是Java后端同学转Agent最容易建立优势、快速做出落地成果的模块，哪怕是刚接触AI领域，靠着后端已有的存储、检索、接口开发功底，也能快速上手，不用死磕复杂算法和模型原理，完全聚焦工程实战。

可以先看下这个教程快速理清楚RAG的基本概念。

https://www.bilibili.com/video/BV1JLN2z4EZQ/?spm_id_from=333.337.search-card.all.click

很多后端同学刚接触Agent会疑惑，已经学会了大模型调用和工具调用，为什么非要单独学RAG，其实核心原因在于原生大模型存在两个无法规避的致命缺陷，直接导致纯LLM搭建的Agent只能停留在Demo阶段，根本无法满足企业级场景的落地要求，这也是RAG成为Agent岗位必学技术的核心背景。

一方面，大模型的训练数据是固定且滞后的，对于企业内部的私有文档、实时业务数据、最新行业资讯、内部知识库这类未进入训练集的内容，模型完全没有认知，回答出来的内容要么脱离实际业务，要么完全过时，根本没法用在客服问答、内部知识库查询、业务数据解读等真实场景。

另一方面，大模型天生存在幻觉问题，会在没有依据的情况下编造信息、扭曲事实，放到企业场景里，轻则导致回答错误影响业务，重则引发合规风险和客户投诉，而RAG就是解决这两大问题的最优解，它不需要微调模型、不改动模型权重，只通过外部知识库检索真实可信的信息，再把检索到的事实依据和用户问题一起交给大模型，让模型基于现有资料生成回答，从根源上杜绝幻觉，同时实现实时知识更新，兼顾安全性和实用性。

对于后端开发者来说，RAG本质上就是把我们熟悉的数据库查询、搜索引擎检索、缓存优化等技术，和大模型生成能力结合起来，完全契合后端的技术思维，学习成本远低于其他AI技术。

从核心逻辑来看，RAG的整套工作流程和后端常规的业务处理流程高度相似，没有复杂的新概念，理解起来非常快，整体可以拆解为一套完整的闭环流程，每一步都能对应后端已有的技术栈。

首先是文档加载与预处理环节，需要把企业内部的PDF、Word、Excel、markdown文档，甚至数据库中的业务数据、接口返回数据统一读取进来，完成格式清洗和文本规整，这一步和后端做数据导入、文件处理的逻辑完全一致；接下来是文本切片处理，也就是把过长的文本拆分成大小合适的文本块，同时设置合理的重叠窗口，避免切片导致语义断裂，这一步类似于后端做数据分片、分页处理，需要把控切片大小和重叠比例，保证后续检索的准确性；然后是向量化嵌入环节，通过开源的嵌入模型把文本块转换成数值向量，这一步不需要自己训练模型，直接调用成熟的开源模型接口即可，重点是理解向量相似度匹配的逻辑，不用深究模型底层原理；之后是核心的检索召回环节，把用户的提问同样转换成向量，通过相似度计算从向量库中匹配出最相关的文本片段，这一步就相当于后端做索引查询、精准检索，还能结合关键词检索做混合检索，提升召回准确率，后续还能通过重排模型优化排序，过滤掉不相关内容；最后是提示词拼接与生成环节，把检索到的有效上下文和用户问题整合进提示词，再调用大模型生成合规、准确的回答。

要系统掌握RAG，可以推荐大家看下这个教程。手把手带你搭建一套完整的RAG系统。

https://www.bilibili.com/video/BV1ei421a7Kc/?spm_id_from=333.337.search-card.all.click&vd_source=e029be45bf88fb0e108467d3b4e2f10c

2. 记忆系统

短期：ConversationBufferMemory

长期：Summary、向量存储

后端结合：Redis 存会话、MySQL 存历史

必看视频

Agent 记忆机制 + 代码实现

https://www.bilibili.com/video/BV1L31XBRE6m/?spm_id_from=333.337.search-card.all.click

五、第四阶段：项目实战

后端转 Agent 必做：2 个可写简历项目，

项目 1：企业级智能客服 Agent（基础）

技术栈：SpringBoot + LangChain + Redis + MySQL + RAG

功能：意图识别、多轮对话、订单查询、退款流程

亮点：工具调用、会话记忆、限流熔断、日志监控

B 站实战

智能客服 Agent 全流程开发https://www.bilibili.com/video/BV1cTcMzyE6P/?spm_id_from=333.337.search-card.all.click

项目 2：多 Agent 协同研报助手（进阶，冲大厂）

技术栈：LangGraph + CrewAI + 向量库 + 文档解析

角色：采集 Agent→分析 Agent→写作 Agent→审核 Agent

亮点：工作流编排、多智能体协同、结构化输出

B 站实战

多 Agent 研报生成项目https://www.bilibili.com/video/BV14eqcBzENn/?spm_id_from=333.337.search-card.all.click

这里推荐的项目教程，核心作用是帮大家快速入门、理清基础流程，本质上都是单机轻量化Demo，只覆盖了核心链路的基础演示，完全达不到企业生产环境的上线标准，这一点后端同学一定要提前分清，千万不能把Demo的简易开发思路直接照搬到实际项目中，避免上线后频繁踩坑。

对于后端转Agent的开发者来说，企业级RAG系统从来不是简单的文档切片加向量检索再加模型生成，而是要结合咱们成熟的后端工程经验，补齐高可用、高并发、数据安全、性能优化、稳定性兜底等全套生产级能力。

数据层面的工程化处理是第一道门槛，Demo里往往只用几份干净规整的文档做测试，而企业真实场景要面对海量杂乱的非结构化数据，涵盖PDF扫描件、复杂表格、带图片文档、加密文件、超长合同以及多格式混合内容，必须搭建标准化的文档解析与清洗链路，做好格式兼容、噪声过滤和敏感信息脱敏，同时要支持增量数据同步、定时更新和版本管理，不能像Demo那样一次性导入后就不再维护，还要搭配细粒度权限控制，限定不同角色的检索范围，从根源规避企业核心数据泄露的风险。

企业级RAG要承接多用户高并发访问，必须做全链路性能优化，比如用Redis缓存高频查询和向量检索结果，降低向量库重复计算压力，优化向量库索引策略，保障百万级数据下的检索速度，严格控制上下文窗口和召回文本长度，避免模型响应超时，再配合线程池、异步调用、限流熔断这些后端常用手段，守住系统稳定性，防止流量峰值压垮服务。

企业场景要求极高的回答准确率，必须杜绝无效召回和幻觉反复，需要采用混合检索、语义重排、多路召回策略，精准过滤低相关内容，同时搭建完善的兜底机制，遇到检索无结果、召回内容不足或模型调用失败的情况，返回标准化提示而非异常报错，再接入全链路日志监控，追踪检索结果、响应耗时和异常信息，方便后续问题排查和效果迭代，完全复用后端运维监控的成熟思路。

还要支持模型、向量库、嵌入模型的灵活切换，适配不同场景的算力与成本需求，整体架构预留足够扩展空间，方便后续对接多Agent协同、工具调用等模块，真正和后端技术栈深度融合，刚好这也是后端同学转型做企业级RAG的核心优势。

像OpenClaw这类轻量化、适配工程落地的Agent框架（龙虾），更是印证了工程打底的重要性，它依托成熟的工程架构，让AI能力快速对接现有后端服务，省去冗余的底层调试，真正实现AI与业务的高效融合。

六、多读论文，保持好奇心

额外给大家整理好刚需学习资源，方便进阶提升：核心论文网站优先用arXiv、Google Scholar、Semantic Scholar，GPT、Claude、Transformer相关经典论文都能免费查阅，Transformer教程推荐Hugging Face官方文档、B站系统精讲与吴恩达相关教程，吃透注意力机制、模型架构等面试核心考点；高频面试题直接刷牛客网、掘金的LLM与Agent专题，重点攻克Transformer原理、Function Calling、RAG落地、ReAct范式、模型幻觉优化等必考内容

未来的后端岗位，必然是AI能力与工程能力深度绑定的复合型赛道，我们要主动拥抱AI Agent、RAG等前沿技术，补齐大模型调用、框架使用的核心技能！关注我@程序员花海，带你拥抱新技术不迷路!

#Agent##软件开发投递记录##你怎么看待AI面试##聊聊我眼中的AI##AI时代，哪些岗位最容易被淘汰#
