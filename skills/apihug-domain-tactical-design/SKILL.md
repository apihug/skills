---
name: apihug-domain-tactical-design
description: Generate, review, and refine domain tactical design documents under BMAD and DDD rules. Use when creating or refining domain tactical design docs from tactical-design.md to 01~06.
---

# Domain Tactical Design Skill

## 任务目标

- 围绕某一个一级业务大域生成并收敛一套 DDD 战术设计文档。
- 先完成主文 `tactical-design.md` 的建模与审计，再派生 `01~06` 标准件。
- 将战术设计对后续 Epic / Story 拆解的影响通过 `architecture.md` 的大域入口索引体现出来。

## 非目标

本 skill 不负责：

- 重做战略设计或重划一级业务大域。
- 静默重划已锁定的 Bounded Context。
- 直接修改 `bmad-create-epics-and-stories`。
- 直接产出代码实现、目录树、package tree 或 source-set 设计。
- 把其它大域的战术文档当作当前大域的事实源。

## 术语定义

- `主文`
  指当前大域的 `tactical-design.md`，是本大域战术设计的主事实源。
- `标准件`
  指从主文派生的 `01~06` 六份标准化文档。
- `派生`
  指从主文做标准化展开，不得引入主文未声明的新设计事实。
- `建模收敛`
  指围绕边界、聚合、服务职责、事件主面、关键约束完成可落文档的稳定判断。
- `锁定输入`
  指已被上游文档确认、默认不得擅自推翻的大域边界、BC 归属或 phase 范围。
- `弱参考`
  指可读可参考，但不能默认覆盖更强 source 的历史战术文档或低优先级资料。

## 输出位置

固定输出目录：

`docs/planning-artifacts/domains/<domain>/`

## 外部入口与内部阶段

本 skill 对外保持单入口：

- `$apihug-domain-tactical-design`

但内部执行必须分为两个阶段：

1. Phase A: Tactical Core Modeling
2. Phase B: Derived Standard Artifacts & Architecture Index Sync

## 必读输入

1. `docs/BMAD.md`
2. `docs/planning-artifacts/prd.md`
3. `docs/planning-artifacts/context-map.md`
4. `docs/planning-artifacts/architecture.md`
5. `references/output-contract.md`
6. `references/review-checklist.md`
7. `references/project-bootstrap.md`
8. `references/finalization-standard.md`
9. `references/prompt-pack.md`

## 条件读取输入

仅当文件存在且对当前大域有直接影响时再读取：

- `docs/planning-artifacts/domain-model-phase1.md`
- `docs/project-context.md`
- `docs/planning-artifacts/domains/domain-design-rules.md`
- 当前大域相关输入文档
- 当前大域既有战术设计文档

## source 优先级

当 source 存在冲突时，按以下优先级收敛：

1. `docs/planning-artifacts/prd.md`
2. `docs/planning-artifacts/context-map.md`
3. `docs/planning-artifacts/architecture.md`
4. 当前大域相关输入文档
5. `docs/planning-artifacts/domain-model-phase1.md`
6. `docs/planning-artifacts/domains/domain-design-rules.md`
7. `docs/project-context.md`
8. 当前大域既有战术设计文档

规则：

- 既有战术设计文档属于弱参考，不得默认覆盖更高优先级 source。
- 若更高优先级 source 未明确，但较低优先级 source 给出约束，可采用，并在主文中标注依据。
- 若高低优先级 source 不可收敛，必须写入“待确认项”，禁止静默选边。

## 输入发现规则

仅当满足以下任一条件时，才视为“当前大域相关输入文档”：

- 文档直接描述该大域的业务边界、子域、聚合、规则、事件或落库约束。
- 文档明确声明该大域在当前 phase 内，且给出新增设计约束。
- 文档对该大域的 BC 覆盖范围、依赖关系或实施优先级给出明确限定。

以下内容默认不算：

- 仅顺带提及该大域的普通说明文档。
- 只做实现回顾、未提供新增设计约束的交付文档。
- 其它大域目录下的战术设计文档。

## 锁定输入与弱参考

- 若 `context-map.md` 已锁定大域定位、BC 归属和上下游边界，则当前 skill 只能细化，不能重划。
- 若 `architecture.md` 已锁定物理模块边界和依赖方向，则当前 skill 只能在其约束下细化，不得静默推翻。
- 既有 `tactical-design.md` 或同域历史战术文档只属于弱参考；可用于校正表达，不可当真相源。

## 上游 readiness 传递规则

本 skill 的正式上游至少包括：

- `prd.md`
- `context-map.md`
- `architecture.md`

启动前 SHOULD 判断上游成熟度：

- 若任一正式上游仅为 `Draft`，SHOULD NOT 启动正式战术设计，除非用户明确要求探索性草稿。
- 若正式上游为 `Ready With Gaps`，MAY 启动，但 MUST 在主文“待确认项”中继承其未闭环项，并且 `finalizationLevel` 不得高于 `Ready With Gaps`。
- 若上游 gap 会直接破坏当前大域的 BC 归属、聚合建模或关键依赖判断，则当前 skill 最多只能产出 `Draft` 或 `Conditional` 状态结果。
- 若正式上游均为 `Final Baseline`，可按正式基线消费。

## Bootstrap 规则

若项目缺少 `docs/planning-artifacts/domains/` 或缺少最低骨架，必须先补齐：

- `docs/planning-artifacts/domains/`
- `docs/planning-artifacts/domains/README.md`
- `docs/planning-artifacts/domains/domain-design-rules.md`
- `docs/planning-artifacts/domains/<domain>/`

Bootstrap 只补缺，不覆盖已有项目内容。

## 核心原则

1. 不重划已锁定的大域或 BC，除非更上游文档明确要求修改。
2. `tactical-design.md` 是当前大域的主事实源。
3. `01~06` 必须从 `tactical-design.md` 派生，不得独立发散。
4. source 不足时必须写“待确认项”，不得补脑。
5. 对后续 Story 拆解的影响通过 `architecture.md` 的稳定入口体现，而不是要求修改其它 skill。

## 允许做的事

- 在既有大域和 BC 边界内完成战术建模收敛。
- 定义聚合、实体、值对象、服务职责、命令、事件、规则和落库约束。
- 从主文派生 `01~06`。
- 同步 `architecture.md` 中该大域的路径、状态、覆盖范围与 readiness 信号。

## 不允许做的事

- 先写 `01~06` 再反推主文。
- 让标准件长期携带“主文未声明，但标准件单独补出”的事实。
- 直接输出代码目录树、package tree 或 source-set 结构。
- 把其它大域现成设计直接当作当前大域模板。

## 标准执行流程

### Phase A: Tactical Core Modeling

本阶段不是模板填空阶段，而是主文建模阶段。执行器必须按 `bmad-agent-architect` 的建模标准工作。

#### 场景 1：从零战术设计

- 允许：基于上游 source 完成当前大域的完整战术建模。
- 禁止：跳过主文建模直接产出标准件。

#### 场景 2：已有上游边界，细化战术设计

- 允许：细化聚合、服务职责、事件主面、关键约束和落库判断。
- 禁止：推翻已锁定的大域边界和 BC 归属。

#### 场景 3：已有主文，校正重构

- 允许：修复结构缺失、矛盾表达、规则遗漏和标准件漂移。
- 禁止：把旧主文当真相源，或在缺乏 source 支撑时重画边界。

#### Phase A 必须完成的收敛项

1. 锁定目标大域与 BC 覆盖范围。
2. 收敛聚合、实体、值对象和服务职责。
3. 收敛命令、事件、查询和关键规则。
4. 明确关键依赖、跨域协作与约束。
5. 生成或修订 `tactical-design.md`。

若上述任一项未收敛：

- 不得进入标准件派生。
- 不得进入审计轮次。
- 必须继续 Phase A 或显式写入“待确认项”。

### Phase B: Derived Standard Artifacts & Architecture Index Sync

本阶段只做标准件展开与下游接口同步，不重新做主建模。

必须完成：

1. 从主文派生 `01~06`。
2. 检查 `01~06` 是否引入主文未声明的新事实。
3. 若发现主文缺口，先回写主文，再重新同步标准件。
4. 回写 `architecture.md` 中该大域的路径、状态、覆盖范围与 readiness 信号。

## 审计与定稿流程

读取：

- `references/review-checklist.md`
- `references/finalization-standard.md`
- `references/prompt-pack.md`

审计轮次固定为：

1. Round 0: Structure Completeness
2. Round 1: Source Alignment
3. Round 2: DDD Consistency
4. Round 3: Implementation Readiness

每轮审计结束后，必须至少同步主文 frontmatter：

- `reviewStatus.round0StructureCompleteness`
- `reviewStatus.round1SourceAlignment`
- `reviewStatus.round2DDDConsistency`
- `reviewStatus.round3ImplementationReadiness`
- `reviewStatus.lastReviewedAt`
- `reviewStatus.reviewNotes`

## prompt-pack 用法

读取：

- `references/prompt-pack.md`

用法：

- 初稿：使用“初始建模提示词”
- 审计 1：使用“Round 1: Source Alignment 审计提示词”
- 审计 2：使用“Round 2: DDD Consistency 审计提示词”
- 审计 3：使用“Round 3: Implementation Readiness 审计提示词”
- 最终收口：使用“Finalization 提示词”

## 强制输出要求

产出完成后，必须明确说明：

- 依据了哪些 source。
- 哪些内容是锁定输入。
- 哪些内容是合理推断。
- 进行了哪几轮审计。
- 是否仍有待确认项。
- `architecture.md` 是否已同步大域入口索引。

## 附加资源

- `references/output-contract.md`
- `references/review-checklist.md`
- `references/project-bootstrap.md`
- `references/finalization-standard.md`
- `references/prompt-pack.md`
- `templates/tactical-design-template.md`
- `templates/01-events-commands-terms-template.md`
- `templates/02-domain-rules-template.md`
- `templates/03-uml-template.md`
- `templates/04-glossary-template.md`
- `templates/05-architecture-template.md`
- `templates/06-physical-data-model-template.md`
- `templates/domains-readme-template.md`
- `templates/project-domain-design-rules-template.md`
