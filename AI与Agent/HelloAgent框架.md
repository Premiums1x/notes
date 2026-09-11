---
title: "HelloAgent框架"
date: 2026-07-22
section: "Agent"
source: http://120.77.152.123:8088/posts/HelloAgentFramework
tags:
  - "Agent"
  - "博客迁移"
---
HelloAgent构建的框架：

1. 在模型支持上通过`HelloAgentsLLM()`，构建通用大模型通信基底。
2. `Config`通用模型配置基类，能通过设置环境变量而非修改代码降低配置繁琐度，不论是大模型API或是本地部署的LLM，程序都能检测provider、api-key、base-url做识别匹配，自动配置默认参数。
3. `Message`消息系统基类，统一大模型通信格式、方便对话历史记录管理
4. `Agent`最顶层的通用Agent基底：

利用抽象类`abc`和抽象方法`abstractmethod`规范一个Agent的核心依赖：名称、LLM 实例、系统提示词和配置。

以及每个子类都必须实现的抽象方法、一些与其他组件建立联系的方法如涉及`Message`消息系统的新增、删除历史消息方法等等。

5. 可以通过重写hello-agent库已提供的经典Agent范式(ReAct 、Simple 、Reflection PlanAndSplve 、FunctionCall)进行重写，结合工具、消息等各种组件重构Agent逻辑。

6.工具系统：

- `Tool`基类，用抽象类和抽象方法规定每个工具统一的接口，如：让每个接口有统一的运行方法，xxx.run() d等等
- `ToolParameter`基类，继承BaseModel，规定工具所拥有的参数，能支持对工具的参数验证和文档生成
- `ToolRegistry`基类，是工具系统的管理中枢，它提供了工具的注册、发现、执行等核心功能。

直接通过Tool对象注册 / 将函数注册（后者需在注册时为函数构建一个工具的schema，包含name、description等元数据）

函数注册： `registry = ToolRegistry()`实例化工具注册表，然后调用`registry.register_function`将name、desc、func参数注册

除了通过函数方式构建工具外，还可 **用类的方式构建工具系统**：例子：构建集成taviily、sepapi API的多源搜索工具，相比函数方式，类方式更适合需要维护状态（如API客户端、配置信息）的工具。

**工具系统高级特性**：

1.工具链，借鉴*工作流* 的“图”的思想，抽象化一个`ToolChain`类，类似构建图的实例。每个节点都是一个`Tool`的调用，包含工具名称、描述、input模板（支持变量替换）和outputkey，输出的键，后续把输出的内容存入该键的value中。往`ToolChain`上添加节点以构建工具链，会按所添加节点的顺序去调用工具。

2. 异步工具支持： 对于耗时的工具操作，我们可以提供异步执行支持。

利用到`asyncio`、`concurrent.futures`库，定义异步任务执行器`AsyncToolExecutor`,支持`单异步工具调用执行`，await到调用结果后返回。以及*多个异步工具调用并行*，用List结构管理每个异步工具调用的发起以及结果存储，await并判断到所有异步工具调用都以得到结果，就用index、result去enumerate 按顺序打印出每个调用结果，**注意**，异步操作要注意线程的并发限制、调用完毕及时进行资源清除，以防止溢出！！
