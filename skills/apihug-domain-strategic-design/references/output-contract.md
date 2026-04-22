# Domain Strategic Design Output Contract

此文件定义 `context-map.md` 的稳定输出骨架。
目标：
- 让 skill 在不同项目中都产出一致、可审查、可下游复用的战略设计文档
- 防止代理只画图、不落规则、不说明依赖

## 总体要求

- 文档主语言：中文
- 关键术语保留英文
- 图和表必须同时存在，不能只有其中一种
- 先给大域，再给 BC，再给关系，再给边界
- 内容必须面向：
  - 业务边界判断
  - 模块划分
  - 代码依赖控制
- 文档只给出模块化战略约束，不直接给出代码目录树、包结构或 source-set 结构
- 文档必须 UTF-8 正常可读；若存在乱码、编码异常或结构破损，不得视为完成态
- 固定章节必须完整出现，不得用前言、blockquote、零散备注替代正式章节
- 不得用“审查关注点”“说明备注”等松散段落替代：
  - `Source Basis 与锁定前提`
  - `待确认项`
  - `审计记录`

## `context-map.md` 固定章节

固定顺序如下：

1. 设计目标与范围
2. Source Basis 与锁定前提
3. 一级业务大域总览
4. Bounded Context 到业务大域映射
5. 大域级 Context Map
6. BC 级 Context Map
7. 边界规则
8. 外部系统与 ACL / Adapter 边界
9. 面向模块划分的落地含义
10. 待确认项
11. 审计记录

## 各章节要求

### 1. 设计目标与范围

必须说清：
- 当前文档服务什么阶段
- 是从零设计，还是在既有 domain model / BC 基础上细化
- 当前文档不做什么
- 若已有旧 `context-map.md`，必须明确它是待修订对象还是仅弱参考

frontmatter 必须包含：

- `sources`
- `reviewStatus.round0StructureCompleteness`
- `reviewStatus.round1SourceAlignment`
- `reviewStatus.round2DDDConsistency`
- `reviewStatus.round3ImplementationReadiness`
- `reviewStatus.lastReviewedAt`
- `reviewStatus.reviewNotes`
- `finalizationLevel`
- `readyForArchitecture`
- `projectContextSynced`

字段约束：

- `reviewStatus.*` 取值：`pending / passed / failed`
- `finalizationLevel` 取值：`Draft / Ready With Gaps / Final Baseline`
- `readyForArchitecture` 取值：`true / false`
- `projectContextSynced` 取值：`true / false`

### 2. Source Basis 与锁定前提

至少包含两张表：

#### 2.1 Source 输入表

列至少包括：
- Source
- 类型
- 用途
- 优先级

#### 2.2 锁定前提表

列至少包括：
- 项
- 是否锁定
- 处理规则

常见锁定项：
- Domain Model
- Bounded Context
- Phase Scope

### 3. 一级业务大域总览

必须是一张大域总览表。

列至少包括：
- 一级业务大域
- 职责
- 状态
- 说明

状态统一只用：
- `IMPLEMENTED`
- `PLANNED`
- `FUTURE SLOT`

### 4. Bounded Context 到业务大域映射

必须是一张映射表。

列至少包括：
- Bounded Context
- 归属一级业务大域
- 状态
- 说明

若当前尚无 BC，但大域需要预留，可在该章显式标注“暂无已锁定 BC”。

### 5. 大域级 Context Map

必须包含：
- 关系说明表
- Mermaid 全景图

#### 5.1 关系说明表

列至少包括：
- 上游 → 下游
- 关系类型
- 数据流
- 说明

关系类型仅允许：
- `Customer-Supplier`
- `Partnership`
- `ACL`
- `Conformist`
- `Shared Kernel`

#### 5.2 大域级 Mermaid

要求：
- 按一级业务大域分组
- 若 source 明确存在关键边界类别（如平台域/业务域/治理域/外部系统边界），可用子图表达
- 节点标签推荐 `English(中文)`
- 图中必须能看出依赖方向

### 6. BC 级 Context Map

必须包含：
- BC 关系表
- BC Mermaid 图

要求：
- 只出现已锁定或本阶段明确需要表达的 BC
- 不要把聚合、实体、值对象塞进这张图
- 这张图是战略边界图，不是战术模型图

### 7. 边界规则

必须编号列出。

每条边界规则至少说明：
- 规则编号
- 规则
- 影响范围

至少覆盖：
- 依赖方向
- source 支持的关键边界类别边界
- 外部系统接入边界
- 治理/审计边界
- 多租户边界（如适用）

### 8. 外部系统与 ACL / Adapter 边界

必须是一张表。

列至少包括：
- 外部系统
- 接入方式
- 进入域
- 关系类型
- 说明

### 9. 面向模块划分的落地含义

必须明确写出：
- 一级模块是否按一级业务大域组织
- 域内是否承载一个或多个 BC
- 哪些模块是 Phase 1 优先落地

同时必须明确当前项目属于哪一种：

- 单体多模块
- 单体单模块

若是单体单模块，必须写清：

- 物理上保持单模块
- 逻辑上按一级业务大域或 BC 切片治理
- 具体代码目录与包结构转交 `architecture.md`

本章至少包含一张正式映射表：
- 一级业务大域
- 一级模块建议
- 说明

不得只给模块名列表而没有映射说明。
不得在本章直接输出 `src/main/...` 目录树或 package tree。

### 10. 待确认项

只保留真正无法由 source 或合理推断解决的问题。

每项至少包括：
- 编号
- 问题
- 影响
- 候选方向

不得用“人工审查建议”“审查关注点”替代本章。

### 11. 审计记录

必须记录至少三轮审计结果：
- Source Alignment
- DDD Consistency
- Implementation Readiness

每轮至少包括：
- 轮次
- 审计目标
- 发现
- 处理结果
- 当前状态

不得把审计结果只放在会话消息里而不落入文档。

## 全景图规范

若文档包含“领域全景图”，必须满足：
- 先按一级业务大域分组
- 域内再放 BC / 子域
- 不把技术层名当域名
- 可带图例，如：
  - `★ Core`
  - `○ Supporting`
  - `◇ Generic`
  - `▧ Semantic Contract`
  - `▤ Event Projection`
  - `▥ Domain Service`

但：
- 图例是可选辅助，不得替代正文表格
- 图中非 BC 元素只能用于解释战略关系，不得演变为技术实现图
- 若使用文本框图/ASCII 图，仍不得省略 Mermaid 正式关系图

## 图文一致性要求

- 表格中的状态必须与图中节点状态一致
- 大域名、BC 名称必须前后一致
- 图中出现的关系，表中必须有解释
- 表中出现的关键关系，图中应尽量体现

## 后置同步要求

完成或更新 `context-map.md` 后，必须检查 `docs/project-context.md` 是否已声明：

- 若 `docs/planning-artifacts/context-map.md` 存在，则后续 architecture 工作必须将其视为正式上游输入
- `architecture.md` 不得静默推翻已锁定的大域边界、BC 归属和关键上下文关系
- 具体模块 / 目录 / 包结构由 `architecture.md` 决定，不由 `context-map.md` 决定
- `project-context.md` 的同步应遵循“只补缺、不覆盖项目特定规则”
