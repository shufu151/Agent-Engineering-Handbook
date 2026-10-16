# 第三章｜Agent 核心组件：从能力边界到可靠执行闭环

> <strong>章节定位：</strong>本章是 Agent 工程体系的组件层与运行时衔接层。六篇正文分别解释 Prompt、Context、Memory、Tools、State 以及它们的集成方式；本 README 负责建立全局知识地图、串联因果关系并提供深入阅读入口，不替代各专题正文。
>
> <strong>阅读主线：</strong>行为约束 → 信息组织 → 跨会话复用 → 外部执行 → 过程控制 → 系统集成。

## 1. 为什么必须把五个组件放在同一个体系中理解

大语言模型（Large Language Model，<strong>LLM</strong>）擅长理解、生成与辅助决策，但企业任务要求的不是一次回答，而是可执行、可验证、可恢复的结果。Prompt 让模型知道应当遵循什么规则，Context 决定本轮能依据哪些信息，Memory 提供未来可复用的经验，Tools 把决策连接到真实系统，State 记录可核验的执行事实。<strong>Agent Runtime（智能体运行时）</strong>负责把这些能力协调为闭环，并在模型输出与实际执行之间设置安全边界。

### 图 1｜章节知识全景思维导图

```mermaid
mindmap
  root((Agent核心组件))
    行为与信息
      Prompt
        Instructions
        指令优先级
        信任边界
      Context
        Context Builder
        Token Budget
        检索与压缩
    经验与执行
      Memory
        短期与长期
        写入与检索
        更新与遗忘
      Tools
        Schema契约
        Function Calling
        执行与反馈
    过程与集成
      State
        状态机
        Checkpoint
        恢复与幂等
      Runtime
        控制流与数据流
        质量验证
        权限与观测
```

<strong>读图：</strong>前三组分别解决行为与输入、经验与行动、执行连续性与系统协调。它们不是相互独立的“插件”，而是在同一次任务中持续交换数据和控制信号。

### 图 2｜从模型能力到系统能力的因果链

```mermaid
flowchart LR
  A[LLM理解与生成] --> B[Prompt约束行为]
  B --> C[Context提供当前依据]
  C --> D[Memory复用长期经验]
  D --> E[Tools访问外部系统]
  E --> F[State记录执行事实]
  F --> G[Runtime组织闭环]
  G --> H[可交付的任务结果]
  H -.质量反馈.-> G
```

这条链是<strong>学习依赖顺序</strong>，不是运行时必须逐个串行执行的固定流程。实际执行中，State、Memory 和 Context 会在多轮循环中反复协作。

## 2. 六篇知识文档的学习导航

| 顺序 | 专题与相对链接 | 一句话核心问题 | 学完后应理解 |
|---|---|---|---|
| 01 | [Prompt、系统指令与提示词设计](./01-Prompt-and-Instructions.md) | 模型应遵循什么规则？ | 指令来源、优先级、结构化约束及其安全边界 |
| 02 | [上下文工程与上下文窗口管理](./02-Context-Engineering.md) | 本轮应该让模型看到什么？ | Context Builder、Token Budget、选择、检索、压缩与更新 |
| 03 | [短期记忆、长期记忆与工程化管理](./03-Memory.md) | 哪些信息值得未来继续使用？ | 记忆分类、写入、检索、冲突治理与遗忘 |
| 04 | [工具契约、调用与结果反馈](./04-Tools.md) | 决策怎样转化为真实操作？ | Tool Schema、注册、Function Calling、MCP、可靠性与权限 |
| 05 | [状态、会话、持久化与故障恢复](./05-State-Management.md) | 任务执行到了哪里，如何安全继续？ | State Schema、状态机、Checkpoint、幂等与恢复 |
| 06 | [核心组件集成与执行闭环](./06-Component-Integration.md) | 五个组件如何组成完整 Agent？ | Runtime、接口契约、控制流、数据流与端到端治理 |

<strong>推荐路径：</strong>初学者按 01 → 06 顺序阅读；已有 Agent 开发经验的读者可先读 [06 集成](./06-Component-Integration.md) 定位系统，再按问题返回相应专题。六篇文档的知识目标不同，README 仅提供整体地图和关键边界。

### 图 3｜学习依赖与交叉引用

```mermaid
flowchart TD
  P[01 Prompt] --> C[02 Context]
  C --> M[03 Memory]
  C --> T[04 Tools]
  M --> I[06 Integration]
  T --> S[05 State]
  S --> I
  P --> I
  C --> I
  T --> I
  S -.恢复信息进入.-> C
  M -.相关记忆注入.-> C
```

<strong>读图：</strong>Context 是模型输入的汇合点，State 是任务事实的记录点，Integration 则解释两者如何与工具执行和记忆复用衔接。

## 3. 五个组件的职责边界：最容易混淆的概念

| 组件 | 回答的问题 | 典型输入或数据 | 不应承担的职责 |
|---|---|---|---|
| <strong>Prompt / Instructions</strong> | 应怎样做？ | 角色、目标、规则、输出约束 | 不能替代实际鉴权、事务和结果校验 |
| <strong>Context</strong> | 本轮能看到什么？ | 当前用户请求、状态摘要、检索片段、工具反馈 | 不能充当唯一持久化存储 |
| <strong>Memory</strong> | 未来要复用什么？ | 经筛选的偏好、事实、历史经验 | 不等于完整聊天记录或权威业务数据库 |
| <strong>Tools</strong> | 能实际执行什么？ | 工具契约、参数、身份、结果 | 模型生成调用意图不等于执行成功 |
| <strong>State</strong> | 当前执行事实是什么？ | 阶段、进度、错误、审批、产物引用 | 不等于模型“记住”了任务 |

这里还要区分 <strong>RAG（Retrieval-Augmented Generation，检索增强生成）</strong>：它是从外部知识源检索并补充模型输入的方法，可以服务于 Context，也可以用于 Memory 检索，但不等同于 Memory 本身。<strong>Checkpoint（检查点）</strong>则是支持状态恢复的持久化依据，不能保证所有外部副作用天然只执行一次。

### 图 4｜组件职责与数据所有权

```mermaid
flowchart TB
  U[用户任务] --> R[Agent Runtime]
  P[Prompt规则] --> CB[Context Builder]
  S[(State任务状态)] --> CB
  M[(Memory长期经验)] --> MR[Memory Retriever]
  MR --> CB
  K[(企业知识库)] --> KR[RAG检索]
  KR --> CB
  R --> CB
  CB --> L[LLM]
  L --> D{模型建议动作}
  D -->|工具请求| X[Tool Executor]
  X --> T[业务工具或API]
  T --> O[执行结果]
  O --> S
  O --> R
  D -->|直接回答| V[结果验证]
  V --> R
```

<strong>读图：</strong>长期经验和企业知识可以共同参与本轮上下文，但它们来源、可信度和更新机制不同。工具执行后的真实结果优先成为 State 中的执行事实，再由运行时决定是否注入下一轮 Context；不是所有工具结果都应自动写成长期 Memory。

### 图 5｜Context、Memory、State 的时间维度

```mermaid
flowchart LR
  A[历史会话与经验] --> M[(Memory跨会话)]
  B[当前任务执行] --> S[(State随步骤变化)]
  M --> C[本轮Context]
  S --> C
  D[当前用户输入] --> C
  C --> L[一次LLM调用]
  L --> E[下一步建议]
  E --> B
  B -.筛选并确认价值.-> M
```

<strong>读图：</strong>Context 是每轮调用的“视图”，State 是当前任务的“事实”，Memory 是跨任务的“可复用资产”。三者可以共享底层存储，却不应混淆生命周期和写入策略。

## 4. 工作原理：Runtime 如何把组件组织成闭环

<strong>Runtime（运行时）</strong>不是第六个知识仓库，而是负责执行循环、工具调度、状态转移、错误处理与终止判断的协调层。它需要同时管理两条链：<strong>Control Flow（控制流）</strong>决定下一步做什么；<strong>Data Flow（数据流）</strong>决定什么信息流向哪个组件。

### 图 6｜一次 Agent 任务的完整控制流

```mermaid
flowchart TD
  A[接收任务] --> B[创建或恢复State]
  B --> C[构建Context]
  C --> D[LLM推理]
  D --> E{下一步动作}
  E -->|调用工具| F[Schema与权限校验]
  F --> G{校验通过?}
  G -->|是| H[Tool Executor执行]
  G -->|否| I[记录拒绝或请求授权]
  H --> J[核验结果并更新State]
  J --> K{任务达标?}
  K -->|否| C
  K -->|是| L[交付与归档]
  E -->|直接完成| N[输出质量验证]
  N --> K
  I --> K
  L --> M[按策略整理Memory]
```

<strong>读图：</strong>LLM 负责语义判断，程序负责权限、状态合法性、幂等与质量门槛。模型输出工具调用请求只是执行的开始，不能视为完成。

### 图 7｜模型、运行时与外部系统的时序协作

```mermaid
sequenceDiagram
  participant U as 用户
  participant R as Runtime
  participant S as State Store
  participant C as Context Builder
  participant M as Memory Store
  participant L as LLM
  participant T as Tool Executor
  U->>R: 提交任务
  R->>S: 初始化或恢复任务
  R->>C: 构建本轮输入
  C->>M: 检索相关经验
  M-->>C: 返回授权记忆
  C-->>R: 返回Context
  R->>L: 发送指令与上下文
  L-->>R: 返回工具调用建议
  R->>T: 校验并执行
  T-->>R: 返回结果或错误
  R->>S: 核验并保存状态
  R->>C: 更新下一轮依据
  R-->>U: 返回最终结果或待确认事项
```

<strong>读图：</strong>时序图强调的是调用责任边界。模型不直接写入状态数据库，也不因为工具反馈中出现命令式文本就获得额外权限。

### 图 8｜信息在一次模型调用前如何汇合

```mermaid
flowchart TB
  I[稳定Instructions] --> B[Context Builder]
  U[当前任务与用户输入] --> B
  S[State摘要] --> B
  M[筛选后的Memory] --> B
  K[RAG检索资料] --> B
  O[必要的Tool Observation] --> B
  B --> Q{容量与可信度检查}
  Q -->|保留相关内容| L[LLM输入]
  Q -->|冗余或超限| Z[压缩、卸载或暂不注入]
```

<strong>读图：</strong>Context Engineering 的价值在于<strong>按需装配</strong>而非无限堆叠。工具结果、网页和文档内容属于低信任数据，不应覆盖高优先级指令。

## 5. 关键机制：从可运行到可恢复、可验证

完整 Agent 必须处理执行失败、状态冲突、权限不足和结果不达标。它们不是孤立的工程优化，而是组件集成后必然出现的跨边界问题。

### 图 9｜故障恢复与幂等决策

```mermaid
flowchart TD
  A[任务中断或工具超时] --> B[读取Checkpoint与State]
  B --> C[检查外部产物或操作结果]
  C --> D{副作用已确认完成?}
  D -->|是| E[跳过重复操作并补记State]
  D -->|否| F{操作可安全重试?}
  F -->|是| G[使用幂等键有限重试]
  F -->|否| H[人工核对或补偿]
  E --> I[恢复后续流程]
  G --> I
  H --> I
```

<strong>读图：</strong>重试不是恢复的全部。<strong>Idempotency（幂等性）</strong>要求重复请求不会产生重复业务副作用；在状态更新与外部操作不能共用一个事务时，必须核对真实结果，必要时采用补偿或人工确认。详见 [05 状态管理](./05-State-Management.md) 与 [04 工具系统](./04-Tools.md)。

### 图 10｜可信执行的分层防护

```mermaid
flowchart TB
  A[用户输入与外部内容] --> B[来源与信任边界]
  B --> C[Prompt与Context组织]
  C --> D[LLM提出建议]
  D --> E[Schema与身份权限检查]
  E --> F{高风险操作?}
  F -->|是| G[人工审批]
  F -->|否| H[工具执行]
  G --> H
  H --> I[业务结果验证]
  I --> J[状态持久化与审计]
```

<strong>读图：</strong>Prompt 可以指导模型，但安全必须在 Runtime 与 Tool Executor 中独立实施。<strong>Schema（结构模式）</strong>约束参数形状，不保证业务语义正确或访问权限合法。

### 图 11｜端到端质量反馈闭环

```mermaid
flowchart LR
  A[用户目标] --> B[执行计划与组件调用]
  B --> C[工具产物]
  C --> D[质量验证]
  D --> E{达到验收标准?}
  E -->|否| F[定位错误并修复]
  F --> B
  E -->|是| G[交付结果]
  G --> H[追踪与评估]
  H -.改进规则与工具.-> B
```

<strong>读图：</strong>系统正确性需要从“模型是否回答得像样”扩展到“任务是否真实完成、产物是否符合业务要求”。<strong>Tracing（执行追踪）</strong>用于观察模型与工具调用，不能代替业务审计或任务持久化。

## 6. 实际应用：企业文档翻译 Agent 的组件协作

以医疗设备企业的 Word、PDF、图片翻译为例，任务要求不仅是翻译文本，还包括术语一致性、数字和型号准确、版式保留、漏译检查与可靠交付。适合采用<strong>确定性 Workflow（工作流）作为主干、Agent 负责局部动态判断</strong>的混合方案。

### 图 12｜文档翻译的业务主链与组件挂载点

```mermaid
flowchart TD
  U[上传Word PDF 图片] --> S[创建Task State]
  S --> P[解析文档与结构]
  P --> C[构建当前页Context]
  G[(企业术语库)] --> C
  M[(已确认的项目Memory)] --> C
  I[翻译Instructions] --> C
  C --> T[翻译Agent与工具]
  T --> Q[数字术语漏译与版式校验]
  Q --> D{质量通过?}
  D -->|否| R[局部修复或人工确认]
  R --> Q
  D -->|是| O[导出并核验文件]
  O --> F[更新State与交付]
  T -.页面进度.-> S
  Q -.检查结果.-> S
```

<strong>读图：</strong>术语库属于受控业务知识源，不应被模型自行总结的 Memory 覆盖。每页翻译与质量检查的进度属于 State；仅有稳定复用价值且经过验证的偏好或修复经验，才适合进入 Memory。

### 图 13｜第 35 页故障时的恢复链路

```mermaid
sequenceDiagram
  participant W as 翻译Worker
  participant D as State数据库
  participant O as 产物存储
  participant R as 恢复Worker
  W->>D: 记录第35页操作意图
  W->>O: 写入第35页翻译产物
  Note over W: 服务崩溃，状态可能尚未确认
  R->>D: 读取任务与检查点
  R->>O: 查询第35页产物及校验和
  O-->>R: 返回实际产物状态
  alt 产物有效
    R->>D: 补记完成并继续第36页
  else 产物缺失或无效
    R->>D: 标记待重试或人工处理
  end
```

<strong>读图：</strong>这个案例串联了 Tool 的外部副作用、State 的持久化、Runtime 的恢复决策和 Context 的后续更新，是本章知识体系的综合应用。

## 7. 工程决策：什么情况下重点复习哪一篇？

### 图 14｜按问题定位知识模块

```mermaid
flowchart TD
  A[Agent出现问题] --> B{主要表现}
  B -->|行为不符合规则| P[01 Prompt]
  B -->|信息缺失或噪声多| C[02 Context]
  B -->|跨会话经验丢失| M[03 Memory]
  B -->|工具失败或副作用| T[04 Tools]
  B -->|进度错乱或无法恢复| S[05 State]
  B -->|跨模块链路不稳定| I[06 Integration]
  P --> I
  C --> I
  M --> I
  T --> I
  S --> I
```

<strong>读图：</strong>这不是互斥分类。例如工具超时可能同时涉及 Tools 的重试策略、State 的执行记录与 Runtime 的错误分支。先定位主故障点，再沿依赖关系追查。

| 典型现象 | 优先检查 | 关联机制 |
|---|---|---|
| 模型忽略格式要求 | [Prompt](./01-Prompt-and-Instructions.md) | 指令结构、角色优先级、输出验证 |
| 对话越长效果越差 | [Context](./02-Context-Engineering.md) | Token Budget、信息筛选、压缩与上下文污染 |
| 新会话无法复用术语偏好 | [Memory](./03-Memory.md) | 写入、检索、命名空间、有效期 |
| 工具参数正确却访问被拒 | [Tools](./04-Tools.md) | Schema 与授权的区别 |
| 任务重启后重复导出文件 | [State](./05-State-Management.md) | Checkpoint、幂等键、外部结果核验 |
| 单组件测试通过但端到端失败 | [Integration](./06-Component-Integration.md) | 接口契约、状态更新、质量闭环与追踪 |

## 8. 本章知识关联与下一阶段的演进

五大组件是<strong>能力基础</strong>，Runtime 是<strong>协调机制</strong>，工作范式则是<strong>如何组织决策与执行步骤</strong>。因此，掌握本章之后，自然进入下一章的 ReAct、Planning、Workflow、Reflection 与模式选择，而不是再重复学习组件定义。

### 图 15｜从核心组件到工作范式

```mermaid
flowchart LR
  A[第三章 核心组件] --> B[Runtime与执行闭环]
  B --> C[第四章 Agent工作范式]
  C --> D[ReAct观察与行动循环]
  C --> E[Planning任务分解]
  C --> F[Workflow确定性编排]
  C --> G[Reflection与Verification]
  D --> H[按任务选择执行模式]
  E --> H
  F --> H
  G --> H
```

<strong>读图：</strong>ReAct 等范式不替代 Prompt、Context、Memory、Tools 或 State，而是在这些基础能力之上定义决策与执行的组织方式。下一章建议从 [04-Agent-Working-Patterns/01-ReAct.md](../04-Agent-Working-Patterns/01-ReAct.md) 开始。

## 9. 核心术语速查

| 术语 | 英文全称 | 在本章中的含义 |
|---|---|---|
| LLM | Large Language Model | 大语言模型，承担语义理解与生成、辅助决策 |
| Prompt | Prompt | 提供给模型的输入内容，包含指令及必要数据 |
| Instructions | Instructions | 规定模型行为、目标和约束的指令 |
| Context | Context | 本次模型调用可见的相关信息 |
| Context Builder | Context Builder | 选择、过滤、组织本轮输入的组件 |
| Token Budget | Token Budget | 在模型上下文容量约束下分配输入输出额度 |
| Memory | Memory | 选择性保存并在未来复用的信息机制 |
| RAG | Retrieval-Augmented Generation | 检索增强生成，将外部资料用于模型回答或决策 |
| Tool Calling | Tool Calling | 模型提出使用外部能力的调用请求 |
| Function Calling | Function Calling | 模型按函数工具定义生成结构化调用请求的机制 |
| MCP | Model Context Protocol | 模型上下文协议，标准化工具与资源连接 |
| Schema | Schema | 数据结构和字段约束定义 |
| State | State | 任务或执行过程中的结构化事实与控制条件 |
| Checkpoint | Checkpoint | 用于执行恢复的持久化检查点 |
| Idempotency | Idempotency | 重复操作不造成重复业务副作用的性质 |
| Runtime | Agent Runtime | 管理执行循环、工具调度、状态和异常的运行时 |
| Workflow | Workflow | 按预定义步骤和条件组织执行的工作流 |
| Tracing | Tracing | 记录模型、工具和节点执行过程的追踪机制 |
| Guardrails | Guardrails | 对输入、输出和操作施加的安全或质量约束机制 |

## 10. 本章总结

<strong>一条主线：</strong>Prompt 定义规则，Context 提供本轮依据，Memory 保存可复用经验，Tools 执行外部动作，State 记录可核验的任务事实；Runtime 将它们组织为<strong>决策 → 执行 → 观察 → 验证 → 更新 → 再决策</strong>的闭环。

<strong>两个边界：</strong>第一，模型提出建议不等于系统已执行；第二，模型看到的信息不等于可信的持久化事实。只有把权限、状态、结果验证和故障恢复放进运行时，Agent 才能从演示系统走向可靠的企业任务系统。

<strong>继续深入：</strong>依次阅读 [01 Prompt](./01-Prompt-and-Instructions.md)、[02 Context](./02-Context-Engineering.md)、[03 Memory](./03-Memory.md)、[04 Tools](./04-Tools.md)、[05 State](./05-State-Management.md) 和 [06 Integration](./06-Component-Integration.md)，再进入 [第四章 Agent 工作范式](../04-Agent-Working-Patterns/README.md)。
