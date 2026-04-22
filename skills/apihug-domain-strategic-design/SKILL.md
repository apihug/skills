---
name: apihug-domain-strategic-design
description: Create or refine a DDD strategic design context-map.md with macro domains, bounded context mapping, context relationships, boundary rules, audit rounds, and implementation-oriented module implications. Use when the user asks for DDD strategic design, context map design, domain landscape design, cross-domain boundary design, or wants a context-map.md produced or reviewed.
---

# Domain Strategic Design Skill

任务目标：
- 产出或重构 `docs/planning-artifacts/context-map.md`
- 先按**一级业务大域**组织，再统合其下的 Bounded Context / 子域
- 定义上下文关系、边界规则、外部系统防腐层、以及 source 明确支持的关键边界类别解耦规则
- 让文档能直接支撑下一步：
  - 一级模块边界建议
  - architecture.md 的正式上游模块约束
  - architecture.md 细化
  - 后续战术设计分域落盘

本 skill 是**战略设计 skill**，不是战术设计 skill。
它不负责：
- 聚合、实体、值对象细节
- 物理表设计
- Story 拆解
- 具体代码目录树 / package tree / source-set 设计

## 术语定义

- `一级业务大域`
  指面向长期业务职责划分的最高层领域边界，用于承载一个或多个 Bounded Context，并为后续模块设计提供稳定边界。
- `BC / Bounded Context`
  指限界上下文，是某一组统一语言、规则和职责成立的边界。BC 可以来自上游已锁定输入，也可以在本阶段基于 source 识别出来。
- `锁定输入`
  指项目中已被确认、默认不可随意推翻的战略输入，例如已确认的 Domain Model、已确认的 BC 划分或明确的 phase 范围。
- `弱参考`
  指可被读取但不能自动视为真相源的输入，例如既有 `context-map.md` 或仅表达旧结论的历史文档。
- `一级模块边界建议`
  指“哪些一级业务大域未来适合映射为一级模块，或在单模块项目中映射为一级逻辑切片”的战略建议，不等于具体模块目录树。
- `architecture.md` 的正式上游模块约束
  指 `context-map.md` 向后续架构设计提供的正式约束集合，包括一级业务大域、BC 归属、依赖方向、关系类型和边界规则。
- `单体多模块`
  指物理上为单体部署，但代码按多个一级模块组织。
- `单体单模块`
  指物理上和代码上都保持单模块，但仍需在模块内部按大域或 BC 保持逻辑隔离。

## 输出位置

固定输出：
- `docs/planning-artifacts/context-map.md`

如项目缺少 `docs/planning-artifacts/`，先初始化该目录，再创建 `context-map.md`。

本 skill 对外保持单入口：

- `$apihug-domain-strategic-design`

但内部执行必须分为三个阶段：

1. Phase A: Strategic Boundary Modeling
2. Phase B: Context Map Consolidation
3. Phase C: Audit, Finalization & Downstream Sync

## 输入与读取优先级

### 必读输入

1. `docs/project-context.md`
2. `docs/planning-artifacts/prd.md`
3. `references/output-contract.md`
4. `references/review-checklist.md`
5. `references/finalization-standard.md`

### 条件读取输入

仅当文件存在且对当前战略边界有直接影响时再读取：

- 当前战略设计相关输入文档
- `docs/planning-artifacts/domain-model*.md`
- `docs/planning-artifacts/architecture.md`
- `docs/planning-artifacts/context-map.md`
- `docs/input-artifacts/**`
- 其它已完成的战略/研究文档

### source 优先级

当 source 存在冲突时，按以下优先级收敛：

1. `docs/planning-artifacts/prd.md`
2. 已锁定的 `docs/planning-artifacts/domain-model*.md`
3. `docs/input-artifacts/**`
4. 其它已完成的战略/研究文档
5. `docs/project-context.md`
6. `docs/planning-artifacts/architecture.md`
7. `docs/planning-artifacts/context-map.md`

说明：

- 在推荐链路 `PRD -> Context Map -> Architecture` 中，`architecture.md` 不是战略真相源，只是技术约束与既有实现参考。
- `architecture.md` 不得推翻 `PRD`、已锁定 Domain Model、已锁定 BC 或已确认的大域边界。
- 既有 `context-map.md` 只作为弱参考或待修订对象，不得被它绑架。
- `docs/project-context.md` 提供 workflow 级约束，但不主导业务边界建模。
- 若某项 BC 归属已被上游文档明确锁定，则其结论优先于弱参考文档，不得被历史 `context-map.md` 或 `architecture.md` 静默改写。

若项目中已明确：
- 已完成 Domain Model
- 已完成 Bounded Context 划分

则默认把它们视为**锁定输入**，后续只能细化、映射、校正表达，不得擅自推翻，除非用户明确要求重做。

### 上游 readiness 传递规则

本 skill 的正式上游是 `prd.md`。启动前 SHOULD 判断上游成熟度：

- 若 `prd.md` 仅为 `Draft`，SHOULD NOT 启动正式战略设计，除非用户明确要求探索性草稿。
- 若 `prd.md` 为 `Ready With Gaps`，MAY 启动，但 MUST 在 `context-map.md` 的"待确认项"中继承其非闭环项，并在 frontmatter 中标注 `finalizationLevel` 不高于 `Ready With Gaps`。
- 若 `prd.md` 为 `Final Baseline`，可按正式基线消费。

若上游 gap 会直接破坏一级业务大域识别或核心 BC 归属判断，则本 skill 最多只能产出 `Draft` 或探索性结果。

### 当前战略设计相关输入文档判定规则

仅在满足以下任一条件时，才视为“当前战略设计相关输入文档”：

- 文档直接描述一级业务大域、BC、上下文关系或边界规则
- 文档明确给出 phase 范围、已锁定 BC、外部系统边界或平台/业务解耦约束
- 文档为当前战略设计提供新增且可验证的业务边界证据

以下内容默认不算：

- 只描述技术实现细节、不提供新增战略约束的文档
- 只引用大域名称、但不提供边界信息的交付文档
- 其它大域战术设计文档

## 核心原则

1. 先定一级业务大域，再挂接 Bounded Context。
2. 不用技术分层名替代业务大域名。
3. 关键边界类别是 **source 驱动的解耦规则**，不是天然的大域名字。
4. Context Map 必须能服务代码模块化，而不是只给人看热闹。
5. Context Map 只回答“按什么业务边界切模块或逻辑切片”，不回答“代码目录具体怎么摆”。
6. 关系类型必须明确，不用泛化箭头混过去。
7. 外部系统进入业务域时，必须明确是否通过 `ACL / Adapter`。
8. 文档必须对照 source 做多轮审计后再定稿，不能一稿即收。

## 允许做的事

- 定义一级业务大域
- 将现有 BC / 子域映射到一级业务大域
- 定义上下文关系：
  - `Customer-Supplier`
  - `Partnership`
  - `ACL`
  - `Conformist`
  - `Shared Kernel`
- 明确依赖方向、事件流、命令流
- 定义 source 支持的关键边界类别边界规则
- 给出面向模块划分的落地含义
- 给出单体单模块场景下的逻辑切片约束
- 标记：
  - `IMPLEMENTED`
  - `PLANNED`
  - `FUTURE SLOT`

## 不允许做的事

- 把技术层命名伪装成一级业务大域
- 把类图 / 聚合图塞进 `context-map.md`
- 擅自新增或删除用户已锁定的 Core Domain / BC
- 只给一张图，不给关系表和边界规则
- 只平铺 BC，不先做大域组织
- 不审 source 就直接拍板
- 直接输出代码目录树、包结构或 Gradle/source-set 结构

## 标准执行流程

### Phase A: Strategic Boundary Modeling

本阶段不是模板填空阶段，而是战略边界建模阶段。
执行器必须按 `bmad-agent-architect` 的建模标准工作，而不是仅根据模板占位直接生成 `context-map.md`。
无论当前是从零设计、细化既有战略输入，还是校正旧 `context-map.md`，都必须按这一标准执行。

#### 第 1 步：锁定输入边界

先判断当前项目属于哪一种：

1. **从零战略设计**
- 没有成型的 domain model / bounded context
- 必须由执行器按 `bmad-agent-architect` 的建模标准，结合 `PRD / input-artifacts / 已锁定输入` 进行战略推导
- 允许：识别一级业务大域、识别 BC 候选、建立初始关系与边界规则
- 禁止：跳过 source 直接按模板脑补完整战略边界

2. **已有战略输入，进行细化**
- 已有 Domain Model / BC 划分
- 默认只能细化，不重划
- 仍必须按 `bmad-agent-architect` 的建模标准执行
- 允许：细化一级业务大域职责、补全 BC 映射、补全关系与边界规则、修正表达不清之处
- 禁止：推翻已锁定 Domain Model、已锁定 BC 归属或已确认 phase 范围，除非用户明确要求重做

3. **已有 context-map.md，进行校正重构**
- 旧 context-map.md 存在
- 仅把它当弱参考，不能被它绑架
- 仍必须按 `bmad-agent-architect` 的建模标准执行
- 允许：修复结构缺失、修正错误关系类型、补齐边界规则、清理技术层假大域、统一术语与状态
- 禁止：把旧 `context-map.md` 当作天然真相源，或在缺乏 source 支撑时重划一级业务大域和核心 BC 边界

#### 第 2 步：先抽取一级业务大域

必须先回答：
- 系统应划分为哪些一级业务大域
- 每个大域的职责是什么
- 每个大域是否足以成为未来一级代码模块或单模块内的一级逻辑切片

一级业务大域命名要求：
- 以业务职责命名
- 稳定、可长期演进
- 不能直接写成：
  - `Platform Foundation`
  - `Application Layer`
  - `MES Execution`
  - `Shared Services`
  这类技术层或实现层名

#### 第 3 步：将现有 BC / 子域挂接到一级大域

必须明确：
- 哪些 BC 已有并锁定
- 各自归属哪个一级业务大域
- 哪些一级业务大域当前只是槽位，还没有落 BC

默认状态标记：
- `IMPLEMENTED`：当前 PRD / 战略范围已落入当前阶段
- `PLANNED`：后续阶段已明确会做
- `FUTURE SLOT`：仅预留槽位，当前不设计内部细节

#### 第 4 步：定义上下文关系

关系必须单独列表，不得只藏在图里。

每条关系至少说明：
- 上游 → 下游
- 关系类型
- 数据流
- 说明

关系类型使用规则：
- `Customer-Supplier`
  - 上游提供稳定能力，下游消费
- `Partnership`
  - 两侧紧密协作，但不代表共享数据库或双向编译依赖
- `ACL`
  - 外部系统进入内部业务域前必须防腐
- `Conformist`
  - 明确接受上游语言与约束
- `Shared Kernel`
  - 仅在 source 明确支持时使用

#### 第 5 步：定义边界规则

至少要覆盖：
- source 明确存在的关键边界类别解耦规则
- 治理/审计边界（如存在）
- 租户隔离边界（如存在）
- 外部系统防腐层边界
- 事件与命令的主要流向

#### 第 5.5 步：明确向 architecture.md 的移交边界

必须明确：

- `context-map.md` 负责一级业务大域、BC 归属、依赖方向、模块化约束
- `architecture.md` 负责具体模块拓扑、单体单模块/单体多模块决策、代码目录与包结构
- 若项目是单体单模块，则 `context-map.md` 只输出“模块内按大域/BC 保持逻辑隔离”的约束，不直接产出目录树

本阶段要求：

- 必须以 `bmad-agent-architect` 的方式完成战略建模，再进入文档落盘
- 优先解决一级业务大域、BC 归属、关系类型、边界规则
- 不得先写 Mermaid 图再反推战略边界
- 若 source 冲突不可收敛，必须在“待确认项”中显式保留
- `context-map.md` frontmatter 中 `sources:` 必须仅记录本次实际采用的输入
- `context-map.md` frontmatter 必须维护结构化状态字段：`reviewStatus`、`finalizationLevel`、`readyForArchitecture`、`projectContextSynced`

战略推导至少必须完成以下收敛项：

1. 一级业务大域识别
2. 已锁定 BC 与候选 BC 识别
3. BC 到一级业务大域映射
4. 上下文关系类型收敛
5. source 支持的关键边界类别边界规则收敛
6. 单体单模块场景下的逻辑切片约束（如适用）

若以上任一项尚未完成收敛：

- 不得进入 `context-map.md` 的正式落盘
- 不得进入审计轮次
- 必须先继续 Phase A 建模，或在“待确认项”中显式保留阻塞问题

### Phase B: Context Map Consolidation

#### 第 6 步：落 `context-map.md`

进入本阶段前，必须确认：

- 已完成 `bmad-agent-architect` 标准下的战略边界建模
- 一级业务大域、BC 映射、关系类型、边界规则已达到可成文状态
- 不存在会直接导致 `context-map.md` 结构失真的核心边界悬空项

必须按 output contract 的固定结构产出。

读取：
- `references/output-contract.md`
- `templates/context-map-template.md`

完成 `context-map.md` 后，必须检查并在需要时更新 `docs/project-context.md`，确保其显式声明：

- 若 `docs/planning-artifacts/context-map.md` 存在，则后续 architecture 工作必须将其视为正式上游输入
- `architecture.md` 不得静默推翻已锁定的一级业务大域、BC 归属或关键上下文关系
- 具体模块 / 目录 / 包结构由 `architecture.md` 决定，不由 `context-map.md` 决定

更新原则：

- 只补充或校正缺失的 workflow 规则
- 不覆盖已有项目特定实现规则
- 若已有等价规则，只统一表述，不重复堆叠

### Phase C: Audit, Finalization & Downstream Sync

#### 第 7 步：三轮审计

读取：
- `references/review-checklist.md`
- `references/finalization-standard.md`

必须先做结构完整性检查，再做三轮审计：

0. **Structure Completeness**
- 是否满足 output contract 固定章节
- 是否把“待确认项/审计记录”真正落文档

1. **Source Alignment 审计**
- 是否忠实覆盖 PRD / input-artifacts
- 是否和已有 domain model / bounded contexts 冲突

2. **DDD Consistency 审计**
- 一级大域是否真的是业务域
- 关系类型是否使用正确
- 是否混入技术分层假大域

3. **Implementation Readiness 审计**
- 是否能指导一级模块划分
- 是否能指导代码依赖方向
- 是否存在会让后续 architecture / tactical design 返工的结构矛盾
- 文档是否 UTF-8 正常可读、无乱码、无结构破损

若审计发现问题：
- 先修正文档
- 再回跑相关审计轮次
- 直到没有结构性矛盾后再定稿

每轮审计结束后，执行器应至少更新 `context-map.md` frontmatter：

- `reviewStatus.round0StructureCompleteness`
- `reviewStatus.round1SourceAlignment`
- `reviewStatus.round2DDDConsistency`
- `reviewStatus.round3ImplementationReadiness`
- `reviewStatus.lastReviewedAt`
- `reviewStatus.reviewNotes`

#### 第 8 步：定稿判定

读取：
- `references/finalization-standard.md`

必须在最终输出时明确给出：
- 当前定稿等级：
  - `Draft`
  - `Ready With Gaps`
  - `Final Baseline`
- 判定理由
- 剩余待确认项
- 是否可以继续：
  - architecture
  - module structure
  - tactical design

定稿前必须同时确认：

- `finalizationLevel` 已同步到 frontmatter
- `readyForArchitecture` 已同步到 frontmatter
- `projectContextSynced` 已同步到 frontmatter

## 提示词与生成指令

读取：
- `references/prompt-pack.md`

用法：
- 初稿：使用“初始生成提示词”
- 审计 1：使用“Source Alignment 审计提示词”
- 审计 2：使用“DDD Consistency 审计提示词”
- 审计 3：使用“Implementation Readiness 审计提示词”
- 最终收口：使用“Finalization 提示词”

## 强制输出要求

产出完成后，必须明确说明：
- 依据了哪些 source
- 哪些内容是锁定输入
- 哪些内容是合理推断
- 进行了哪几轮审计
- 是否还有待确认项
- `docs/project-context.md` 是否已同步上述 architecture 上游约束

## 附加资源

- `references/output-contract.md`
- `references/review-checklist.md`
- `references/finalization-standard.md`
- `references/prompt-pack.md`
- `templates/context-map-template.md`
