# 日志、追踪、监控与审计：Agent 系统可观测性

> 所属章节：第七章「企业级工程与部署」  
> 文件建议名：`03-Observability.md`  
> 本文定位：系统化技术知识文档，覆盖核心概念、运行机制、工程实践、选型与常见误区。

## 1. 本章目标与核心结论

可观测性（Observability）是指通过系统对外暴露的运行数据，推断系统内部状态、识别异常并定位原因的能力。对于
Agent
系统，它不仅需要回答服务是否可用、请求为什么变慢、故障发生在哪里，还需要解释模型为何选择某个工具、检索结果如何影响生成、任务为何循环或失败，以及最终结果是否满足业务要求。

<strong>本章的核心结论：</strong>日志、指标、分布式追踪和审计记录各有职责，必须通过稳定的关联标识、统一的语义和明确的数据治理策略协同工作；运行轨迹还应与独立的质量评估结合，才能形成从异常发现到根因定位、结果验证和持续改进的工程闭环。

传统服务的观测重点通常是请求、资源和依赖；Agent
的观测对象则进一步扩展到模型调用、工具执行、检索、状态变化、多 Agent
交接、质量校验和业务结果。模型 API
返回成功，只能证明某次调用成功，不能证明整个任务成功，更不能证明结果正确。

## 2. 全局知识网络

``` mermaid
flowchart TB
    O(["Agent 可观测性"])

    O --> SIG["遥测信号"]
    O --> TRACE["执行轨迹"]
    O --> OPS["运维诊断"]
    O --> QUALITY["质量与业务结果"]
    O --> GOV["安全与治理"]
    O --> PLATFORM["技术平台"]

    SIG --> LOG["日志 Logging"]
    SIG --> METRIC["指标 Metrics"]
    SIG --> AUDIT["审计记录 Audit"]
    SIG --> PROF["性能剖析 Profiling"]

    TRACE --> SPAN["Trace / Span"]
    TRACE --> CTX["上下文传播"]
    TRACE --> FLOW["工作流与 Agent 步骤"]
    TRACE --> ASYNC["异步、并行与恢复"]

    OPS --> SLI["SLI / SLO"]
    OPS --> ALERT["告警与事件响应"]
    OPS --> RCA["根因分析"]
    OPS --> COST["性能与成本归因"]

    QUALITY --> EVAL["离线与在线评估"]
    QUALITY --> FEEDBACK["人工反馈与业务反馈"]
    QUALITY --> REGRESSION["质量回归"]

    GOV --> PRIVACY["脱敏与最小化采集"]
    GOV --> ACCESS["访问控制与保留策略"]
    GOV --> INTEGRITY["审计完整性与防篡改"]

    PLATFORM --> OTEL["OpenTelemetry"]
    PLATFORM --> COLLECTOR["Collector"]
    PLATFORM --> BACKEND["指标 / 日志 / 追踪后端"]

    LOG <--> SPAN
    METRIC <--> SLI
    SPAN <--> RCA
    FLOW <--> EVAL
    AUDIT <--> ACCESS
    PRIVACY <--> COLLECTOR
    COST <--> METRIC
    REGRESSION <--> ALERT

    classDef core fill:#16324F,color:#fff,stroke:#16324F,stroke-width:2px;
    classDef signal fill:#DCEBFA,color:#17324D,stroke:#6A9BCB;
    classDef trace fill:#DDF4E4,color:#17452B,stroke:#61A879;
    classDef ops fill:#FFF0D5,color:#6A4210,stroke:#D8A44D;
    classDef quality fill:#EDE2FA,color:#43256B,stroke:#9C7AC8;
    classDef governance fill:#FBE2E2,color:#6A2424,stroke:#CF7777;
    classDef platform fill:#E5E7EB,color:#252A31,stroke:#858B94;

    class O core;
    class SIG,LOG,METRIC,AUDIT,PROF signal;
    class TRACE,SPAN,CTX,FLOW,ASYNC trace;
    class OPS,SLI,ALERT,RCA,COST ops;
    class QUALITY,EVAL,FEEDBACK,REGRESSION quality;
    class GOV,PRIVACY,ACCESS,INTEGRITY governance;
    class PLATFORM,OTEL,COLLECTOR,BACKEND platform;
```

<strong>图谱解读：</strong>中心节点是 Agent
可观测性。它向外连接遥测信号、执行轨迹、运维诊断、质量评估、安全治理和技术平台。跨分支连线体现了实际工程中的依赖：追踪关联日志，指标支持
SLO
判断，执行轨迹为质量评估提供上下文，脱敏策略则应在采集与导出链路中生效。它不是一条固定的执行流程，而是一个持续互相反馈的观测网络。

## 3. 核心概念：四类数据及其边界

### 3.1 日志、指标、追踪与审计

| 类型       | 英文                | 主要回答的问题           | 常见内容                         | 主要用途                 |
|:-----------|:--------------------|:-------------------------|:---------------------------------|:-------------------------|
| 日志       | Logging             | 某个时间点发生了什么？   | 异常、业务事件、工具错误         | 解释具体事件             |
| 指标       | Metrics             | 系统整体表现如何？       | 延迟、错误率、请求量、Token 用量 | 趋势、告警、容量规划     |
| 分布式追踪 | Distributed Tracing | 一次任务经过了哪些组件？ | Trace、Span、父子关系、耗时      | 定位调用路径与关键耗时   |
| 审计记录   | Audit Logging       | 谁在何时执行了什么操作？ | 主体、资源、授权、操作结果       | 责任追溯、合规与安全调查 |
| 性能剖析   | Profiling           | 进程内部资源消耗在哪里？ | CPU 栈、内存分配、热点函数       | 定位代码级性能瓶颈       |

日志、指标和追踪通常被称为可观测性的三大信号。审计与性能剖析并不是简单的第四、第五种同类替代物：审计强调责任追溯与证据完整性，剖析强调代码和运行时资源的细粒度分析。

<strong>不要混淆运行日志和审计日志。</strong>运行日志可以依据采样、容量和保留策略进行管理；关键审计事件则应按照合规要求保障完整性、访问控制和防篡改能力，不能随意采样丢弃。

### 3.2 三大信号为什么必须关联

指标可能提示某模型的 P95
延迟升高；日志可能记录某工具出现超时；追踪可以确认该工具是否位于异常请求的关键路径上。只有关联查询，才能把”现象”还原为”原因”。

常见关联字段包括：

- `trace_id`：一次端到端执行的追踪标识。
- `span_id`：一次具体操作的标识。
- `parent_span_id`：当前 Span 的父操作标识。
- `request_id`：应用请求标识。
- `workflow_id` 或业务任务 ID：跨请求、队列和服务关联同一业务任务。
- 服务名、环境、版本、时间戳和执行状态：辅助筛选、聚合和版本比较。

并非所有字段都必须在每条数据中重复存储。应根据遥测格式、数据后端和查询场景选择字段，并通过
Trace Context（追踪上下文）传播必要的关联信息。

## 4. OpenTelemetry：统一遥测基础设施

OpenTelemetry（开放式遥测，简称
OTel）是一组用于生成、采集、处理和导出遥测数据的
API、SDK、规范与工具。它不是 Grafana，也不是某个固定的数据库或监控平台。

### 4.1 组件职责

``` mermaid
flowchart TB
    APP["业务应用与 Agent"]

    APP --> AUTO["自动插桩<br/>HTTP、数据库、框架"]
    APP --> MANUAL["手动插桩<br/>业务事件、Agent 步骤"]

    AUTO --> SDK["OpenTelemetry API / SDK"]
    MANUAL --> SDK

    SDK --> TRACE["Trace / Span"]
    SDK --> METRIC["Metrics"]
    SDK --> LOG["Logs"]

    TRACE --> PIPE["导出管线"]
    METRIC --> PIPE
    LOG --> PIPE

    PIPE --> COL["Collector（可选）"]
    COL --> PROC["过滤、脱敏、采样、批处理、路由"]
    PROC --> T["追踪后端"]
    PROC --> M["指标后端"]
    PROC --> L["日志后端"]

    T --> UI["查询、可视化与告警"]
    M --> UI
    L --> UI

    classDef app fill:#DCEBFA,stroke:#6A9BCB,color:#17324D;
    classDef sdk fill:#DDF4E4,stroke:#61A879,color:#17452B;
    classDef pipeline fill:#FFF0D5,stroke:#D8A44D,color:#6A4210;
    classDef backend fill:#EDE2FA,stroke:#9C7AC8,color:#43256B;

    class APP,AUTO,MANUAL app;
    class SDK,TRACE,METRIC,LOG sdk;
    class PIPE,COL,PROC pipeline;
    class T,M,L,UI backend;
```

<strong>图谱解读：</strong>自动插桩覆盖常见框架调用，手动插桩补充业务语义。SDK
负责创建与导出遥测数据；Collector
可在集中出口处执行过滤、脱敏、采样和路由；后端负责存储、查询和展示。Collector
并非所有部署都必需，但在多服务、多个数据后端或需要统一治理时通常有价值。

### 4.2 语义约定

Semantic
Conventions（语义约定）规定遥测字段的名称、类型和含义，降低不同语言、服务和后端之间的集成成本。生成式
AI（Generative AI，GenAI）相关语义约定正在扩展模型调用、Token
使用、检索等场景的属性定义。

常见观测属性：

| 对象       | 建议记录的属性                                 |
|:-----------|:-----------------------------------------------|
| 模型调用   | Provider、模型名、请求耗时、结束原因、调用状态 |
| Token 使用 | 输入 Token、输出 Token、缓存 Token（若可获得） |
| 工具调用   | 工具名、状态、耗时、重试次数                   |
| RAG 检索   | 查询标识、检索耗时、结果数、相关性分数         |
| Agent 运行 | 工作流名、任务 ID、迭代次数、执行状态          |
| 质量校验   | 校验类型、失败类别、评估结果                   |

GenAI 语义约定会随规范版本演进。工程实施前应核对字段的稳定性、SDK
支持情况和后端兼容性，不应把实验性字段当作永久稳定的接口。

## 5. 分布式追踪：Trace、Span 与上下文传播

### 5.1 Trace 和 Span 的关系

Trace（追踪）代表一次端到端操作；Span（跨度）代表其中一个操作及其时间范围。多个
Span 通过父子关系形成调用树。每个 Span
可以记录开始与结束时间、状态、属性、事件和异常信息。

例如，文档翻译任务可以包含 `translate_document` 根 Span，以及
`parse_pdf`、`retrieve_terms`、`llm_translate`、`validate_translation`
和 `render_pdf` 等子 Span。

当模型调用多个工具或并行执行子任务时，应保留合理的父子关系。并行任务的耗时会重叠，因此不能简单地把所有
Span 时长相加来计算端到端耗时；应通过时间线和依赖关系分析关键路径。

### 5.2 上下文传播

Context Propagation（上下文传播）是指将 Trace ID、Span ID
等追踪上下文从一个组件传递到另一个组件。常见方式包括 HTTP
Header、消息队列元数据和框架提供的上下文机制。

跨服务调用若没有正确传播上下文，追踪就可能断裂；但对异步任务、长时间运行的工作流而言，也不能假设所有步骤都处于同一个
HTTP 请求生命周期中。此时应使用稳定的业务任务 ID，并根据实际因果关系构建
Span 父子关系或 Span Link。

### 5.3 Trace 的工程价值

- 将总耗时分解为解析、检索、模型调用、工具执行和渲染等阶段。
- 找到异常请求的关键路径和耗时热点。
- 关联异常日志、指标峰值和具体执行步骤。
- 识别重复调用、重试风暴和状态恢复异常。
- 对比不同模型、提示词版本和工作流版本的表现。

## 6. Agent 专用追踪模型

### 6.1 为什么模型 API 日志不够

一次 Agent 任务可能包含多轮模型推理、工具调用、检索、状态更新、Agent
交接和失败恢复。即使每次模型 API 都成功，整个任务仍可能失败。

需要区分三个层次：

1.  **请求级观测**：模型 API 是否成功、耗时多少、消耗多少 Token。
2.  **步骤级观测**：工具是否正确调用、检索是否有效、状态是否发生预期变化。
3.  **任务级观测**：任务是否完成、结果是否正确、是否满足业务约束。

执行成功、业务成功和质量合格是三个不同的判断维度。

### 6.2 推荐的 Span 类型

| Span 类型       | 记录内容                         | 主要诊断问题          |
|:----------------|:---------------------------------|:----------------------|
| Workflow Span   | 任务 ID、总耗时、最终状态        | 整体任务是否完成      |
| Agent Span      | Agent 名称、角色、执行阶段       | 哪个 Agent 负责该步骤 |
| LLM Span        | 模型名、耗时、Token、结束原因    | 模型调用是否异常      |
| Tool Span       | 工具名、参数摘要、返回状态       | 工具是否失败或误用    |
| Retrieval Span  | 检索策略、结果数、分数分布       | 检索质量是否退化      |
| Handoff Span    | 来源 Agent、目标 Agent、交接状态 | 多 Agent 协作是否异常 |
| Validation Span | 规则结果、错误类型、修复次数     | 质量问题发生在哪里    |

### 6.3 Agent 执行轨迹网络图

``` mermaid
flowchart TB
    W(["Workflow Trace"])

    W --> PLAN["规划与路由"]
    W --> EXEC["执行与协作"]
    W --> STATE["状态与恢复"]
    W --> VERIFY["结果验证"]
    W --> OBS["成本与诊断"]

    PLAN --> ROUTER["Router / 意图判断"]
    PLAN --> MODEL["LLM 调用"]
    EXEC --> TOOL["Tool Span"]
    EXEC --> RETRIEVE["Retrieval Span"]
    EXEC --> HANDOFF["Agent Handoff"]
    STATE --> CHECKPOINT["Checkpoint"]
    STATE --> RETRY["重试与恢复"]
    STATE --> LOOP["循环次数与退出原因"]
    VERIFY --> RULE["规则校验"]
    VERIFY --> EVAL["质量评估"]
    OBS --> TOKENS["Token 与调用成本"]
    OBS --> LATENCY["耗时与关键路径"]

    MODEL --> TOOL
    RETRIEVE --> EVAL
    TOOL --> RULE
    CHECKPOINT --> RETRY
    LOOP --> LATENCY
    RULE --> EVAL
    EVAL --> W

    classDef core fill:#16324F,color:#fff,stroke:#16324F,stroke-width:2px;
    classDef plan fill:#DCEBFA,color:#17324D,stroke:#6A9BCB;
    classDef execute fill:#DDF4E4,color:#17452B,stroke:#61A879;
    classDef state fill:#FFF0D5,color:#6A4210,stroke:#D8A44D;
    classDef verify fill:#EDE2FA,color:#43256B,stroke:#9C7AC8;
    classDef observe fill:#FBE2E2,color:#6A2424,stroke:#CF7777;

    class W core;
    class PLAN,ROUTER,MODEL plan;
    class EXEC,TOOL,RETRIEVE,HANDOFF execute;
    class STATE,CHECKPOINT,RETRY,LOOP state;
    class VERIFY,RULE,EVAL verify;
    class OBS,TOKENS,LATENCY observe;
```

<strong>图谱解读：</strong>工作流 Trace
不是单纯的模型调用列表，而是将规划、执行、状态、验证和诊断联系起来。模型调用可能驱动工具选择，工具和检索结果影响验证，状态恢复影响后续执行；质量评估结果又可以反向关联到整条工作流。对于异步或并行任务，图中的逻辑关系应按真实执行因果建模，不能机械地假设所有节点都严格串行。

### 6.4 循环、重试、并行与状态恢复

应重点观测以下异常：

- **循环过多**：记录迭代次数、退出原因和累计耗时。
- **重复工具调用**：结合工具名、参数摘要和结果状态识别重复执行。
- **重试风暴**：区分首次失败、重试失败和最终失败。
- **并行 Agent**：保留实际并行时间段和依赖关系。
- **异步任务**：使用稳定的任务关联标识，避免上下文丢失。
- **状态恢复**：记录 Checkpoint 标识、恢复位置和恢复后的执行结果。

对于长时间运行的任务，建议把工作流生命周期与单次 HTTP
请求生命周期分开。请求可能只负责创建任务，而真正的执行跨越多个队列消息、进程或服务。

## 7. 日志设计：结构化、可关联、可检索

### 7.1 结构化日志

建议采用结构化日志格式，而不是只输出自然语言字符串。常见字段包括：

- 时间戳、日志级别、服务名、环境和版本。
- Trace ID、Span ID、业务任务 ID。
- 操作类型、工具名称、模型名称。
- 执行状态、错误分类、耗时和重试次数。
- 必要的业务维度及关联标识。

日志应支持从某个 Trace ID
反查相关事件，也应支持从一条错误日志跳转到对应的 Trace。

### 7.2 标签基数与日志索引

高基数字段（例如 Trace ID、用户 ID、任意 URL
或完整查询文本）不应不加区分地作为索引标签。以 Loki
为例，低基数标签适合描述服务、环境等稳定维度；高基数信息通常更适合放在日志正文或结构化元数据中。

错误的标签设计会导致索引膨胀、内存压力上升、查询性能下降和存储成本增加。字段是否适合做标签，应依据后端索引机制和真实查询模式决定。

### 7.3 日志级别与异常分类

| 级别             | 建议用途                                     |
|:-----------------|:---------------------------------------------|
| DEBUG            | 开发调试细节，生产环境谨慎启用               |
| INFO             | 正常业务阶段和关键状态变化                   |
| WARN             | 可恢复异常、降级或需要关注的情况             |
| ERROR            | 当前操作失败，需要调查或重试                 |
| FATAL / CRITICAL | 进程或关键业务能力无法继续（按框架约定使用） |

应优先采用稳定的错误分类，例如
`model_timeout`、`tool_permission_denied`、`retrieval_empty`、`validation_failed`，而不是依赖自由文本匹配所有异常。

## 8. 指标、SLO 与告警

### 8.1 四大黄金信号

Google SRE 提出的四大黄金信号是：

- **延迟（Latency）**：请求或任务耗时。
- **流量（Traffic）**：系统承载的请求量或任务量。
- **错误（Errors）**：失败请求、异常任务或错误结果。
- **饱和度（Saturation）**：资源接近容量上限的程度。

它们覆盖服务健康的主要维度，但 Agent
系统还需要补充任务完成率、迭代次数、质量校验失败率、Token
消耗和单位任务成本等指标。

### 8.2 指标类型

| 类型      | 含义             | 典型用途                         |
|:----------|:-----------------|:---------------------------------|
| Counter   | 只增不减的累计值 | 请求总数、错误总数、Token 总量   |
| Gauge     | 当前可升可降的值 | 活跃任务、队列长度、内存使用量   |
| Histogram | 观测值的分布统计 | 请求延迟、任务耗时、Token 数分布 |

Histogram 可以支持分位数和分布分析。对于延迟，不应只观察平均值；P95、P99
等高分位数能反映尾部请求的体验。实际使用时要理解直方图桶、聚合方式和后端查询语义。

### 8.3 Agent 指标建议

| 指标类别 | 示例                                      |
|:---------|:------------------------------------------|
| 工作流   | 任务成功率、完成耗时、平均迭代次数        |
| 模型     | 调用错误率、输入输出 Token、首 Token 延迟 |
| 工具     | 调用成功率、超时率、重试率                |
| RAG      | 检索延迟、空结果率、候选文档数量          |
| 质量     | 评估通过率、关键错误率、人工返工率        |
| 成本     | 单任务成本、Token 成本、工具调用成本      |
| 资源     | CPU、内存、队列积压、并发占用             |

### 8.4 指标与告警的关系网络

``` mermaid
flowchart TB
    M(["服务与 Agent 指标"])

    M --> GOLD["黄金信号"]
    M --> TASK["任务指标"]
    M --> MODEL["模型与工具指标"]
    M --> QUALITY["质量指标"]
    M --> COST["成本指标"]

    GOLD --> LAT["延迟"]
    GOLD --> TRAFFIC["流量"]
    GOLD --> ERR["错误"]
    GOLD --> SAT["饱和度"]

    TASK --> SUCCESS["任务成功率"]
    TASK --> ITER["迭代次数"]
    MODEL --> TOKEN["Token 使用"]
    MODEL --> TOOLERR["工具错误率"]
    QUALITY --> PASS["评估通过率"]
    QUALITY --> REWORK["返工率"]
    COST --> UNIT["单位任务成本"]

    LAT --> SLI["SLI / SLO"]
    ERR --> SLI
    SUCCESS --> SLI
    PASS --> SLI
    SAT --> CAPACITY["容量分析"]
    TOKEN --> UNIT
    ITER --> UNIT
    TOOLERR --> ALERT["告警与事件响应"]
    SLI --> ALERT
    CAPACITY --> ALERT
    REWORK --> ALERT

    classDef core fill:#16324F,color:#fff,stroke:#16324F,stroke-width:2px;
    classDef metrics fill:#DCEBFA,color:#17324D,stroke:#6A9BCB;
    classDef business fill:#DDF4E4,color:#17452B,stroke:#61A879;
    classDef decision fill:#FFF0D5,color:#6A4210,stroke:#D8A44D;

    class M core;
    class GOLD,LAT,TRAFFIC,ERR,SAT,MODEL,TOKEN,TOOLERR metrics;
    class TASK,SUCCESS,ITER,QUALITY,PASS,REWORK,COST,UNIT business;
    class SLI,CAPACITY,ALERT decision;
```

<strong>图谱解读：</strong>基础设施指标和 Agent 业务指标最终都应服务于
SLO、容量判断和告警处置。任务成功率下降可能来自模型、检索、工具或资源瓶颈；单位任务成本则与
Token、迭代次数和工具调用量共同相关。只观察 CPU 或模型 API
成功率，无法完整反映业务质量。

### 8.5 SLI、SLO 与错误预算

- SLI（Service Level Indicator，服务等级指标）：衡量实际服务表现的指标。
- SLO（Service Level Objective，服务等级目标）：对 SLI 设定的目标范围。
- SLA（Service Level
  Agreement，服务等级协议）：面向用户或客户的服务承诺，通常包含约定与责任。

Agent 的 SLO 应尽量以用户可感知的任务完成情况定义，而不只是模型 API
可用性。例如，可以定义在规定时间内完成且通过质量校验的任务比例。

错误预算用于表达在一个评估窗口内允许发生的服务目标偏差。它可以帮助团队在新功能发布与可靠性改进之间作出取舍。告警则应关联影响范围、持续时间、责任人和处置手册，避免只对瞬时阈值触发大量无行动价值的通知。

## 9. 日志、追踪、指标的关联诊断

### 9.1 从异常现象到根因的分析路径

``` mermaid
flowchart TB
    A(["发现异常"])

    A --> MET["指标：错误率或 P95 上升"]
    A --> LOG["日志：错误分类与异常事件"]
    A --> TRACE["追踪：调用路径与 Span 耗时"]

    MET --> SCOPE["确定影响范围与时间窗口"]
    LOG --> ERROR["识别异常类型"]
    TRACE --> PATH["定位关键路径与慢 Span"]

    SCOPE --> JOIN["按时间、服务和关联 ID 交叉验证"]
    ERROR --> JOIN
    PATH --> JOIN

    JOIN --> ROOT["形成可验证的根因假设"]
    ROOT --> FIX["修复、降级或回滚"]
    FIX --> VERIFY["验证指标、追踪和业务质量是否恢复"]
    VERIFY --> A

    classDef core fill:#16324F,color:#fff,stroke:#16324F,stroke-width:2px;
    classDef signal fill:#DCEBFA,color:#17324D,stroke:#6A9BCB;
    classDef analyze fill:#FFF0D5,color:#6A4210,stroke:#D8A44D;
    classDef action fill:#DDF4E4,color:#17452B,stroke:#61A879;

    class A core;
    class MET,LOG,TRACE signal;
    class SCOPE,ERROR,PATH,JOIN,ROOT analyze;
    class FIX,VERIFY action;
```

<strong>图谱解读：</strong>这是一张诊断流程图，而不是完整知识网络。它展示指标、日志和追踪如何互补：指标界定影响范围，日志给出错误线索，追踪还原执行路径；形成根因假设后必须修复并验证，避免把时间相关性误判为因果关系。

### 9.2 常见故障示例

假设文档翻译任务的 P95 延迟突然升高：

1.  先看任务完成耗时、模型耗时和解析耗时的趋势，判断延迟增加发生在哪个阶段。
2.  按 Trace ID 检查慢任务的 Span 时间线，识别关键路径上的长耗时操作。
3.  检查对应日志，确认是否发生超时、重试、队列等待或降级。
4.  对比模型版本、提示词版本、文档大小、检索结果数和并发情况。
5.  修复后重新观察延迟、成功率和质量指标，确认没有以降低质量或提高成本为代价。

## 10. 质量评估与可观测性的闭环

可观测性提供运行证据，Evaluation（评估）判断运行结果是否符合预期。两者应结合，但不能相互替代。

### 10.1 评估维度

| 维度         | 观测证据                     | 评估方式                 |
|:-------------|:-----------------------------|:-------------------------|
| 执行正确性   | Span、状态、异常日志         | 工作流规则与状态校验     |
| 检索质量     | 候选文档、相关性分数         | 标注数据集、检索指标     |
| 生成质量     | 模型输出、业务上下文         | 规则、人工评审、模型评审 |
| 工具使用质量 | 工具参数、返回值和调用轨迹   | 参数校验、执行结果检查   |
| 安全性       | 权限事件、策略命中记录       | 安全规则、红队测试       |
| 业务效果     | 任务完成率、返工率、用户反馈 | 业务指标与人工抽检       |

LLM-as-a-Judge（基于大语言模型的评审）可用于扩展线上评估，但需要使用人工标注集验证一致性、偏差和误判。评审模型的分数不能直接视为绝对真实的质量指标。

### 10.2 以文档翻译 Agent 为例

适合纳入质量监测的业务指标包括：

- 原文内容覆盖率。
- 数字、单位、型号及术语一致性。
- 表格、图片和图注保留率。
- 版面裁切、重叠和错位率。
- 人工修正比例。
- 每份合格文档的成本与完成时间。

这些指标应关联到对应任务的 Trace ID 或稳定业务任务
ID。这样，当某项质量指标下降时，工程人员才能回到具体失败步骤，判断问题来自解析、翻译、布局重建还是最终渲染。

### 10.3 质量反馈网络

``` mermaid
flowchart TB
    Q(["Agent 质量闭环"])

    Q --> RUN["生产运行轨迹"]
    Q --> DATA["评估数据集"]
    Q --> JUDGE["规则与模型评审"]
    Q --> HUMAN["人工反馈"]
    Q --> BUSINESS["业务结果"]

    RUN --> FAIL["失败样本与异常分类"]
    DATA --> BASELINE["基准质量与回归对比"]
    JUDGE --> SCORE["质量分数与错误类别"]
    HUMAN --> LABEL["人工标注与纠错"]
    BUSINESS --> KPI["完成率、返工率、成本"]

    FAIL --> ANALYZE["根因分析"]
    BASELINE --> ANALYZE
    SCORE --> ANALYZE
    LABEL --> ANALYZE
    KPI --> ANALYZE

    ANALYZE --> CHANGE["调整提示词、检索、工具或工作流"]
    CHANGE --> TEST["离线评估与回归测试"]
    TEST --> DEPLOY["灰度发布"]
    DEPLOY --> Q

    classDef core fill:#16324F,color:#fff,stroke:#16324F,stroke-width:2px;
    classDef evidence fill:#DCEBFA,color:#17324D,stroke:#6A9BCB;
    classDef analysis fill:#EDE2FA,color:#43256B,stroke:#9C7AC8;
    classDef improve fill:#DDF4E4,color:#17452B,stroke:#61A879;

    class Q core;
    class RUN,DATA,JUDGE,HUMAN,BUSINESS,FAIL,BASELINE,SCORE,LABEL,KPI evidence;
    class ANALYZE analysis;
    class CHANGE,TEST,DEPLOY improve;
```

<strong>图谱解读：</strong>质量闭环将线上轨迹、离线基准、模型评审、人工反馈和业务结果汇聚到根因分析，再通过回归测试与灰度发布验证改动。观测数据让问题可定位，评估数据让改动可比较，业务指标则判断改动是否真正有价值。

## 11. 审计、安全与隐私治理

### 11.1 运行日志与审计日志的区别

| 维度 | 运行日志               | 安全审计日志                           |
|:-----|:-----------------------|:---------------------------------------|
| 目标 | 排障和性能分析         | 责任追溯、合规证明                     |
| 内容 | 错误、耗时、运行状态   | 操作主体、目标资源、授权决策、操作结果 |
| 采样 | 可按容量和风险管理     | 关键审计事件应满足完整性要求           |
| 保留 | 根据调试价值与成本管理 | 按合规、风险和业务要求制定策略         |
| 权限 | 开发、运维按职责授权   | 通常需要更严格的访问控制与隔离         |

审计事件应根据业务风险记录操作主体、目标资源、操作类型、授权依据、执行结果、时间戳和关联标识。对于访问企业知识库、调用外部系统、修改业务数据或导出文件等敏感操作，应能回答：谁发起操作、授权是否通过、实际执行了什么、最终结果是什么。

### 11.2 数据最小化与脱敏

追踪数据可能包含提示词、检索片段、工具参数、模型输出、文档内容和个人信息。建议：

- 默认不记录完整提示词、原始文档和工具参数。
- 仅在经过审批的调试场景中开启内容捕获。
- 对密码、令牌、个人信息和业务机密执行过滤或脱敏。
- 对模型输入输出设置独立访问权限与保留期限。
- 将追踪元数据与业务内容分开管理。
- 对高风险操作记录授权、审批和执行结果。
- 定期检查异常日志是否意外输出密钥、文档内容或完整请求体。

<strong>关键取舍：</strong>完整内容追踪有利于复现模型行为，但会扩大数据泄露面。企业环境应根据环境、租户、任务类型和调试授权实施分级采集，而不是默认记录所有模型输入输出。

### 11.3 治理策略应进入数据管线

脱敏不能只依赖开发人员自觉。应把数据过滤、敏感字段清除、采样、权限控制和保留策略纳入遥测采集链路，并通过测试验证规则确实生效。对于审计要求较高的系统，还应考虑独立存储、访问审查、完整性保护和留存证明。

## 12. 采样、性能开销与成本控制

可观测性本身会消耗
CPU、网络、存储和查询资源。若每次模型调用都记录完整输入输出，并为每个用户或
Trace 创建大量高基数标签，可能造成显著成本和隐私风险。

### 12.1 常见优化手段

- **采样（Sampling）**：控制需要完整保留的追踪比例。
- **批处理（Batching）**：减少频繁导出造成的开销。
- **过滤（Filtering）**：删除低价值或不允许采集的字段。
- **聚合（Aggregation）**：在指标层聚合数据，避免保存不必要的高频原始值。
- **分级保留（Retention
  Tiers）**：按数据价值和合规要求设置不同保留周期。
- **动态调试策略**：在发生异常时对选定任务提高追踪细节，但必须受权限和隐私策略约束。

头部采样（Head Sampling）通常在追踪开始时决定是否采集；尾部采样（Tail
Sampling）则可以基于完整或部分追踪特征作决定，例如保留错误和慢请求。尾部采样通常需要在
Collector 或其他集中处理环节缓存并汇集
Span，因此会增加内存、处理复杂度和数据等待时间。

### 12.2 采样不是审计的替代品

普通性能追踪可以采样，但关键安全审计事件不能简单套用相同的采样策略。应分别定义调试追踪、业务指标和审计记录的采集要求，并验证各自的数据完整性。

### 12.3 观测成本的构成

观测成本不仅包括存储费用，还包括：

- 应用插桩与数据导出开销。
- Collector 的处理、缓存和网络成本。
- 日志索引与追踪存储成本。
- 高基数查询和仪表盘计算成本。
- 数据保留、备份与合规审计成本。
- 团队维护、告警治理和故障排查成本。

成本优化应基于数据价值，而不是简单地删除最详细的追踪数据。对于复杂 Agent
任务，保留错误任务的完整轨迹可能比保留大量成功任务的完整内容更有价值，但具体策略应通过故障复现需求和业务风险验证。

## 13. 典型技术栈与选型依据

| 方案                                   | 适用场景                        | 优势                     | 需要评估                               |
|:---------------------------------------|:--------------------------------|:-------------------------|:---------------------------------------|
| OpenTelemetry + Prometheus + Grafana   | 自建服务与企业监控              | 标准化、灵活、可控       | 需要维护部署、存储与保留策略           |
| OpenTelemetry + Loki + Tempo + Grafana | 需要日志与追踪关联              | 跨信号排查较方便         | 标签基数、存储容量和查询成本           |
| OpenAI Agents SDK 内置追踪             | 使用 OpenAI Agents SDK 的工作流 | Agent 专用事件与集成方便 | 数据出口、访问控制、保留政策和隐私要求 |
| LangSmith                              | LangChain 生态及 Agent 调试评估 | 追踪和在线评估结合       | 平台接入、数据治理与使用成本           |
| 自定义业务审计系统                     | 医疗、金融及敏感企业应用        | 可贴合授权与审计要求     | 不能用普通运行日志替代审计系统         |

选型时应从需求出发，而不是为了工具数量而部署所有组件。优先确定必须观测的业务目标、数据敏感级别、保留要求、部署模式、现有技术栈和团队维护能力，再决定是否需要独立
Collector、专用追踪平台或在线评估服务。

## 14. 与其他 Agent 模块的关系

可观测性不是孤立模块，而是连接执行、知识、状态、评估和安全的横向基础设施。

``` mermaid
flowchart TB
    O(["可观测性"])

    O --> CORE["Agent 核心执行"]
    O --> KNOW["知识与检索"]
    O --> STATE["状态与记忆"]
    O --> TOOLS["工具与外部系统"]
    O --> EVAL["评估与反馈"]
    O --> SEC["安全与权限"]
    O --> REL["可靠性与成本"]

    CORE --> PLAN["规划、路由、模型调用"]
    KNOW --> RET["检索结果、引用与相关性"]
    STATE --> CKPT["状态变化、Checkpoint、恢复"]
    TOOLS --> CALL["调用参数、返回状态、超时"]
    EVAL --> QUALITY["质量分数、回归与人工反馈"]
    SEC --> POLICY["授权、拒绝、审计事件"]
    REL --> SLO["SLO、重试、熔断、成本指标"]

    PLAN --> O
    RET --> O
    CKPT --> O
    CALL --> O
    QUALITY --> O
    POLICY --> O
    SLO --> O

    classDef core fill:#16324F,color:#fff,stroke:#16324F,stroke-width:2px;
    classDef module fill:#DCEBFA,color:#17324D,stroke:#6A9BCB;
    classDef detail fill:#DDF4E4,color:#17452B,stroke:#61A879;

    class O core;
    class CORE,KNOW,STATE,TOOLS,EVAL,SEC,REL module;
    class PLAN,RET,CKPT,CALL,QUALITY,POLICY,SLO detail;
```

<strong>图谱解读：</strong>每个 Agent
模块都产生可观测事件；可观测性将这些事件统一关联，再把诊断结果反馈给可靠性、质量、安全和成本治理。它不是执行工作流的替代品，而是理解、验证和持续改进工作流的基础。

具体关系如下：

- **Prompt
  与模型调用**：记录模型、版本、调用参数摘要和结果状态，避免默认保存敏感提示词全文。
- **RAG
  与知识库**：关联查询、检索策略、候选结果和最终使用证据，支持分析检索退化。
- **Memory
  与状态管理**：观测状态变化、Checkpoint、恢复位置和重复执行风险。
- **Tools 与 MCP**：记录工具身份、调用状态、权限决策、超时和结果摘要。
- **Workflow 与多 Agent 协作**：通过 Trace 和 Span
  还原路由、交接、并行执行和失败恢复。
- **Evaluation**：利用追踪样本定位质量问题，并通过离线基准和线上评估验证改进。
- **可靠性与成本**：用延迟、错误率、重试率、Token
  和单位任务成本支持超时、降级、熔断与资源规划。
- **安全与权限**：将敏感操作与授权结果写入受控审计链路，并限制遥测数据暴露。

## 15. 企业级落地路线

### 阶段一：建立最小可用观测能力

1.  为每个业务任务生成稳定的任务 ID。
2.  接入模型、工具、检索和主要业务步骤的追踪。
3.  输出结构化日志，并关联 Trace ID。
4.  建立请求量、延迟、错误率和任务成功率等基础指标。
5.  明确日志、追踪和审计的敏感数据边界。

### 阶段二：完善诊断与告警

1.  建立 P95/P99 延迟、错误率和任务完成率仪表盘。
2.  配置针对模型超时、工具失败、队列积压和质量下降的告警。
3.  建立常见故障的排查手册。
4.  验证跨服务和异步任务的上下文传播。
5.  对异常任务保留足够的诊断证据。

### 阶段三：形成质量与成本闭环

1.  建立可复现的离线评估数据集。
2.  将生产追踪样本与质量评估结果关联。
3.  统计单任务 Token、工具调用量和单位合格任务成本。
4.  通过灰度发布和回归测试验证提示词、模型或工作流改动。
5.  定期审查采样、保留、访问权限和数据脱敏策略。

### 阶段四：持续治理

1.  根据服务目标调整告警和错误预算策略。
2.  定期复盘重大故障，更新错误分类和排查手册。
3.  审查遥测管线的容量、查询性能和数据成本。
4.  对审计完整性、脱敏规则和访问控制开展验证。
5.  随规范和 SDK 版本更新检查语义约定及兼容性。

## 16. 常见误区与工程难点

| 误区                       | 问题                               | 正确做法                               |
|:---------------------------|:-----------------------------------|:---------------------------------------|
| 只看模型 API 成功率        | 忽略任务级失败和质量错误           | 同时观测步骤状态、任务结果和质量指标   |
| 只采集日志，不做关联       | 很难还原跨服务调用路径             | 统一 Trace ID、任务 ID 和上下文传播    |
| 把所有 Span 时长相加       | 并行操作时间重叠会导致误判         | 分析时间线、依赖和关键路径             |
| 将高基数字段全部做标签     | 索引、内存和查询成本膨胀           | 依据后端机制设计低基数标签             |
| 默认保存全部提示词与输出   | 增大敏感数据泄露风险               | 默认最小化采集，按授权分级开启内容捕获 |
| 用普通日志替代审计         | 难以证明关键操作完整性             | 建立独立的审计策略与访问控制           |
| 把模型评审分数当作真值     | 评审模型可能存在偏差               | 用人工标注集校验评估可靠性             |
| 只建设仪表盘，没有处置流程 | 异常被发现后仍无人处理             | 为告警指定责任人、手册和验证步骤       |
| 过度采样或完全不采样       | 前者成本过高，后者可能丢失诊断信息 | 按风险、价值和保留目标设计采样策略     |
| 将相关性直接视为因果       | 可能错误归因                       | 用追踪、日志、版本和对照实验验证假设   |

## 17. 面试与设计评审中的关键问题

### 17.1 为什么 Agent 需要比传统 Web 服务更细的追踪？

因为 Agent
的行为通常具有动态性：模型可能选择不同工具、重复调用、检索多批数据、执行多轮推理或交接给不同
Agent。单看 HTTP
请求无法解释这些内部决策路径，因此需要对模型调用、工具、检索、状态变化和交接建立层级化追踪，并把运行轨迹与业务质量评估关联。

### 17.2 日志、指标和追踪如何配合定位一次慢请求？

先从指标识别异常时间窗口和影响范围，再通过 Trace 定位关键路径上的慢
Span，最后查看关联日志确认超时、重试、资源错误或外部依赖异常。修复后需要同时验证延迟、错误率和业务结果是否恢复。

### 17.3 为什么不能将全部 Trace ID 设置成 Prometheus 标签？

Trace ID
的基数几乎随请求数量持续增长，作为指标标签会产生大量时间序列，增加内存、索引和查询成本。应通过日志和追踪保存高基数关联信息，指标则使用稳定、有限的标签维度。

### 17.4 如何判断 Agent 的结果是否正确？

运行追踪只能说明执行了什么，不能独立证明结果正确。应建立与任务类型匹配的规则校验、离线基准、人工评估或模型评审，并将评估结果关联到具体运行轨迹，形成可复现的质量回归流程。

### 17.5 如何平衡调试能力与数据安全？

默认采集必要的元数据，不默认记录完整提示词、文档内容和工具返回值；对敏感字段进行过滤和脱敏；按任务和环境分级控制内容捕获；为审计事件设定独立的完整性、保留和访问控制要求。

## 18. 参考资料与权威来源

以下资料适合继续深入阅读。规范、SDK
和平台功能可能演进，实际工程使用前应核对最新版本及其兼容性。

### OpenTelemetry

- [官方文档](https://opentelemetry.io/docs/)
- [Collector 文档](https://opentelemetry.io/docs/collector/)
- [Trace
  语义约定](https://opentelemetry.io/docs/specs/semconv/general/trace/)
- [Metric
  语义约定](https://opentelemetry.io/docs/specs/semconv/general/metrics/)
- [GenAI 语义约定](https://opentelemetry.io/docs/specs/semconv/gen-ai/)

### Prometheus 与 Google SRE

- [Prometheus 概览](https://prometheus.io/docs/introduction/overview/)
- [Prometheus
  指标类型](https://prometheus.io/docs/concepts/metric_types/)
- [Prometheus
  告警规则](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- [Google SRE：Monitoring Distributed
  Systems](https://sre.google/sre-book/monitoring-distributed-systems/)

### Grafana 生态

- [Grafana Tempo](https://grafana.com/docs/tempo/latest/introduction/)
- [Grafana Loki](https://grafana.com/docs/loki/latest/)
- [Grafana Tempo
  数据源](https://grafana.com/docs/grafana/latest/datasources/tempo/)

### Agent 追踪与评估

- [OpenAI Agents SDK Python
  文档](https://openai.github.io/openai-agents-python/)
- [OpenAI Agents SDK
  追踪](https://openai.github.io/openai-agents-python/zh/tracing/)
- [OpenAI Agents SDK
  GitHub](https://github.com/openai/openai-agents-python)
- [LangSmith 官方文档](https://docs.langchain.com/langsmith/)
- [LangSmith
  在线评估](https://docs.langchain.com/langsmith/online-evaluations-llm-as-judge)

## 19. 本章总结

本章的核心知识网络可以概括为：

<strong>统一遥测标准负责规范数据，日志解释事件，指标发现异常，追踪还原路径，审计保留责任证据，评估判断业务质量，告警与事件响应推动修复，数据治理确保整个过程安全可控。</strong>

企业级 Agent 可观测性最终应形成以下闭环：

<strong>产生遥测数据 → 统一关联 → 存储与查询 → 发现异常 → 定位根因 →
评估业务质量 → 修复问题 → 验证改进。</strong>

最重要的工程原则是：

1.  日志、指标和追踪互补，不能相互替代。
2.  模型请求成功不等于任务成功，任务成功也不等于结果合格。
3.  审计与调试的目标不同，必须分别定义完整性、采样、访问与保留策略。
4.  观测能力必须与告警处置、质量评估和回归验证相结合。
5.  可观测性平台自身也需要进行性能、成本、隐私和容量治理。

与第七章其他模块的关系是：`01-Reliability-and-Fault-Tolerance.md`
关注超时、重试、熔断和恢复；`02-Performance-and-Cost.md`
关注延迟、吞吐、资源与成本；`03-Observability.md`
为它们提供可量化的运行证据，并把业务质量、安全事件和运行状态关联起来。
