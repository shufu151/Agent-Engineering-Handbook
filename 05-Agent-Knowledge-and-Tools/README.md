# Agent 知识与工具能力：中心放射式知识总览与学习导航

> <strong>本章定位：</strong>把知识获取、工具接入、任务执行与可靠交付连成一个 Agent 工程系统。本 README 是全章知识地图与学习入口，不重复七篇专题的推导和实现细节。
>
> <strong>阅读方式：</strong>每幅图以中心主题为枢纽，上方与下方各三个主分支向中心汇聚或从中心辐射；外围节点代表二级知识。少量虚线只标记关键的跨模块依赖，避免大量长距离连线穿越图面。

## 一、总览：七篇专题如何组成一个系统

Agent 不仅要“知道”，还要“能做”。知识链将外部资料转化为可核实的证据；行动链将任务意图转化为实际操作；运行时负责把两者协调为有状态、受权限约束、可验证的执行过程。

### 全章核心知识网络

```mermaid
flowchart TB
    ROOT["Agent 知识与工具能力"]
    A["RAG：知识增强"]
    A1["解析、切分、索引"]
    A2["检索、引用、生成"]
    B["Hybrid：召回融合"]
    B1["BM25 精确匹配"]
    B2["向量语义召回"]
    C["Reranking：证据精排"]
    C1["相关性重估"]
    C2["排序与上下文选择"]
    D["Function Calling：调用契约"]
    D1["Schema 与参数"]
    D2["工具结果回填"]
    E["MCP + Skills：能力组织"]
    E1["标准化外部接入"]
    E2["任务方法复用"]
    F["协同执行：交付闭环"]
    F1["路由、状态与恢复"]
    F2["权限、校验与追踪"]
    A --> ROOT
    A1 --> A
    A2 --> A
    B --> ROOT
    B1 --> B
    B2 --> B
    C --> ROOT
    C1 --> C
    C2 --> C
    ROOT --> D
    D --> D1
    D --> D2
    ROOT --> E
    E --> E1
    E --> E2
    ROOT --> F
    F --> F1
    F --> F2
    A ~~~ B
    B ~~~ C
    D ~~~ E
    E ~~~ F
    C -. "证据驱动" .-> F
    E -. "接入与指导" .-> D
    classDef core fill:#17365d,stroke:#0b2340,color:#ffffff,stroke-width:3px
    classDef blue fill:#e8f2ff,stroke:#3b82c4,color:#123d67
    classDef green fill:#e7f6ed,stroke:#389268,color:#14532d
    classDef orange fill:#fff0de,stroke:#cb883e,color:#784313
    classDef purple fill:#f1ebff,stroke:#8c71bf,color:#4c2785
    classDef pink fill:#fce8ef,stroke:#c2648a,color:#84234d
    classDef teal fill:#e1f7f5,stroke:#2d918b,color:#135b56
    class ROOT core
    class A,A1,A2 blue
    class B,B1,B2 teal
    class C,C1,C2 purple
    class D,D1,D2 green
    class E,E1,E2 orange
    class F,F1,F2 pink
```

<strong>图谱解读：</strong>六个主分支围绕同一执行目标组织。RAG、Hybrid、Reranking 是知识链的三个层次；Function Calling、MCP、Skills 分别负责调用表达、能力接入和任务方法；第七篇将这些能力纳入任务状态、权限与质量约束。虚线强调两个关键交点：证据影响决策，能力组织影响工具调用。

<strong>先建立三个边界：</strong>检索命中不等于证据充分；模型生成工具调用参数不等于获得执行授权；工具执行成功不等于用户任务完成。

## 二、知识链：如何把原始资料变成可信证据

这一条链解决模型缺少私有、最新或领域知识的问题。其因果关系是：原始资料先被正确解析与索引，召回阶段才能找到候选；候选覆盖足够后，重排才有意义；最后还要检查权限、版本和引用。

### 知识检索与证据质量

```mermaid
flowchart TB
    ROOT["从文档到可信证据"]
    KA["数据准备"]
    KA1["结构解析与 Chunk"]
    KA2["元数据、版本、ACL"]
    KB["多路召回"]
    KB1["BM25 / 关键词"]
    KB2["Dense / Embedding"]
    KC["融合与排序"]
    KC1["RRF 候选融合"]
    KC2["Cross-Encoder 精排"]
    KD["上下文构建"]
    KD1["去重与多样性"]
    KD2["Token 预算与压缩"]
    KE["可信约束"]
    KE1["权限过滤与新鲜度"]
    KE2["来源、页码与引用"]
    KF["效果评测"]
    KF1["Recall@K / nDCG"]
    KF2["证据充分与引用准确"]
    KA --> ROOT
    KA1 --> KA
    KA2 --> KA
    KB --> ROOT
    KB1 --> KB
    KB2 --> KB
    KC --> ROOT
    KC1 --> KC
    KC2 --> KC
    ROOT --> KD
    KD --> KD1
    KD --> KD2
    ROOT --> KE
    KE --> KE1
    KE --> KE2
    ROOT --> KF
    KF --> KF1
    KF --> KF2
    KA ~~~ KB
    KB ~~~ KC
    KD ~~~ KE
    KE ~~~ KF
    KA -. "权限元数据" .-> KE
    KC -. "排序结果" .-> KD
    classDef core fill:#17365d,stroke:#0b2340,color:#ffffff,stroke-width:3px
    classDef blue fill:#e8f2ff,stroke:#3b82c4,color:#123d67
    classDef green fill:#e7f6ed,stroke:#389268,color:#14532d
    classDef orange fill:#fff0de,stroke:#cb883e,color:#784313
    classDef purple fill:#f1ebff,stroke:#8c71bf,color:#4c2785
    classDef pink fill:#fce8ef,stroke:#c2648a,color:#84234d
    classDef teal fill:#e1f7f5,stroke:#2d918b,color:#135b56
    class ROOT core
    class KA,KA1,KA2 blue
    class KB,KB1,KB2 teal
    class KC,KC1,KC2 purple
    class KD,KD1,KD2 green
    class KE,KE1,KE2 orange
    class KF,KF1,KF2 pink
```

<strong>图谱解读：</strong>召回关注“不要漏”，重排关注“排得准”，上下文构建关注“装得下且有依据”，治理关注“有权限且版本正确”。例如设备型号、错误码和标准编号常需要关键词精确召回；自然语言描述则可由向量检索补充。重排不能凭空恢复未被召回的文档。

| 主题 | 解决的关键问题 | 深入阅读 |
|---|---|---|
| RAG | 知识如何摄取、检索并用于生成 | [01-RAG.md](./01-RAG.md) |
| Hybrid Search | 精确匹配与语义召回如何互补 | [02-Hybrid-Search.md](./02-Hybrid-Search.md) |
| Reranking | 如何从候选中选出最可信、最有用的证据 | [03-Reranking.md](./03-Reranking.md) |

## 三、行动链：模型如何从“提出动作”走向“完成操作”

工具体系有三个容易混淆的层次：<strong>Function Calling</strong> 是模型输出调用意图与结构化参数的机制；<strong>MCP（Model Context Protocol，模型上下文协议）</strong> 标准化外部工具、资源和提示模板的发现与访问；<strong>Agent Skills</strong> 则封装完成某类任务所需的指导、脚本和参考资料。三者不是互相替代的产品。

### 工具接入与能力复用

```mermaid
flowchart TB
    ROOT["Agent 外部行动能力"]
    TA["调用表达"]
    TA1["工具名称与 JSON Schema"]
    TA2["参数校验与结果回填"]
    TB["能力发现"]
    TB1["工具目录与选择"]
    TB2["候选过滤与路由"]
    TC["标准连接"]
    TC1["MCP Host / Client / Server"]
    TC2["Tools / Resources / Prompts"]
    TD["任务方法"]
    TD1["SKILL.md 与描述"]
    TD2["渐进式加载与脚本"]
    TE["执行边界"]
    TE1["认证、授权与审批"]
    TE2["超时、重试、幂等"]
    TF["执行反馈"]
    TF1["工具结果与错误"]
    TF2["任务状态与可追踪性"]
    TA --> ROOT
    TA1 --> TA
    TA2 --> TA
    TB --> ROOT
    TB1 --> TB
    TB2 --> TB
    TC --> ROOT
    TC1 --> TC
    TC2 --> TC
    ROOT --> TD
    TD --> TD1
    TD --> TD2
    ROOT --> TE
    TE --> TE1
    TE --> TE2
    ROOT --> TF
    TF --> TF1
    TF --> TF2
    TA ~~~ TB
    TB ~~~ TC
    TD ~~~ TE
    TE ~~~ TF
    TD -. "指导选择" .-> TB
    TC -. "提供能力" .-> TA
    classDef core fill:#17365d,stroke:#0b2340,color:#ffffff,stroke-width:3px
    classDef blue fill:#e8f2ff,stroke:#3b82c4,color:#123d67
    classDef green fill:#e7f6ed,stroke:#389268,color:#14532d
    classDef orange fill:#fff0de,stroke:#cb883e,color:#784313
    classDef purple fill:#f1ebff,stroke:#8c71bf,color:#4c2785
    classDef pink fill:#fce8ef,stroke:#c2648a,color:#84234d
    classDef teal fill:#e1f7f5,stroke:#2d918b,color:#135b56
    class ROOT core
    class TA,TA1,TA2 green
    class TB,TB1,TB2 teal
    class TC,TC1,TC2 blue
    class TD,TD1,TD2 purple
    class TE,TE1,TE2 orange
    class TF,TF1,TF2 pink
```

<strong>图谱解读：</strong>Skill 指导“怎么完成任务”，MCP 解决“如何连接能力”，Function Calling 解决“模型怎样提出调用”。真正执行、鉴权、重试和记录状态的是应用运行时与服务端。尤其不能把 Skill 中的文字指令视为自动获得的工具权限。

| 主题 | 关键理解 | 深入阅读 |
|---|---|---|
| Function Calling | Tool Schema、选择、调用循环、并发与错误处理 | [04-Function-Calling.md](./04-Function-Calling.md) |
| MCP | Host/Client/Server、传输、资源、授权与协议版本 | [05-MCP.md](./05-MCP.md) |
| Agent Skills | SKILL.md、发现、激活、渐进式加载与治理 | [06-Agent-Skills.md](./06-Agent-Skills.md) |

## 四、协同决策：什么时候查知识，什么时候调用工具？

固定“先 RAG 后 Tool”不是通用解法。制度解释以检索为主；实时设备状态以 API 查询为主；按制度提交审批则需要知识与实时事实联合判断，并在写操作前验证权限。正确策略取决于<strong>目标、数据时效性、依赖关系与操作风险</strong>。

### 知识与工具的联合决策

```mermaid
flowchart TB
    ROOT["任务驱动的能力路由"]
    RA["任务识别"]
    RA1["问答 / 查询 / 写操作"]
    RA2["时效性与风险等级"]
    RB["检索优先"]
    RB1["规则、制度与依据"]
    RB2["版本与证据检查"]
    RC["工具优先"]
    RC1["实时状态与精确数据"]
    RC2["返回事实再决定检索"]
    RD["并行执行"]
    RD1["独立知识与事实分支"]
    RD2["等待依赖汇合"]
    RE["迭代执行"]
    RE1["观察—判断—行动"]
    RE2["有界循环与停止条件"]
    RF["结果裁决"]
    RF1["证据是否充分"]
    RF2["操作是否获授权"]
    RA --> ROOT
    RA1 --> RA
    RA2 --> RA
    RB --> ROOT
    RB1 --> RB
    RB2 --> RB
    RC --> ROOT
    RC1 --> RC
    RC2 --> RC
    ROOT --> RD
    RD --> RD1
    RD --> RD2
    ROOT --> RE
    RE --> RE1
    RE --> RE2
    ROOT --> RF
    RF --> RF1
    RF --> RF2
    RA ~~~ RB
    RB ~~~ RC
    RD ~~~ RE
    RE ~~~ RF
    RA -. "风险约束" .-> RF
    RB -. "证据核实" .-> RF
    classDef core fill:#17365d,stroke:#0b2340,color:#ffffff,stroke-width:3px
    classDef blue fill:#e8f2ff,stroke:#3b82c4,color:#123d67
    classDef green fill:#e7f6ed,stroke:#389268,color:#14532d
    classDef orange fill:#fff0de,stroke:#cb883e,color:#784313
    classDef purple fill:#f1ebff,stroke:#8c71bf,color:#4c2785
    classDef pink fill:#fce8ef,stroke:#c2648a,color:#84234d
    classDef teal fill:#e1f7f5,stroke:#2d918b,color:#135b56
    class ROOT core
    class RA,RA1,RA2 blue
    class RB,RB1,RB2 teal
    class RC,RC1,RC2 green
    class RD,RD1,RD2 purple
    class RE,RE1,RE2 orange
    class RF,RF1,RF2 pink
```

<strong>图谱解读：</strong>从中心向外看，是不同任务条件下的策略选择；从外围向中心看，是证据与操作反馈共同影响下一次决策。并行仅适用于没有前后依赖的分支；迭代必须有最大轮次、超时和成本预算。

<strong>工程取舍：</strong>稳定、可审计的业务流程优先使用确定性 Workflow；探索性问题可在受限范围内使用模型动态路由；高风险写操作必须由服务端策略独立授权。

## 五、运行时：把一次调用升级为可恢复的业务任务

复杂 Agent 需要管理任务状态、依赖、证据、文件产物和副作用。聊天历史不能代替任务数据库，Trace 也不能代替授权与事务保证。

### 状态、可靠性与可观测性

```mermaid
flowchart TB
    ROOT["可靠 Agent Runtime"]
    SA["任务规划"]
    SA1["DAG 依赖与并行"]
    SA2["条件分支与停止条件"]
    SB["状态管理"]
    SB1["Checkpoint / Resume"]
    SB2["任务与会话状态分离"]
    SC["执行可靠性"]
    SC1["超时、退避、熔断"]
    SC2["幂等键与副作用去重"]
    SD["证据管理"]
    SD1["来源、版本、引用"]
    SD2["检索与工具结果关联"]
    SE["权限安全"]
    SE1["ACL / 租户隔离"]
    SE2["Prompt Injection 防护"]
    SF["质量追踪"]
    SF1["Trace、指标、审计"]
    SF2["验证、审批、交付门禁"]
    SA --> ROOT
    SA1 --> SA
    SA2 --> SA
    SB --> ROOT
    SB1 --> SB
    SB2 --> SB
    SC --> ROOT
    SC1 --> SC
    SC2 --> SC
    ROOT --> SD
    SD --> SD1
    SD --> SD2
    ROOT --> SE
    SE --> SE1
    SE --> SE2
    ROOT --> SF
    SF --> SF1
    SF --> SF2
    SA ~~~ SB
    SB ~~~ SC
    SD ~~~ SE
    SE ~~~ SF
    SC -. "失败恢复" .-> SB
    SE -. "来源隔离" .-> SD
    classDef core fill:#17365d,stroke:#0b2340,color:#ffffff,stroke-width:3px
    classDef blue fill:#e8f2ff,stroke:#3b82c4,color:#123d67
    classDef green fill:#e7f6ed,stroke:#389268,color:#14532d
    classDef orange fill:#fff0de,stroke:#cb883e,color:#784313
    classDef purple fill:#f1ebff,stroke:#8c71bf,color:#4c2785
    classDef pink fill:#fce8ef,stroke:#c2648a,color:#84234d
    classDef teal fill:#e1f7f5,stroke:#2d918b,color:#135b56
    class ROOT core
    class SA,SA1,SA2 blue
    class SB,SB1,SB2 teal
    class SC,SC1,SC2 green
    class SD,SD1,SD2 purple
    class SE,SE1,SE2 orange
    class SF,SF1,SF2 pink
```

<strong>图谱解读：</strong>任务规划决定先后依赖，Checkpoint 保存可恢复状态，幂等性处理“外部操作已成功但本地未记录”的重复执行风险。证据治理和权限控制同时覆盖检索与工具。最终是否交付由业务校验结果决定，而不是由模型自行宣称成功。

| 失败情形 | 不推荐做法 | 更稳妥的处理 |
|---|---|---|
| 检索无结果 | 编造答案 | 查询改写、补检或明确证据不足 |
| 工具超时 | 无限重试 | 限次退避、状态核对 |
| 提交结果未知 | 直接重复提交 | 幂等键、查询外部状态 |
| 权限不足 | 换工具绕过 | 停止并请求授权 |
| 译文校验失败 | 全量重新执行 | 定位失败步骤并局部修复 |

## 六、贯穿案例：企业医疗设备文档翻译 Agent

同一个任务可以检验七篇知识是否真正连成系统。以“将英文设备说明书翻译成中文，保留版式并检查型号、单位”为例：先识别文件与任务约束；从受控术语库取得依据；通过工具解析、翻译、排版；再以规则与视觉校验交付文件。

### 企业文档翻译能力协同

```mermaid
flowchart TB
    ROOT["可交付的文档翻译任务"]
    PA["任务理解"]
    PA1["文件类型与目标语言"]
    PA2["版式、术语、合规约束"]
    PB["知识检索"]
    PB1["术语表与产品型号"]
    PB2["质量标准与版本"]
    PC["文件操作"]
    PC1["PDF / Word / 图片解析"]
    PC2["翻译、排版与导出"]
    PD["技能复用"]
    PD1["翻译 Skill 指导"]
    PD2["校验 Skill 与脚本"]
    PE["执行管理"]
    PE1["分支并行与状态保存"]
    PE2["失败重试与人工确认"]
    PF["质量交付"]
    PF1["数字、单位、型号校验"]
    PF2["视觉回归与质量报告"]
    PA --> ROOT
    PA1 --> PA
    PA2 --> PA
    PB --> ROOT
    PB1 --> PB
    PB2 --> PB
    PC --> ROOT
    PC1 --> PC
    PC2 --> PC
    ROOT --> PD
    PD --> PD1
    PD --> PD2
    ROOT --> PE
    PE --> PE1
    PE --> PE2
    ROOT --> PF
    PF --> PF1
    PF --> PF2
    PA ~~~ PB
    PB ~~~ PC
    PD ~~~ PE
    PE ~~~ PF
    PB -. "校验依据" .-> PF
    PD -. "执行指导" .-> PC
    classDef core fill:#17365d,stroke:#0b2340,color:#ffffff,stroke-width:3px
    classDef blue fill:#e8f2ff,stroke:#3b82c4,color:#123d67
    classDef green fill:#e7f6ed,stroke:#389268,color:#14532d
    classDef orange fill:#fff0de,stroke:#cb883e,color:#784313
    classDef purple fill:#f1ebff,stroke:#8c71bf,color:#4c2785
    classDef pink fill:#fce8ef,stroke:#c2648a,color:#84234d
    classDef teal fill:#e1f7f5,stroke:#2d918b,color:#135b56
    class ROOT core
    class PA,PA1,PA2 blue
    class PB,PB1,PB2 teal
    class PC,PC1,PC2 green
    class PD,PD1,PD2 purple
    class PE,PE1,PE2 orange
    class PF,PF1,PF2 pink
```

<strong>图谱解读：</strong>术语库提供“翻译依据”，工具提供“文件处理动作”，Skills 提供“可复用的操作方法”，Runtime 提供“执行与恢复能力”，质量检查决定“能否交付”。对于医疗设备型号、数值与单位，优先使用确定性比对；版式质量可结合规则和视觉评估。任何自动修复都应有边界与复核机制。

## 七、如何选型：不同任务需要不同的组合

| 场景 | 核心能力组合 | 关键原因 |
|---|---|---|
| 企业制度问答 | RAG + Hybrid + Reranking | 依据、覆盖、排序与引用 |
| 设备实时状态查询 | Function Calling + 业务 API / MCP | 需要最新的结构化事实 |
| 按制度办理审批 | RAG + API + Workflow + 审批 | 规则依据与副作用同时存在 |
| 技术文档翻译 | Skills + 检索 + 文档工具 + 校验 | 多步骤方法复用与产物质量 |
| 跨系统多步骤任务 | MCP + Runtime + 状态恢复 | 接入、依赖、错误和权限治理 |

<strong>选型原则：</strong>先定义业务正确性和权限边界，再选择检索与工具；优先把确定性操作交给代码或规则；只有确实需要动态判断时才引入自主规划。

## 八、学习导航：七篇专题与推荐顺序

建议按“知识准备 → 召回优化 → 证据排序 → 调用表达 → 标准接入 → 能力复用 → 协同工程”的顺序阅读。

### 学习路线与知识依赖

```mermaid
flowchart TB
    ROOT["七篇专题学习地图"]
    LA["01 RAG"]
    LA1["摄取、索引与检索"]
    LA2["证据与引用"]
    LB["02 Hybrid Search"]
    LB1["关键词与向量"]
    LB2["融合与召回"]
    LC["03 Reranking"]
    LC1["精排模型"]
    LC2["候选质量"]
    LD["04 Function Calling"]
    LD1["调用协议与 Schema"]
    LD2["执行循环"]
    LE["05 MCP + 06 Skills"]
    LE1["连接外部系统"]
    LE2["复用任务方法"]
    LF["07 协同整合"]
    LF1["路由、编排、恢复"]
    LF2["治理、评测、交付"]
    LA --> ROOT
    LA1 --> LA
    LA2 --> LA
    LB --> ROOT
    LB1 --> LB
    LB2 --> LB
    LC --> ROOT
    LC1 --> LC
    LC2 --> LC
    ROOT --> LD
    LD --> LD1
    LD --> LD2
    ROOT --> LE
    LE --> LE1
    LE --> LE2
    ROOT --> LF
    LF --> LF1
    LF --> LF2
    LA ~~~ LB
    LB ~~~ LC
    LD ~~~ LE
    LE ~~~ LF
    LC -. "知识链汇总" .-> LF
    LE -. "行动链汇总" .-> LF
    classDef core fill:#17365d,stroke:#0b2340,color:#ffffff,stroke-width:3px
    classDef blue fill:#e8f2ff,stroke:#3b82c4,color:#123d67
    classDef green fill:#e7f6ed,stroke:#389268,color:#14532d
    classDef orange fill:#fff0de,stroke:#cb883e,color:#784313
    classDef purple fill:#f1ebff,stroke:#8c71bf,color:#4c2785
    classDef pink fill:#fce8ef,stroke:#c2648a,color:#84234d
    classDef teal fill:#e1f7f5,stroke:#2d918b,color:#135b56
    class ROOT core
    class LA,LA1,LA2 blue
    class LB,LB1,LB2 teal
    class LC,LC1,LC2 purple
    class LD,LD1,LD2 green
    class LE,LE1,LE2 orange
    class LF,LF1,LF2 pink
```

<strong>图谱解读：</strong>前三篇建立知识获取与质量优化基础；第四至六篇建立工具执行、外部接入与可复用方法；第七篇将前六篇整合为具有状态、安全与评测的完整 Agent 系统。

1. [01｜RAG：检索增强生成与完整链路](./01-RAG.md) — 理解知识如何进入模型上下文。
2. [02｜Hybrid Search：向量、关键词与混合检索](./02-Hybrid-Search.md) — 理解召回互补与融合策略。
3. [03｜Reranking：重排序与检索质量优化](./03-Reranking.md) — 理解候选精排和证据质量。
4. [04｜Function Calling：函数调用协议与执行流程](./04-Function-Calling.md) — 理解工具契约、参数与调用循环。
5. [05｜MCP：Model Context Protocol 的组成与边界](./05-MCP.md) — 理解跨系统能力发现与标准接入。
6. [06｜Agent Skills：可复用能力封装](./06-Agent-Skills.md) — 理解能力发现、渐进式加载与任务方法复用。
7. [07｜Knowledge and Tool Integration：知识检索与工具调用协同](./07-Knowledge-and-Tool-Integration.md) — 理解完整运行时、可靠性与工程取舍。

## 九、读完本章应形成的工程认知

<strong>第一，知识与行动分离但相互依赖。</strong>检索提供依据，工具提供真实操作；实时事实通常来自业务系统，不能把所有信息都当作静态文档检索。

<strong>第二，协议、方法与执行器各有边界。</strong>Function Calling 表达调用，MCP 标准化接入，Skills 组织任务知识，Runtime 负责调度、权限、状态与恢复。

<strong>第三，正确性必须端到端验证。</strong>检索质量、工具调用成功率和最终任务成功率是不同指标。企业级系统必须同时评测证据质量、权限安全、工具副作用、交付产物与失败恢复。

<strong>全章总结：</strong>以可信知识支持判断，以受控工具完成行动，以可复用 Skills 降低重复开发，以可靠 Runtime 保障状态和安全，最终形成<strong>可验证、可恢复、可追溯</strong>的 Agent 工程闭环。

