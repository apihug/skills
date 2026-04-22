# Tactical Design Review Checklist

本文档定义 `tactical-design.md` 及其派生标准件的四轮审计清单。

## Round 0: Structure Completeness

目标：
- 确认主文和标准件的结构满足 output contract。

检查项：
1. `tactical-design.md` 是否包含 contract 规定的固定章节。
2. 主文 frontmatter 是否包含 `sources`、`reviewStatus`、`finalizationLevel`、`epicStoryReadiness`、`architectureIndexSynced`。
3. `01~06` 是否全部存在并使用对应模板结构。
4. `03-uml.md` 与 `06-physical-data-model.md` 是否保留 Mermaid 源，而不是要求渲染图。
5. 文档是否 UTF-8 正常可读、无乱码。

输出要求：
- 若结构不完整，先修结构，再进入 Round 1。

## Round 1: Source Alignment

目标：
- 确认主文忠实覆盖 PRD、context-map、architecture 与域内 source。

检查项：
1. 主文 `sources` 是否只记录本次真实采用的输入。
2. 是否与 `prd.md` 的 phase 范围一致。
3. 是否与 `context-map.md` 的大域定位、BC 归属和上下游边界一致。
4. 是否与 `architecture.md` 的模块边界和依赖方向一致。
5. 是否错误引用了其它大域的现成结论。
6. 若上游仅为 `Ready With Gaps`，是否继承并显式记录了未闭环项。

输出要求：
- 列出发现。
- 标记是否需要修正。
- 修改后回查本轮。

## Round 2: DDD Consistency

目标：
- 确认主文与标准件是战术设计，而不是零散的实现提示。

检查项：
1. 子域、聚合、实体、值对象划分是否清晰。
2. Application Service / Domain Service 职责是否越界。
3. 事件、命令、查询、规则是否能回溯到主文。
4. `01~06` 是否只做标准化展开，而非引入主文未声明的新事实。
5. `03/05/06` 是否与主文建模一致。
6. 是否错误重划了已锁定的大域或 BC。

输出要求：
- 列出结构性问题。
- 给出修正建议。
- 修正后回查本轮。

## Round 3: Implementation Readiness

目标：
- 确认该套战术文档足以支撑后续 Epic / Story 拆解与实现。

检查项：
1. 是否足以支撑 `Epic / Story` 拆解。
2. 是否存在直接阻断实现的结构性矛盾。
3. 是否仍有必须拍板而未标出的关键决策。
4. `architecture.md` 中的大域入口索引是否已同步。
5. `epicStoryReadiness` 是否与文档真实状态一致。

输出要求：
- 结论应明确为：
  - `BLOCKED`
  - `CONDITIONAL`
  - `READY`
- 结论词汇映射如下：
  - `BLOCKED` -> `epicStoryReadiness = Blocked`
  - `CONDITIONAL` -> `epicStoryReadiness = Conditional`
  - `READY` -> `epicStoryReadiness = Ready`
- 最终定稿等级仍必须再对照 `finalization-standard.md`。

## 审计收口规则

- 不允许跳过 Round 0。
- 若某轮发现结构性问题，必须先修文档，再重新执行该轮。
- 标准件发现主文缺口时，必须先回写主文，再重新同步标准件。
- 若存在乱码、不可读内容或 frontmatter 状态缺失，不得进入最终收口。
