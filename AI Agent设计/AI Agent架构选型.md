
# 简介

AI Agent架构反映了人工智能处理复杂任务的能力，核心在于如何组织内部模块以实现目标导向的行为。选择合适的Agent架构决定Agent的运行效率、可控性和准确性。这是我近期在做Agent设计时对各种架构的片面认知，欢迎捉虫。本文章用于指导读者了解Agent的工作模式，并对后续设计Agent工作流程提供帮助。这是一个系列文章，后续会针对Agent常见节点的规划、设计和提示词的构建展开讨论。文章不讨论提出某种架构的厂商做出的原生框架或工具，只讨论各种框架的优缺点或自建可能面临的问题。

# 架构介绍

## ReAct

ReAct架构采用一个“思考 -> 行动 -> 观察”迭代循环。Agent首先基于当前目标和已有信息进行思考，然后执行一个具体动作，例如调用一个工具或API，接着观察该行动产生的结果，并利用这个新信息进入下一轮的思考，直至任务完成。
由于每一步都包含大语言模型输出的思考记录，整个决策过程具有良好的透明度和可追溯性，便于调试和理解。

### 优点

良好的灵活性和适应性，能够根据实时获取的信息动态调整策略，非常适合处理探索性强、路径不确定的任务。

### 缺点

可能存在效率低下和错误累积的风险，如果初始判断失误或陷入无效循环，则可能导致资源浪费和最终失败。比如在该架构下问题分类节点对节点作出错误分类，所产生的影响会随着每次迭代遗留下来，得到的结果可能就会偏离预期，造成任务失败或错误地结束流程。


## Plan-and-Execute

Plan-and-Execute架构将任务分解为两个截然不同的阶段：规划和执行。Agent首先生成一个详细的、分步骤的执行计划，随后由一个独立的执行器严格按照此计划逐一完成各项子任务。

### 优点

P-a-E架构具有高准确性和可控性的特点，通过预先规划减少执行过程中的不确定性，避免了ReAct可能陷入的无效循环。

### 缺点

缺乏灵活性，同时，生成高质量计划本身计算成本高昂，整体任务的总成本通常高于ReAct，因为规划阶段会使用大量token作出语义解释生成方案。

## LLMCompiler

LLMCompiler可以解决函数调用密集型任务的并行化效率问题，内部结构主要由三个关键组件构成：规划器、代码生成器和执行引擎。当接收到一个包含多个潜在可并行执行函数调用的复杂请求时，规划器首先利用一次大型语言模型调用，解析用户意图并将其分解为一系列独立或具有依赖关系的子任务。随后，规划器会自动构建一个DAG来表示这些任务之间的依赖关系。代码生成器则根据此计划生成具体的、可执行的代码或指令序列，执行引擎负责调度并行运行这些任务。

### 优点

能够高效地处理大量工具调用，显著减少了不必要的大型语言模型交互。

### 缺点

这种设计的刚性也构成了其主要局限性，其协作范式被严格限定在“计划-执行”的编译器模式内。

## AutoGen

AutoGen的设计核心是事件驱动与模块化分层，AutoGen提出了一个三层架构：核心层、聊天层和扩展层。最底层的核心层是整个框架的基石，它是一个事件驱动的编程框架，为构建响应式的、异步的多智能体系统提供了基础能力，所以Agent能够对各种事件做出响应，而非陷入阻塞式的同步等待。中间的聊天层在此基础上封装了更高级的应用程序接口，用于快速构建标准的对话式Agent系统。最顶层的第一方扩展层则包含了各种Agent模板和各类实用的工具集，开发者可以直接复用或在此基础上进行二次开发。

### 优点

提升了框架的可维护性和可扩展性，既可以深入到底层核心实现高度定制化的交互逻辑，也可以利用高层API快速搭建功能完备的业务应用。

### 缺点

如果选择自建该框架则需要较长的开发周期保证核心层的事件驱动可用性，该架构更适合作为APP底座成为服务资源而非Agent应用。

## Multi-Agent Frameworks

### 协调架构

存在一个中央控制器，负责接收初始任务、将其分解并分配给各个专业智能体，同时监控任务进度、整合中间结果直至最终完成。

#### 优点

控制力强、状态清晰，易于管理和监督。

#### 缺点

中央节点可能成为性能瓶颈，并且一旦该节点发生故障，整个系统的协作就会中断。

### 去中心化架构

Agent之间没有指挥节点，通过消息传递协议直接相互通信和协调。

#### 优点

更具备弹性和容灾能力，因为单个Agent的失效不会导致整个系统崩溃

#### 缺点

增加了协调的复杂性，需要设计共识算法或协调协议来避免混乱。

# 选型决策框架

## 场景复杂度

- ReAct架构凭借轻量级迭代循环就可高效完成，避免过度设计带来的开销。

- Plan-and-Execute通过预规划建立任务拓扑结构，有效规避执行过程中的决策漂移。

- LLMCompiler的DAG调度能力可将串行任务转化为并行流水线，提升吞吐量。若涉及多角色协作（如数据分析+报告生成+可视化），则需引入Multi-Agent Frameworks的协调架构。

## 确定性要求

- Plan-and-Execute的“先规划后执行”模式通过人类可审阅的计划文档提供强约束，避免LLM在执行中突发创意。

- AutoGen的事件驱动架构亦可通过预设状态机实现确定性流转。

- ReAct在每轮迭代中保留思考记录，既允许一定灵活性，又可通过日志回溯定位问题。

- 去中心化Multi-Agent架构允许Agent间自由协商，适合在需要高容错性的场景，比如IOT设备协同、分布式训练、知识库操作，但需配套完善的监控与熔断机制。

## 成本敏感度

- LLMCompiler通过单次LLM调用生成完整执行计划，将N次工具调用压缩为1次规划+并行执行，显著降低Token消耗。

- ReAct在简单任务中因迭代次数少一样有成本优势。

- Plan-and-Execute的规划阶段虽然需额外LLM调用，但可以减少执行中的试错成本，总体成本与ReAct接近。

- Multi-Agent Frameworks的协调架构虽引入中央控制器开销，但通过专业化分工可以提升结果质量。

## 响应速度

- ReAct的单步决策延迟最低，适合工具调用链短的任务。

- LLMCompiler可以实现并行任务，端到端延迟优于串行执行。

- Plan-and-Execute在规划阶段具有额外的延迟，但执行阶段稳定可控。AutoGen的事件驱动架构支持异步任务处理，适合长周期任务

- Multi-Agent Frameworks的去中心化架构虽通信开销大，但可通过Agent并行工作缩短整体耗时。

## 可解释性

- ReAct的每步Thought-Action-Observation链条天然形成决策日志，便于追溯。

- Plan-and-Execute的计划文档可直接作为审计证据。

- LLMCompiler的DAG可视化可直观展示任务依赖关系，但代码生成环节存在黑盒风险。

- 去中心化Multi-Agent的涌现行为难以完全追溯，需额外设计消息日志与状态快照机制。

# 主流框架选择

## 轻量级单Agent场景

LangChain/Dify的ReAct实现，因其生态成熟、工具集成丰富，适合快速验证。

## 高并发工具调用场景

LLMCompiler，需要预定义工具依赖关系，对动态依赖场景需结合ReAct做运行时调整。

# 总结

我认为AI Agent架构选型的本质是在灵活性、确定性、效率三者间寻找平衡，是由场景驱动的抉择，要明确任务的工具调用复杂度、确定性要求、成本约束，再匹配架构，避免为用Multi-Agent而用Multi-Agent。先验证优于一步到位，从ReAct起步验证核心逻辑，当遇到规划缺失或问题时，再引入Plan-and-Execute或LLMCompiler的特定能力可以保证能力的快速迭代。根据我的经验来看，单一架构难以覆盖所有场景，实践中可以考虑多架构的融合策略，如需求分析、问题分类等节点使用Plan-and-Execute架构完成作为工具节点，但完整模型使用ReAct架构不断迭代得到用户需要的最终结果。根据我的模型优化经验来说，在生产中监控能力至关重要，完善的日志记录和检索可以避免工具陷入黑盒无从调整，推荐LangFuse作为监控平台，把日志接入ELK方便检索和分析。

Reference:

[ReAct vs Plan-and-Execute: A Practical Comparison of LLM Agent Patterns - DEV Community](https://dev.to/jamesli/react-vs-plan-and-execute-a-practical-comparison-of-llm-agent-patterns-4gh9)

[Multi-Agent Systems and Workflow Survey: Design Principles and Implementation Strategies for Modern AI Systems](https://www.researchgate.net/publication/395128299_Multi-Agent_Systems_and_Workflow_Survey_Design_Principles_and_Implementation_Strategies_for_Modern_AI_Systems)

[GitHub - SqueezeAILab/LLMCompiler: \[ICML 2024\] LLMCompiler: An LLM Compiler for Parallel Function Calling](https://github.com/SqueezeAILab/LLMCompiler)

[LLMCompiler](https://langchain-ai.github.io/langgraph/tutorials/llm-compiler/LLMCompiler/)

[ReAct 框架 \| Prompt Engineering Guide](https://www.promptingguide.ai/zh/techniques/react)

[AutoGen](https://autogen.microsoft.com/)

[AutoGen v0.4: Reimagining the foundation of agentic AI for scale, extensibility, and robustness - Microsoft Research](https://www.microsoft.com/en-us/research/blog/autogen-v0-4-reimagining-the-foundation-of-agentic-ai-for-scale-extensibility-and-robustness/)

[Query Data via SDKs - Langfuse](https://langfuse.com/docs/api-and-data-platform/features/query-via-sdk)
