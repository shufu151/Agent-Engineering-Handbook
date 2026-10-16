# 06｜Agent Skills：可复用 Agent 能力封装、渐进式加载与企业级治理

> 章节定位：第五章 Agent 知识与工具体系。本文以 Agent Skills 开放规范为核心，区分规范约定、平台特性与工程建议；不同宿主和 MCP 扩展的具体字段与支持范围应以对应版本官方文档为准。

## 阅读路线

问题与定位 → 规范与元数据 → 渐进式加载 → 发现和触发 → 指令与资源 → 粒度与组合 → 工具及 MCP 协作 → 安全与治理 → 评测 → 企业案例。每节以原理为主，以多分支知识网络呈现关系。


## 1. 问题起点：从临时提示词到可复用能力

Agent Skills 不是新模型，而是将任务方法、约束、脚本和资源封装为可发现、可复用的能力包。传统长 System Prompt 在任务增加时出现上下文膨胀、职责混杂、难以版本控制的问题；将所有步骤硬编码又降低对开放任务的适应性。Skill 在两者之间提供可维护的任务知识层。


### 1.1 核心机制与设计取舍


### 1.2 多层级知识网络

```mermaid
flowchart LR
  CORE1["问题起点：从临时提示词到可复用能力"]
  B1_0["需求背景"]
  CORE1 --- B1_0
  B1_0 --> N1_0_0["长提示词膨胀"]
  B1_0 --> N1_0_1["重复规则"]
  B1_1["模型工具解耦"]
  CORE1 --- B1_1
  B1_1 --> N1_1_0["封装价值"]
  B1_1 --> N1_1_1["复用"]
  B1_2["按需加载"]
  CORE1 --- B1_2
  B1_2 --> N1_2_0["可审计"]
  B1_2 --> N1_2_1["边界"]
  B1_3["不是 Agent"]
  CORE1 --- B1_3
  B1_3 --> N1_3_0["不是权限系统"]
  B1_3 --> N1_3_1["不是工具本身"]
  B1_0 -.-> B1_1
  B1_1 -.-> B1_2
  B1_2 -.-> B1_3
  B1_3 -.-> B1_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE1 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B1_0,N1_0_0,N1_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B1_1,N1_1_0,N1_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B1_2,N1_2_0,N1_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B1_3,N1_3_0,N1_3_1 module3;
```


<strong>图谱解读：</strong>以“问题起点：从临时提示词到可复用能力”为中心，分别连接“需求背景”“模型工具解耦”“按需加载”“不是 Agent”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 1.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“长提示词膨胀”与“封装价值”建立可复现的验证用例。对于“不是权限系统”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 2. 开放规范与目录契约

Agent Skills 开放规范以目录为交付单元，至少需要带 YAML frontmatter 的 SKILL.md。name 和 description 是基础元数据；scripts、references、assets 是按需组织的惯例目录，并非所有 Skill 必须包含。平台可能添加自身字段或限制，跨平台移植时应区分开放规范与宿主扩展。


### 2.1 核心机制与设计取舍


典型目录如下，`SKILL.md` 是入口，其他文件夹按需提供：

```text
document-translation/
├── SKILL.md
├── scripts/
│   └── check_numbers.py
├── references/
│   └── terminology.md
└── assets/
    └── report-template.docx
```

`SKILL.md` 顶部使用 YAML frontmatter。下面是示例，不是所有宿主都支持相同扩展字段：

```yaml
---
name: document-translation
description: Translate technical documents while preserving layout and checking terminology, numbers and model identifiers. Use for document translation tasks; not for general conversation.
---
```


### 2.2 多层级知识网络

```mermaid
flowchart LR
  CORE2["开放规范与目录契约"]
  B2_0["规范契约"]
  CORE2 --- B2_0
  B2_0 --> N2_0_0["SKILL.md"]
  B2_0 --> N2_0_1["YAML 元数据"]
  B2_1["目录结构"]
  CORE2 --- B2_1
  B2_1 --> N2_1_0["内容组件"]
  B2_1 --> N2_1_1["scripts"]
  B2_2["references"]
  CORE2 --- B2_2
  B2_2 --> N2_2_0["assets"]
  B2_2 --> N2_2_1["可移植性"]
  B2_3["宿主差异"]
  CORE2 --- B2_3
  B2_3 --> N2_3_0["路径约束"]
  B2_3 --> N2_3_1["版本检查"]
  B2_0 -.-> B2_1
  B2_1 -.-> B2_2
  B2_2 -.-> B2_3
  B2_3 -.-> B2_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE2 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B2_0,N2_0_0,N2_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B2_1,N2_1_0,N2_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B2_2,N2_2_0,N2_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B2_3,N2_3_0,N2_3_1 module3;
```


<strong>图谱解读：</strong>以“开放规范与目录契约”为中心，分别连接“规范契约”“目录结构”“references”“宿主差异”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 2.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“SKILL.md”与“内容组件”建立可复现的验证用例。对于“路径约束”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 3. 元数据设计与语义边界

name 是稳定标识，description 应说明做什么、何时使用及何时不使用。好的描述要兼顾召回与精度：太宽泛容易误触发，太狭窄则漏触发。对相近 Skill 需要划定任务目标和优先级，不应依赖模型猜测隐含边界。


### 3.1 核心机制与设计取舍


description 应写清触发正例与反例；如果两个 Skill 都能处理 PDF，应进一步区分是“翻译整份文档”还是“修复版式”。否则两个能力可能竞争相同请求。


### 3.2 多层级知识网络

```mermaid
flowchart LR
  CORE3["元数据设计与语义边界"]
  B3_0["元数据"]
  CORE3 --- B3_0
  B3_0 --> N3_0_0["name"]
  B3_0 --> N3_0_1["description"]
  B3_1["compatibility"]
  CORE3 --- B3_1
  B3_1 --> N3_1_0["选择质量"]
  B3_1 --> N3_1_1["正向条件"]
  B3_2["排除条件"]
  CORE3 --- B3_2
  B3_2 --> N3_2_0["示例任务"]
  B3_2 --> N3_2_1["治理"]
  B3_3["命名冲突"]
  CORE3 --- B3_3
  B3_3 --> N3_3_0["版本标识"]
  B3_3 --> N3_3_1["审查"]
  B3_0 -.-> B3_1
  B3_1 -.-> B3_2
  B3_2 -.-> B3_3
  B3_3 -.-> B3_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE3 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B3_0,N3_0_0,N3_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B3_1,N3_1_0,N3_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B3_2,N3_2_0,N3_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B3_3,N3_3_0,N3_3_1 module3;
```


<strong>图谱解读：</strong>以“元数据设计与语义边界”为中心，分别连接“元数据”“compatibility”“排除条件”“命名冲突”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 3.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“name”与“选择质量”建立可复现的验证用例。对于“版本标识”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 4. 渐进式披露与上下文经济

Progressive Disclosure（渐进式披露）把加载拆为能力目录摘要、SKILL.md 具体指导、参考文件与脚本等按需资源。节省上下文不是唯一目标：它也降低不相关规则干扰，并使知识维护更模块化。实际读取文件的时机取决于宿主实现。


### 4.1 核心机制与设计取舍


| 加载阶段 | 读取内容 | 为什么这样设计 |
|---|---|---|
| 发现 | 名称和描述 | 以较小上下文开销确定候选 |
| 激活 | `SKILL.md` 主体 | 获得任务决策规则 |
| 执行 | 相关参考文件、脚本、资产 | 仅引入本次任务需要的内容 |

<strong>渐进式披露不是懒加载的绝对保证</strong>：不同宿主可能预加载、缓存或主动读取不同资源。应以具体运行时行为评测 Token 成本和选择效果。


### 4.2 多层级知识网络

```mermaid
flowchart LR
  CORE4["渐进式披露与上下文经济"]
  B4_0["加载层级"]
  CORE4 --- B4_0
  B4_0 --> N4_0_0["元数据发现"]
  B4_0 --> N4_0_1["指令激活"]
  B4_1["资源按需"]
  CORE4 --- B4_1
  B4_1 --> N4_1_0["成本收益"]
  B4_1 --> N4_1_1["Token 节省"]
  B4_2["干扰减少"]
  CORE4 --- B4_2
  B4_2 --> N4_2_0["维护隔离"]
  B4_2 --> N4_2_1["限制"]
  B4_3["检索误差"]
  CORE4 --- B4_3
  B4_3 --> N4_3_0["缓存失效"]
  B4_3 --> N4_3_1["引用缺失"]
  B4_0 -.-> B4_1
  B4_1 -.-> B4_2
  B4_2 -.-> B4_3
  B4_3 -.-> B4_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE4 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B4_0,N4_0_0,N4_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B4_1,N4_1_0,N4_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B4_2,N4_2_0,N4_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B4_3,N4_3_0,N4_3_1 module3;
```


<strong>图谱解读：</strong>以“渐进式披露与上下文经济”为中心，分别连接“加载层级”“资源按需”“干扰减少”“检索误差”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 4.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“元数据发现”与“成本收益”建立可复现的验证用例。对于“缓存失效”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 5. 发现、匹配与激活

Skill 的触发既可能由用户显式指定，也可能由模型根据任务语义隐式选择。核心质量维度是 Trigger Precision 与 Trigger Recall；存在多候选时可结合分类路由、任务条件、负面条件和显式优先级。选择机制是启发式或模型决策，不等于安全授权。


### 5.1 核心机制与设计取舍


隐式激活是模型选择问题，显式激活是用户或流程指定问题。两者都需要执行端权限检查；描述字段中的“允许访问”并不构成授权。


### 5.2 多层级知识网络

```mermaid
flowchart LR
  CORE5["发现、匹配与激活"]
  B5_0["发现路径"]
  CORE5 --- B5_0
  B5_0 --> N5_0_0["项目目录"]
  B5_0 --> N5_0_1["用户目录"]
  B5_1["远程目录"]
  CORE5 --- B5_1
  B5_1 --> N5_1_0["匹配方式"]
  B5_1 --> N5_1_1["显式指定"]
  B5_2["隐式匹配"]
  CORE5 --- B5_2
  B5_2 --> N5_2_0["候选排序"]
  B5_2 --> N5_2_1["风险"]
  B5_3["误触发"]
  CORE5 --- B5_3
  B5_3 --> N5_3_0["漏触发"]
  B5_3 --> N5_3_1["描述冲突"]
  B5_0 -.-> B5_1
  B5_1 -.-> B5_2
  B5_2 -.-> B5_3
  B5_3 -.-> B5_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE5 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B5_0,N5_0_0,N5_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B5_1,N5_1_0,N5_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B5_2,N5_2_0,N5_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B5_3,N5_3_0,N5_3_1 module3;
```


<strong>图谱解读：</strong>以“发现、匹配与激活”为中心，分别连接“发现路径”“远程目录”“隐式匹配”“误触发”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 5.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“项目目录”与“匹配方式”建立可复现的验证用例。对于“漏触发”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 6. 指令层：任务策略与停止条件

SKILL.md 应描述输入要求、决策分支、工具使用条件、失败处理和完成定义。与其列出所有可能命令，不如明确不可省略的检查、可选步骤和人类确认边界。对低风险开放任务保留灵活度，对高风险操作增加确定性约束。


### 6.1 核心机制与设计取舍


### 6.2 多层级知识网络

```mermaid
flowchart LR
  CORE6["指令层：任务策略与停止条件"]
  B6_0["任务策略"]
  CORE6 --- B6_0
  B6_0 --> N6_0_0["目标与输入"]
  B6_0 --> N6_0_1["分支条件"]
  B6_1["交付标准"]
  CORE6 --- B6_1
  B6_1 --> N6_1_0["执行约束"]
  B6_1 --> N6_1_1["必须步骤"]
  B6_2["可选步骤"]
  CORE6 --- B6_2
  B6_2 --> N6_2_0["终止条件"]
  B6_2 --> N6_2_1["异常管理"]
  B6_3["澄清缺参"]
  CORE6 --- B6_3
  B6_3 --> N6_3_0["失败升级"]
  B6_3 --> N6_3_1["人工确认"]
  B6_0 -.-> B6_1
  B6_1 -.-> B6_2
  B6_2 -.-> B6_3
  B6_3 -.-> B6_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE6 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B6_0,N6_0_0,N6_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B6_1,N6_1_0,N6_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B6_2,N6_2_0,N6_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B6_3,N6_3_0,N6_3_1 module3;
```


<strong>图谱解读：</strong>以“指令层：任务策略与停止条件”为中心，分别连接“任务策略”“交付标准”“可选步骤”“澄清缺参”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 6.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“目标与输入”与“执行约束”建立可复现的验证用例。对于“失败升级”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 7. 脚本、参考资料与资产协同

脚本适合数字比对、文件结构检查等确定性操作；参考资料适合术语、政策、长规则；assets 适合模板和静态交付资源。脚本是否执行由运行环境决定，Skill 文本不能自行获得系统权限。对于脚本输出，模型只需要读取结果和必要解释，而非全部源代码。


### 7.1 核心机制与设计取舍


### 7.2 多层级知识网络

```mermaid
flowchart LR
  CORE7["脚本、参考资料与资产协同"]
  B7_0["资源类型"]
  CORE7 --- B7_0
  B7_0 --> N7_0_0["scripts"]
  B7_0 --> N7_0_1["references"]
  B7_1["assets"]
  CORE7 --- B7_1
  B7_1 --> N7_1_0["执行接口"]
  B7_1 --> N7_1_1["工具调用"]
  B7_2["运行环境"]
  CORE7 --- B7_2
  B7_2 --> N7_2_0["结果摘要"]
  B7_2 --> N7_2_1["工程约束"]
  B7_3["依赖锁定"]
  CORE7 --- B7_3
  B7_3 --> N7_3_0["路径安全"]
  B7_3 --> N7_3_1["产物校验"]
  B7_0 -.-> B7_1
  B7_1 -.-> B7_2
  B7_2 -.-> B7_3
  B7_3 -.-> B7_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE7 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B7_0,N7_0_0,N7_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B7_1,N7_1_0,N7_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B7_2,N7_2_0,N7_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B7_3,N7_3_0,N7_3_1 module3;
```


<strong>图谱解读：</strong>以“脚本、参考资料与资产协同”为中心，分别连接“资源类型”“assets”“运行环境”“依赖锁定”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 7.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“scripts”与“执行接口”建立可复现的验证用例。对于“路径安全”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 8. Skill 粒度、组合与依赖

以可独立交付的用户目标为主要拆分标准。过粗会形成万能 Skill，过细会带来路由与协调开销。复合任务可由主 Skill 指导多个工具，必要时组合专用 Skill；但循环依赖、隐式优先级和相互矛盾的指令必须显式处理。


### 8.1 核心机制与设计取舍


组合设计需要检查共享资源的版本一致性。两个 Skill 若分别要求不同的术语表版本，Runtime 应根据任务配置选择一个权威版本，或要求用户澄清。


### 8.2 多层级知识网络

```mermaid
flowchart LR
  CORE8["Skill 粒度、组合与依赖"]
  B8_0["拆分维度"]
  CORE8 --- B8_0
  B8_0 --> N8_0_0["任务目标"]
  B8_0 --> N8_0_1["复用边界"]
  B8_1["变更频率"]
  CORE8 --- B8_1
  B8_1 --> N8_1_0["组合模式"]
  B8_1 --> N8_1_1["主从调用"]
  B8_2["并列协作"]
  CORE8 --- B8_2
  B8_2 --> N8_2_0["共享资源"]
  B8_2 --> N8_2_1["冲突控制"]
  B8_3["依赖图"]
  CORE8 --- B8_3
  B8_3 --> N8_3_0["优先级"]
  B8_3 --> N8_3_1["循环检测"]
  B8_0 -.-> B8_1
  B8_1 -.-> B8_2
  B8_2 -.-> B8_3
  B8_3 -.-> B8_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE8 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B8_0,N8_0_0,N8_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B8_1,N8_1_0,N8_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B8_2,N8_2_0,N8_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B8_3,N8_3_0,N8_3_1 module3;
```


<strong>图谱解读：</strong>以“Skill 粒度、组合与依赖”为中心，分别连接“拆分维度”“变更频率”“并列协作”“依赖图”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 8.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“任务目标”与“组合模式”建立可复现的验证用例。对于“优先级”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 9. 与 Prompt、Tool、Workflow 的分层

Prompt 是单次或持久指令；Tool 提供可执行操作；Function Calling 表达模型调用意图；Workflow 强制组织步骤；Skill 封装可复用任务知识。复杂企业系统通常由 Workflow 固化必须遵守的审批与校验，由 Skill 提供专业策略，再由工具完成实际操作。


### 9.1 核心机制与设计取舍


固定工作流适合不可跳过的合规步骤，Skill 适合可复用但需要灵活判断的任务方法。将审批、支付、删除等操作只写成 Skill 文字说明，不足以强制执行。


### 9.2 多层级知识网络

```mermaid
flowchart LR
  CORE9["与 Prompt、Tool、Workflow 的分层"]
  B9_0["概念层"]
  CORE9 --- B9_0
  B9_0 --> N9_0_0["Prompt 指令"]
  B9_0 --> N9_0_1["Tool 能力"]
  B9_1["Skill 方法"]
  CORE9 --- B9_1
  B9_1 --> N9_1_0["执行层"]
  B9_1 --> N9_1_1["Function Calling"]
  B9_2["Workflow"]
  CORE9 --- B9_2
  B9_2 --> N9_2_0["Runtime"]
  B9_2 --> N9_2_1["协作原则"]
  B9_3["确定性优先"]
  CORE9 --- B9_3
  B9_3 --> N9_3_0["策略与执行分离"]
  B9_3 --> N9_3_1["安全独立"]
  B9_0 -.-> B9_1
  B9_1 -.-> B9_2
  B9_2 -.-> B9_3
  B9_3 -.-> B9_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE9 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B9_0,N9_0_0,N9_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B9_1,N9_1_0,N9_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B9_2,N9_2_0,N9_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B9_3,N9_3_0,N9_3_1 module3;
```


<strong>图谱解读：</strong>以“与 Prompt、Tool、Workflow 的分层”为中心，分别连接“概念层”“Skill 方法”“Workflow”“确定性优先”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 9.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“Prompt 指令”与“执行层”建立可复现的验证用例。对于“策略与执行分离”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 10. 与 MCP、RAG、Memory 的协作

MCP（Model Context Protocol）用于标准化外部能力接入；RAG（Retrieval-Augmented Generation，检索增强生成）负责检索证据；Memory 负责保存动态状态或经验。Skill 可以规定检索策略、调用 MCP 工具以及读写记忆的条件，但不能替代三者各自的实现与安全边界。


### 10.1 核心机制与设计取舍


RAG 检索结果可能包含不可信内容，Skill 只能规定“需要证据”而不能保证证据真实。MCP Tool 负责连接服务，Runtime 负责鉴权和调用，Memory 则保存随时间变化的任务状态。


### 10.2 多层级知识网络

```mermaid
flowchart LR
  CORE10["与 MCP、RAG、Memory 的协作"]
  B10_0["外部接入"]
  CORE10 --- B10_0
  B10_0 --> N10_0_0["MCP Tools"]
  B10_0 --> N10_0_1["Resources"]
  B10_1["远程 Skill 分发"]
  CORE10 --- B10_1
  B10_1 --> N10_1_0["知识获取"]
  B10_1 --> N10_1_1["RAG"]
  B10_2["术语检索"]
  CORE10 --- B10_2
  B10_2 --> N10_2_0["证据校验"]
  B10_2 --> N10_2_1["状态协作"]
  B10_3["Memory"]
  CORE10 --- B10_3
  B10_3 --> N10_3_0["任务状态"]
  B10_3 --> N10_3_1["版本关联"]
  B10_0 -.-> B10_1
  B10_1 -.-> B10_2
  B10_2 -.-> B10_3
  B10_3 -.-> B10_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE10 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B10_0,N10_0_0,N10_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B10_1,N10_1_0,N10_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B10_2,N10_2_0,N10_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B10_3,N10_3_0,N10_3_1 module3;
```


<strong>图谱解读：</strong>以“与 MCP、RAG、Memory 的协作”为中心，分别连接“外部接入”“远程 Skill 分发”“术语检索”“Memory”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 10.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“MCP Tools”与“知识获取”建立可复现的验证用例。对于“任务状态”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 11. MCP Skills 扩展与跨平台分发

MCP Skills 扩展为远程 Skill 的发现和内容获取提供协议化方式。其实现与支持程度依赖协议版本、扩展版本和宿主。远程 Skill 的发布者、资源标识、摘要、缓存与授权应一起管理；不能把远程 Skill 的工具声明视为自动获得本机执行权限。


### 11.1 核心机制与设计取舍


### 11.2 多层级知识网络

```mermaid
flowchart LR
  CORE11["MCP Skills 扩展与跨平台分发"]
  B11_0["远程分发"]
  CORE11 --- B11_0
  B11_0 --> N11_0_0["发现清单"]
  B11_0 --> N11_0_1["内容读取"]
  B11_1["来源标识"]
  CORE11 --- B11_1
  B11_1 --> N11_1_0["一致性"]
  B11_1 --> N11_1_1["版本摘要"]
  B11_2["缓存策略"]
  CORE11 --- B11_2
  B11_2 --> N11_2_0["失效处理"]
  B11_2 --> N11_2_1["安全边界"]
  B11_3["授权独立"]
  CORE11 --- B11_3
  B11_3 --> N11_3_0["供应链"]
  B11_3 --> N11_3_1["租户隔离"]
  B11_0 -.-> B11_1
  B11_1 -.-> B11_2
  B11_2 -.-> B11_3
  B11_3 -.-> B11_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE11 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B11_0,N11_0_0,N11_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B11_1,N11_1_0,N11_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B11_2,N11_2_0,N11_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B11_3,N11_3_0,N11_3_1 module3;
```


<strong>图谱解读：</strong>以“MCP Skills 扩展与跨平台分发”为中心，分别连接“远程分发”“来源标识”“缓存策略”“授权独立”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 11.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“发现清单”与“一致性”建立可复现的验证用例。对于“供应链”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 12. 安全：提示注入、脚本与供应链

Skill 可能包含恶意指令、外部依赖、危险脚本或敏感信息外传路径。必须把 Skill 视为待审查的内容和代码供应链：来源校验、最小权限、沙箱、网络出口控制、凭据隔离、审批和审计都不能仅靠自然语言指令保证。


### 12.1 核心机制与设计取舍


风险控制应分为内容审查、执行沙箱、网络与文件权限、凭据隔离、审计追踪五层。对第三方 Skill 的更新应视作供应链变更，不能默认延续旧版本的信任。


### 12.2 多层级知识网络

```mermaid
flowchart LR
  CORE12["安全：提示注入、脚本与供应链"]
  B12_0["攻击面"]
  CORE12 --- B12_0
  B12_0 --> N12_0_0["提示注入"]
  B12_0 --> N12_0_1["恶意脚本"]
  B12_1["依赖投毒"]
  CORE12 --- B12_1
  B12_1 --> N12_1_0["防护机制"]
  B12_1 --> N12_1_1["沙箱"]
  B12_2["最小权限"]
  CORE12 --- B12_2
  B12_2 --> N12_2_0["审批"]
  B12_2 --> N12_2_1["供应链"]
  B12_3["签名摘要"]
  CORE12 --- B12_3
  B12_3 --> N12_3_0["来源审查"]
  B12_3 --> N12_3_1["回滚"]
  B12_0 -.-> B12_1
  B12_1 -.-> B12_2
  B12_2 -.-> B12_3
  B12_3 -.-> B12_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE12 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B12_0,N12_0_0,N12_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B12_1,N12_1_0,N12_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B12_2,N12_2_0,N12_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B12_3,N12_3_0,N12_3_1 module3;
```


<strong>图谱解读：</strong>以“安全：提示注入、脚本与供应链”为中心，分别连接“攻击面”“依赖投毒”“最小权限”“签名摘要”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 12.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“提示注入”与“防护机制”建立可复现的验证用例。对于“来源审查”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 13. 生命周期与质量评测

一个 Skill 的成熟过程是需求定义、开发、测试、审查、发布、监控、迭代和退役。测试必须同时覆盖是否触发与触发后是否成功，使用正样本、负样本、干扰上下文和失败注入；版本更新需进行回归测试。


### 13.1 核心机制与设计取舍


触发精度与召回率分别衡量误触发和漏触发，建议使用同一批标注任务进行评估：

$$
\mathrm{Precision}=\frac{TP}{TP+FP}
$$

$$
\mathrm{Recall}=\frac{TP}{TP+FN}
$$

其中 TP 为正确触发，FP 为错误触发，FN 为应该触发却未触发。若分母为零，应在评测协议中明确定义处理规则，不能直接将其解释为满分。


### 13.2 多层级知识网络

```mermaid
flowchart LR
  CORE13["生命周期与质量评测"]
  B13_0["生命周期"]
  CORE13 --- B13_0
  B13_0 --> N13_0_0["设计开发"]
  B13_0 --> N13_0_1["评测审查"]
  B13_1["发布监控"]
  CORE13 --- B13_1
  B13_1 --> N13_1_0["选择指标"]
  B13_1 --> N13_1_1["Precision"]
  B13_2["Recall"]
  CORE13 --- B13_2
  B13_2 --> N13_2_0["误触发率"]
  B13_2 --> N13_2_1["执行指标"]
  B13_3["任务成功率"]
  CORE13 --- B13_3
  B13_3 --> N13_3_0["延迟成本"]
  B13_3 --> N13_3_1["安全违规"]
  B13_0 -.-> B13_1
  B13_1 -.-> B13_2
  B13_2 -.-> B13_3
  B13_3 -.-> B13_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE13 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B13_0,N13_0_0,N13_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B13_1,N13_1_0,N13_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B13_2,N13_2_0,N13_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B13_3,N13_3_0,N13_3_1 module3;
```


<strong>图谱解读：</strong>以“生命周期与质量评测”为中心，分别连接“生命周期”“发布监控”“Recall”“任务成功率”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 13.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“设计开发”与“选择指标”建立可复现的验证用例。对于“延迟成本”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 14. 企业文档翻译 Agent 的能力设计

面向 Word、PDF 和图片翻译，主 Skill 可定义整体交付流程，子能力围绕 PDF 版式、图片文字翻译、术语规范、质量检查展开。文件解析、数字核对和渲染属于工具或脚本；术语规范是参考知识；长任务状态与权限由 Runtime 维护。


### 14.1 核心机制与设计取舍


| 能力 | 适合封装位置 | 原因 |
|---|---|---|
| 完整文档翻译交付 | 主 Skill | 面向可复用用户目标 |
| PDF 版式恢复规则 | 专用 Skill 或参考文件 | 视独立复用频率决定 |
| 数字、单位、型号比对 | 脚本或 Tool | 结果应确定且可测试 |
| 术语库查询 | RAG / MCP Tool | 数据动态更新，需来源和权限 |
| 长任务状态 | Runtime / 任务系统 | 需要持久化、取消和恢复 |

一次任务从文档识别开始，先判定可提取文本与扫描内容，按类型加载对应参考规则；随后查询术语、翻译内容并保留结构信息。校验阶段优先运行确定性数字与型号比对，再做语言和版式质量评估。出现失败时只对受影响区域修复，限制循环次数，并在高风险交付前要求人工复核。


### 14.2 多层级知识网络

```mermaid
flowchart LR
  CORE14["企业文档翻译 Agent 的能力设计"]
  B14_0["任务入口"]
  CORE14 --- B14_0
  B14_0 --> N14_0_0["文档识别"]
  B14_0 --> N14_0_1["翻译目标"]
  B14_1["质量约束"]
  CORE14 --- B14_1
  B14_1 --> N14_1_0["能力拆分"]
  B14_1 --> N14_1_1["版式保持"]
  B14_2["术语校验"]
  CORE14 --- B14_2
  B14_2 --> N14_2_0["数字型号检查"]
  B14_2 --> N14_2_1["执行治理"]
  B14_3["重试上限"]
  CORE14 --- B14_3
  B14_3 --> N14_3_0["人工复核"]
  B14_3 --> N14_3_1["交付追踪"]
  B14_0 -.-> B14_1
  B14_1 -.-> B14_2
  B14_2 -.-> B14_3
  B14_3 -.-> B14_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE14 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B14_0,N14_0_0,N14_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B14_1,N14_1_0,N14_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B14_2,N14_2_0,N14_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B14_3,N14_3_0,N14_3_1 module3;
```


<strong>图谱解读：</strong>以“企业文档翻译 Agent 的能力设计”为中心，分别连接“任务入口”“质量约束”“术语校验”“重试上限”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 14.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“文档识别”与“能力拆分”建立可复现的验证用例。对于“人工复核”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 15. 前沿方向、误区与工程选型

自动从成功轨迹提炼 Skill、跨宿主移植、远程分发与自动评测是值得关注的方向，但自动生成不代表已验证。不要把 Skill 数量、目录复杂度或脚本数量当作质量指标；应按任务复用率、触发准确率、成功率、风险和维护成本判断。


### 15.1 核心机制与设计取舍


前沿能力需要实验验证：从历史轨迹自动生成 Skill 可能过拟合偶然成功路径；动态组合可能放大指令冲突；远程分发可能扩大供应链攻击面。


### 15.2 多层级知识网络

```mermaid
flowchart LR
  CORE15["前沿方向、误区与工程选型"]
  B15_0["发展方向"]
  CORE15 --- B15_0
  B15_0 --> N15_0_0["自动提炼"]
  B15_0 --> N15_0_1["动态组合"]
  B15_1["持续评测"]
  CORE15 --- B15_1
  B15_1 --> N15_1_0["常见误区"]
  B15_1 --> N15_1_1["越多越好"]
  B15_2["等于 Agent"]
  CORE15 --- B15_2
  B15_2 --> N15_2_0["自动授权"]
  B15_2 --> N15_2_1["选型依据"]
  B15_3["复用频率"]
  CORE15 --- B15_3
  B15_3 --> N15_3_0["风险等级"]
  B15_3 --> N15_3_1["维护成本"]
  B15_0 -.-> B15_1
  B15_1 -.-> B15_2
  B15_2 -.-> B15_3
  B15_3 -.-> B15_0
  classDef center fill:#1D4ED8,color:#ffffff,stroke:#1E40AF,stroke-width:2px;
  class CORE15 center;
  classDef module0 fill:#DBEAFE,stroke:#64748B,color:#111827;
  class B15_0,N15_0_0,N15_0_1 module0;
  classDef module1 fill:#DCFCE7,stroke:#64748B,color:#111827;
  class B15_1,N15_1_0,N15_1_1 module1;
  classDef module2 fill:#FEF3C7,stroke:#64748B,color:#111827;
  class B15_2,N15_2_0,N15_2_1 module2;
  classDef module3 fill:#F3E8FF,stroke:#64748B,color:#111827;
  class B15_3,N15_3_0,N15_3_1 module3;
```


<strong>图谱解读：</strong>以“前沿方向、误区与工程选型”为中心，分别连接“发展方向”“持续评测”“等于 Agent”“复用频率”四类关联模块；虚线表示模块之间需要共同考虑的约束或依赖，并非强制的运行顺序。


### 15.3 工程落地与失败边界

实施时应先明确本节能力的输入、责任主体和成功判据，再为“自动提炼”与“常见误区”建立可复现的验证用例。对于“风险等级”等边界情形，应记录失败原因、触发上下文与实际执行轨迹；若行为涉及外部工具或写入操作，还必须在 Runtime 层实施权限和副作用控制。不要将 Skill 文本中的建议误当作已执行的检查。


## 16. 全局知识关联与工程选型

| 问题 | 优先选择 | 理由 |
|---|---|---|
| 一次性简单指令 | Prompt | 无需额外生命周期 |
| 确定性计算 | 函数或脚本 | 易测试、低开销 |
| 可重复的专业任务 | Skill | 复用决策知识与资源 |
| 强制审批和状态流转 | Workflow / Runtime | 必须由程序保证约束 |
| 远程共享工具 | MCP | 标准化能力发现与调用 |
| 动态知识证据 | RAG | 提供检索、排序和来源 |
| 跨会话业务状态 | Memory / 数据库 | 持久化和生命周期管理 |

<strong>核心原则：</strong>Skill 组织“如何完成任务”的可复用知识，Tool 提供“实际做什么”的操作能力，Runtime 保证“怎样安全、可靠地执行”，而 MCP 负责“怎样标准化连接外部能力”。四者不是替代关系，而是不同层级的协作关系。

## 17. 常见误区与面试要点

1. <strong>Skill 等于 Prompt 吗？</strong>不等于。Skill 是以 `SKILL.md` 为入口的能力包，可以包含脚本、参考文件和资产。
2. <strong>Skill 可以自己执行脚本吗？</strong>不能脱离宿主运行环境。执行权来自 Runtime 与工具权限。
3. <strong>为什么采用渐进式披露？</strong>为了减少无关上下文、改善能力组织，并允许按需使用细节。
4. <strong>Skill 越多越好吗？</strong>不是。粒度失衡会增加路由冲突、维护成本和误触发。
5. <strong>如何证明 Skill 有效？</strong>对比有无 Skill 的任务成功率、触发精度与召回、延迟、成本和安全事件。
6. <strong>Skill 与 MCP 如何配合？</strong>Skill 说明操作策略，MCP 暴露远程工具与资源，Runtime 执行权限控制。
7. <strong>怎样防止 Skill 投毒？</strong>验证来源、审查内容和脚本、隔离执行、限制网络权限、固定版本并记录审计。
8. <strong>什么时候不用 Skill？</strong>单次短指令、纯确定性函数、没有复用价值的任务，通常无需额外封装。

## 18. 参考资料与进一步验证

以下链接优先指向规范或官方仓库。规范与平台功能可能持续更新，涉及字段、路径、远程扩展时应以实际版本为准。

- [Agent Skills 开放规范](https://agentskills.io/specification)

- [Agent Skills 规范仓库](https://github.com/agentskills/agentskills)

- [Anthropic Skills 仓库](https://github.com/anthropics/skills)

- [OpenAI Skills 仓库](https://github.com/openai/skills)

- [OpenAI Skills 开发文档](https://developers.openai.com/api/docs/guides/tools-skills)

- [MCP 官方规范](https://modelcontextprotocol.io/specification/)

- [MCP Skills 扩展](https://skills.extensions.modelcontextprotocol.io/specification/stable/skills)

- [LangGraph 官方文档](https://docs.langchain.com/oss/python/langgraph/overview)


## 19. 总结

Agent Skills 的完整逻辑是：重复任务产生可复用需求，开放规范定义能力包，元数据支持发现，渐进式披露控制上下文，指令与资源指导执行，Runtime 与 Tool 负责真实动作，安全与评测保障长期复用。企业项目应将 Skill 视为需要版本、测试、权限与审计的工程资产，而不是一份写得很长的提示词。
