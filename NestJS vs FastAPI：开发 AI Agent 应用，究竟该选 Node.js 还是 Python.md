假设我们用 NestJS 和 FastAPI 各写一个最简单的 Agent：

接收用户问题，调用大模型，执行一两个工具，再把结果流式返回给前端。

你会发现，两边都能做，代码量可能也没有想象中那么大。这时候讨论谁的性能更好、谁的语法更简洁，意义不大。

真正的差异要等项目准备上线之后才会出现。

当系统加入用户登录、权限、会话、文件处理、任务队列、失败重试、人工审批和运行记录之后，你面对的已经不是一个“大模型接口”，而是一个需要持续运行的后端系统。

所以，NestJS 和 FastAPI 的选择，不能只看谁更适合调用大模型。你要先弄清楚：项目的复杂度主要来自 AI 计算，还是用户、权限和业务流程？

我的判断是：**复杂度主要来自产品工程，NestJS 更合适；复杂度主要来自数据处理、RAG 和 Python AI 工具链，FastAPI 更合适。边缘部署和集群部署会调整选择的权重，但不会单独决定语言。**

下面讨论的，都是这句话的适用条件和例外。

![02-selection-complexity](https://george-1257136373.cos.ap-guangzhou.myqcloud.com/obsidianPictures/02-selection-complexity.png)

## NestJS 和 FastAPI 都不是 Agent 框架

NestJS 是一个 Node.js 后端框架，FastAPI 是一个 Python Web 框架。它们负责接收请求、校验参数、管理依赖、连接数据库和提供 API，但不会自动解决 Agent 编排问题。

一个能够投入实际使用的 Agent，通常包含下面几层：

```
用户和客户端
      ↓
鉴权、限流、租户、计费
      ↓
Agent 编排、工具调用、上下文管理
      ↓
任务队列、数据库、向量检索
      ↓
模型 API 或本地推理服务
```

NestJS 和 FastAPI 主要处在服务接口与业务编排这一层。

Agent 的工具调用、交接、状态恢复和运行追踪，往往还需要 Agent SDK、工作流框架或自己实现的执行循环。

![03-agent-system-layers](https://george-1257136373.cos.ap-guangzhou.myqcloud.com/obsidianPictures/03-agent-system-layers.png)

过去很多人选择 Python，是因为 Agent 框架和 AI 库主要集中在 Python 生态。这个判断现在需要更新。

TypeScript 已经有 OpenAI Agents SDK、Vercel AI SDK 和 Mastra 等可用方案。以 <font color="#D23F31">OpenAI Agents SDK</font>〔1〕〔2〕为例，它的 Python 和 TypeScript 版本都支持工具、结构化输出、Agent 交接和运行编排。

Python 侧则有 <font color="#D23F31">LangGraph</font>〔3〕、Pydantic AI，以及大量数据处理和评测工具。

因此，Node.js 已经可以独立完成不少 Agent 项目。只有当项目进入复杂状态编排、AI 数据处理或实验性工作流时，Python 工具链的选择才会明显多起来。

## 跨语言成本下降了，但没有消失

如果 NestJS 项目需要调用 Python 能力，过去通常要自己设计 RPC 接口、参数契约和错误格式。

现在可以把 Python 函数封装成 MCP Server，再由 TypeScript Agent 作为 Client 调用。

<font color="#D23F31">MCP</font>〔4〕统一了工具发现、参数 Schema、调用协议和错误表示，降低了跨语言暴露 Agent 工具的接口成本。

但它没有替你解决服务部署、鉴权、超时、链路追踪和大数据传输。

这意味着“需要使用 Python 库”不再等于“整个后端必须采用 FastAPI”。

如果 Python 调用非常频繁、对延迟敏感，或者要传输大量数据，直接使用 Python 仍然更省事。

反过来，如果只是偶尔调用 OCR、文件解析或检索工具，单独暴露服务也可以接受。

跨语言以后更容易拆，并不意味着第一版就应该拆。等资源、团队或部署边界真正出现，再增加第二套服务，通常更稳妥。

## FastAPI 更接近 AI 计算

FastAPI 最大的优势不是接口性能，而是它和 Python AI 生态之间几乎没有语言边界。

如果 Agent 需要处理下面这些任务，FastAPI 通常更顺手：

- 文档解析、切片和向量化
- OCR、语音和图像处理
- 本地 Embedding 或 Rerank
- 调用 PyTorch、Transformers 等 Python 库
- 运行数据分析和机器学习代码
- 快速验证新的 Agent 工作流

算法工程师写好的 Python 函数，可以直接封装成 Agent 工具，不需要先改写成 TypeScript，也不用额外建立一个跨语言服务。

FastAPI 同时支持异步接口和 WebSocket，调用模型 API、数据库和远程工具这类 I/O 密集型任务并不会成为它的短板。<font color="#D23F31">FastAPI Async</font>〔5〕对同步函数和异步函数的使用边界有比较完整的说明。

不过，“使用 FastAPI 就能方便地跑本地模型”只说对了一部分。

真到了稳定推理阶段，不少团队会把模型从 API 进程里搬出去，交给 vLLM、TGI 或 Triton 这类专用推理服务。

这时候 FastAPI 也只是在调用远程服务，与 NestJS 没有本质区别。

Python 真正省事的地方是模型前后的胶水代码：数据预处理、切片策略、检索融合、Rerank、评测脚本，以及算法同事经常调整的函数。

这些能力每天都可能变化，为每一个函数单独维护服务并不划算。

FastAPI 的自由度也会带来一个问题：项目很容易越写越散。

小项目里，一个路由文件同时负责提示词、模型调用、数据库写入和异常处理，短期很快，半年后就不一定好维护。

模块边界、依赖管理、队列、权限和服务治理，需要团队自己建立约束。

还有一点容易被忽略：FastAPI 自带的 `BackgroundTasks` 适合发送通知、写日志之类的小任务。

一个可能运行几分钟、需要重试和恢复的 Agent，不应该只依赖它。<font color="#D23F31">FastAPI Background Tasks</font>〔6〕也建议将重计算任务交给 Celery 等外部任务系统处理。

## NestJS 更接近产品工程

NestJS 的优势出现在业务开始变复杂之后。

它的模块、依赖注入、Guard、Interceptor 和装饰器，会迫使开发者给代码划分边界。对于包含用户、组织、权限、订阅、账单和操作审计的 AI SaaS，这套结构通常比自由组合更容易长期维护。

NestJS 还提供了相对完整的队列和微服务集成。<font color="#D23F31">NestJS Queues</font>〔9〕和 <font color="#D23F31">NestJS Microservices</font>〔10〕对 BullMQ、Redis、RabbitMQ、Kafka、NATS 和 gRPC 等通信方式都有对应支持，适合用来搭建复杂业务后台。

如果前端也是 TypeScript，NestJS 还有一个现实优势：前后端可以共享一部分类型、校验规则和工具代码。

对全栈团队来说，减少一种主要开发语言，往往比获得某个框架的局部性能优势更有价值。

NestJS 的问题也很明确。

<font color="#D23F31">NestJS SSE</font>〔11〕以 RxJS `Observable` 为核心。如果团队不熟悉 RxJS，加入工具状态、取消信号和自定义错误事件时，理解和调试成本可能高于直接使用异步生成器。

如果系统又严重依赖 Python 库，MCP 可以降低接口成本，却不会消除跨语言部署、监控和故障排查的成本。这部分要在选型时一起计算。

![04-ai-glue-vs-product-skeleton](https://george-1257136373.cos.ap-guangzhou.myqcloud.com/obsidianPictures/04-ai-glue-vs-product-skeleton.png)

## 部署方式会改变选择权重

![image.png](https://george-1257136373.cos.ap-guangzhou.myqcloud.com/obsidianPictures/20260901121308833.png)

“边缘部署”经常被混成一个概念，其实至少包含两种完全不同的环境。

### CDN Edge 更偏向 TypeScript

Cloudflare Workers 这类 CDN Edge Runtime 靠近用户，适合处理鉴权、限流、地域路由、缓存和请求转发。

在这类平台上，JavaScript 和 TypeScript 的运行时通常更成熟。以 Cloudflare Workers 为例，TypeScript 是一等支持语言，平台也提供了一部分 Node.js API 兼容能力。

<font color="#D23F31">Cloudflare TypeScript</font>〔13〕和 <font color="#D23F31">Node.js Compatibility</font>〔14〕列出了当前的支持范围。<font color="#D23F31">Python Workers</font>〔15〕目前仍处于 open beta，需要额外的兼容性标志，第三方包也要符合纯 Python 或 Pyodide 条件。

这里容易出现一个误判：TypeScript 适合 Edge，不等于完整 NestJS 适合 Edge。

NestJS 的模块初始化和 Node.js API 依赖会增加适配成本。

<font color="#D23F31">NestJS Serverless</font>〔12〕虽然提供了冷启动优化方案，但 Serverless Functions 和受限的 Edge Runtime 不是同一种环境。

CDN Edge 层最好保持轻量，只处理一小段流程：

```
鉴权 → 限流 → 请求预处理 → 地域路由 → 转发到 Agent 服务
```

Agent 的完整执行循环包含多轮模型调用、工具重试、人工审批和状态持久化，可能运行几十秒甚至几分钟。

把这些过程绑在一次边缘请求里，故障恢复会变得很麻烦。

### 设备侧 Edge 更偏向 Python AI 生态

门店服务器、工控机或用户本地设备也是边缘节点，但它们通常可以运行完整 Linux 和容器。

如果设备需要运行本地模型、OCR、语音识别或 Embedding，Python 能直接复用现有 AI 代码，FastAPI 更方便。

NestJS 更适合设备管理、协议转换、业务规则和离线同步。

所以，边缘部署确实会影响语言选择，但不能只得出“边缘选 JavaScript”。CDN Edge 更偏向 TypeScript，设备侧 AI 计算则更偏向 Python。

![05-cdn-edge-vs-device-edge](https://george-1257136373.cos.ap-guangzhou.myqcloud.com/obsidianPictures/05-cdn-edge-vs-device-edge.png)

### 集群部署不会直接决定语言

如果部署目标是 Docker 和 Kubernetes，NestJS 与 FastAPI 都可以构建镜像、运行多个副本，再由负载均衡器分发流量。

到了这里，工作负载比框架更重要。

NestJS 运行在 Node.js 事件循环上，适合大量网络请求和远程工具调用。

CPU 密集型任务应该交给 Worker Threads、任务队列或独立服务，避免阻塞接口进程。

FastAPI 可以通过 Uvicorn 同时处理多个请求。

<font color="#D23F31">FastAPI Containers</font>〔8〕建议在 Kubernetes 这类集群中，由集群负责复制容器，通常一个容器运行一个 Uvicorn 进程，而不是在每个容器内部再启动大量 Worker。

如果一个 Python 进程加载了 1GB 的机器学习模型，启动 4 个进程通常就要分别占用相应内存。<font color="#D23F31">FastAPI Deployment</font>〔7〕也提醒，多进程不会自动共享这些模型变量。

对于包含本地模型的系统，更合理的做法不是让每个 API Pod 都加载一份模型，而是把接口和推理拆开：

```
NestJS 或 FastAPI API Pod
          ↓
      任务队列
          ↓
Agent Worker Pod
          ↓
模型 API 或 GPU 推理 Pod
```

API Pod 负责接收请求和返回运行编号，Agent Worker 负责执行模型与工具调用，GPU Pod 专门处理推理。这样可以分别扩容，不会因为接口流量增加，就被迫复制昂贵的模型实例。

集群扩容也不能只盯着 CPU。Agent 的大量时间都在等待模型 API 或外部工具，CPU 使用率不高，不代表系统没有压力。等待任务数、运行时长、首 Token 延迟、模型并发限制、Token 消耗和推理队列长度，往往更能反映真实负载。

<font color="#D23F31">Kubernetes HPA</font>〔16〕可以根据 CPU、内存和自定义指标调整 Pod 数量，因此 NestJS 和 FastAPI 都能接入同一套扩容机制。

![06-independent-cluster-scaling](https://george-1257136373.cos.ap-guangzhou.myqcloud.com/obsidianPictures/06-independent-cluster-scaling.png)

## 两个框架都解决不了的生产问题

框架选型最容易盖住两件事：Agent 如何恢复，以及它被允许做什么。

### 状态恢复

当请求被负载均衡到另一个 Pod，或者原来的 Pod 被重启，内存里的对话状态、执行步骤和工具结果都会消失。会话、运行记录、幂等键和审批状态必须保存在数据库或其他外部存储中。流式连接中断后，客户端也应该能够通过 `run_id` 重新获取执行进度。

如果 Agent 会运行很久，或者需要等待人工审批，还要考虑持久化执行。它要求系统记录已经完成的步骤，在进程崩溃后继续运行，避免重新调用模型或重复执行工具。

![07-durable-agent-resume](https://george-1257136373.cos.ap-guangzhou.myqcloud.com/obsidianPictures/07-durable-agent-resume.png)

<font color="#D23F31">Temporal</font>〔17〕和 <font color="#D23F31">Restate</font>〔18〕是通用的持久化执行引擎；<font color="#D23F31">LangGraph Persistence</font>〔19〕通过检查点保存图状态，支持中断、恢复和人工介入。NestJS 和 FastAPI 本身都不提供这些能力。

### 工具权限

Agent 的工具调用，本质上是让模型参与决定系统要执行什么操作。哪些工具可以自动调用，哪些操作必须人工确认，不同租户能看到哪些工具，都要由权限系统控制。

用户上传的文档、检索到的网页和工具返回内容也可能包含提示词注入。系统需要区分数据与指令，并在删除、转账、发送消息等敏感操作前再次校验权限或请求确认。<font color="#D23F31">OWASP LLM Top 10</font>〔20〕对这类风险有系统整理。

这些问题和语言无关，但实现时会依赖权限模型、审计日志和租户系统。业务侧要求越复杂，NestJS 的工程结构越有价值；如果项目本来就是 Python 服务，FastAPI 同样可以完成，只是团队需要自己建立这些边界。

## 最后应该怎么选

如果项目大量使用 Python AI 库、本地模型、RAG 和数据处理，选择 FastAPI 通常更省事。

如果 Agent 是一个成熟 SaaS 产品中的功能，系统还有复杂的用户、权限、计费、队列和微服务，NestJS 更容易管理。

如果只是调用云端模型，再封装几个远程 API 工具，两者都能做好。团队熟悉哪一种语言，往往就是更重要的选择依据。

至于“用 NestJS 做业务后台，再用 FastAPI 做 Agent 服务”这套混合架构，我不建议第一版就上。

MCP 降低了后续暴露工具的接口成本，团队可以等 Python 工作负载需要独立部署、独立扩容，或者组织边界已经明确时再拆。

选择框架之前，可以先回答五个问题：

1. Agent 是否严重依赖 Python 专属库，而且调用足够频繁？
2. 项目的复杂度主要来自 AI 计算，还是用户、权限和业务流程？
3. 团队最熟悉 TypeScript 还是 Python？
4. AI 工作负载是否需要和业务接口独立扩容？
5. 单次执行会不会持续很久，是否需要中断恢复和人工审批？

如果第五个问题的答案是“会”，首先要确定的是状态和持久化执行方案，Web 框架反而排在后面。

答案如果还不明确，就先用团队最熟悉的框架完成第一版。等系统真的出现资源、组织或部署边界，再拆服务。

如果两边都能满足需求，我会优先选择团队熟悉的语言，并把更多时间留给状态恢复、权限、可观测性和成本控制。真正上线后，这些问题比框架之间的接口性能差异更容易让系统出故障。

## 参考资料

〔1〕<font color="#D23F31">OpenAI Agents SDK Python</font>：https://openai.github.io/openai-agents-python/agents/
〔2〕<font color="#D23F31">OpenAI Agents SDK TypeScript</font>：https://openai.github.io/openai-agents-js/
〔3〕<font color="#D23F31">LangGraph Overview</font>：https://docs.langchain.com/oss/python/langgraph/overview
〔4〕<font color="#D23F31">MCP Tools</font>：https://modelcontextprotocol.io/specification/draft/server/tools
〔5〕<font color="#D23F31">FastAPI Async</font>：https://fastapi.tiangolo.com/async/
〔6〕<font color="#D23F31">FastAPI Background Tasks</font>：https://fastapi.tiangolo.com/tutorial/background-tasks/
〔7〕<font color="#D23F31">FastAPI Deployment</font>：https://fastapi.tiangolo.com/deployment/concepts/
〔8〕<font color="#D23F31">FastAPI Containers</font>：https://fastapi.tiangolo.com/deployment/docker/
〔9〕<font color="#D23F31">NestJS Queues</font>：https://docs.nestjs.com/techniques/queues
〔10〕<font color="#D23F31">NestJS Microservices</font>：https://docs.nestjs.com/microservices/basics
〔11〕<font color="#D23F31">NestJS SSE</font>：https://docs.nestjs.com/techniques/server-sent-events
〔12〕<font color="#D23F31">NestJS Serverless</font>：https://docs.nestjs.com/faq/serverless
〔13〕<font color="#D23F31">Cloudflare TypeScript</font>：https://developers.cloudflare.com/workers/languages/typescript/
〔14〕<font color="#D23F31">Cloudflare Node.js</font>：https://developers.cloudflare.com/workers/runtime-apis/nodejs/
〔15〕<font color="#D23F31">Cloudflare Python</font>：https://developers.cloudflare.com/workers/languages/python/
〔16〕<font color="#D23F31">Kubernetes HPA</font>：https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
〔17〕<font color="#D23F31">Temporal</font>：https://docs.temporal.io/
〔18〕<font color="#D23F31">Restate</font>：https://docs.restate.dev/
〔19〕<font color="#D23F31">LangGraph Persistence</font>：https://docs.langchain.com/oss/python/langgraph/persistence
〔20〕<font color="#D23F31">OWASP LLM Top 10</font>：https://owasp.org/www-project-top-10-for-large-language-model-applications/
