# Strategic Design Review Checklist

本文档定义 `context-map.md` 的四轮审计清单。

## Round 0: Structure Completeness

目标：
- 确认文档结构满足 output contract

检查项：
1. 固定 11 个章节是否完整出现。
2. `Source Basis 与锁定前提` 是否是正式章节，而不是前言备注。
3. `待确认项` 是否真实存在，而不是被“审查关注点”替代。
4. `审计记录` 是否真实落文档，而不是只存在于会话。
5. `面向模块划分的落地含义` 是否包含正式映射表，而不是只有列表。
6. frontmatter 状态字段是否完整存在。
7. 文档是否 UTF-8 正常可读、无乱码。

输出要求：
- 若结构不完整，先修结构，再进入 Round 1。

## Round 1: Source Alignment

目标：
- 确认文档忠实覆盖 source。
- 确认没有把旧文档当真相。

检查项：
1. 是否明确列出所有核心 source。
2. 是否把旧 `context-map.md` 仅作为弱参考，而不是默认真相。
3. 若 Domain Model / BC 已锁定，是否明确说明“只能细化，不重划”。
4. 一级业务大域是否能从 PRD / input-artifacts / 其它上游 source 得到支撑。
5. 是否遗漏当前阶段明确存在的重要业务域或边界域。
6. 状态 `IMPLEMENTED / PLANNED / FUTURE SLOT` 是否与 source 阶段一致。
7. 是否把后续阶段内容提前误写成当前阶段已落地。
8. `architecture.md` 是否仅作为参考约束，而非战略真相源。

输出要求：
- 列出发现。
- 标记是否需要修正。
- 修改后回查本轮。

## Round 2: DDD Consistency

目标：
- 确认文档是战略设计，而不是技术分层图。

检查项：
1. 一级业务大域是否真的是业务职责域，而不是技术域。
2. 是否误用 `Platform Foundation`、`Application Layer` 这类假大域。
3. 一级业务大域与 BC 的层级关系是否清晰。
4. BC 是否被错误地直接升级为一级业务大域。
5. 关系类型是否使用准确：
   - `Customer-Supplier`
   - `Partnership`
   - `ACL`
   - `Conformist`
   - `Shared Kernel`
6. `Partnership` 是否被滥用。
7. `ACL` 是否正确落在外部系统边界上。
8. 是否明确 source 支持的关键边界类别之间的依赖方向。
9. 图中是否混入聚合、实体、值对象等战术元素。
10. 是否越界给出了具体代码目录树或 package tree。

输出要求：
- 列出结构性问题。
- 给出修正建议。
- 修正后确认“无技术层假大域”。

## Round 3: Implementation Readiness

目标：
- 确认文档可直接支撑后续模块划分与 architecture 细化。

检查项：
1. 一级业务大域是否足以映射为未来一级代码模块或单模块内一级逻辑切片。
2. 是否明确“一 级模块按大域组织，域内承载一个或多个 BC”。
3. 是否明确依赖方向可指导代码依赖控制。
4. 是否能据此继续做 `architecture.md` 的模块矩阵。
5. 是否能据此继续做各大域战术设计落盘。
6. 是否存在会让下游返工的悬空关系。
7. 是否存在状态、关系、边界规则前后不一致。
8. `docs/project-context.md` 是否已同步 architecture 上游约束。

输出要求：
- 结论应明确为：
  - `NOT READY`
  - `READY WITH GAPS`
  - `READY`
- 结论词汇映射如下：
  - `NOT READY` -> 最终定稿等级不得高于 `Draft`
  - `READY WITH GAPS` -> 最终定稿等级对应 `Ready With Gaps`
  - `READY` -> 表示可进入 `Final Baseline` 判定，但仍必须再对照 `finalization-standard.md`

## 审计收口规则

- 不允许跳过 Round 0。
- 若某轮发现结构性问题，必须先修文档，再重新执行该轮。
- 不允许跳过 Round 2 直接进入 Round 3。
- 最终文档至少应达到 `READY WITH GAPS`。
- 若达到 `READY`，仍需保留真实的待确认项，不得为了“好看”全部删光。
- 最终定稿等级判定必须再对照 `finalization-standard.md`，不能只凭主观感觉收口。
- 若存在乱码、不可读内容或 frontmatter 状态缺失，不得进入最终收口。
