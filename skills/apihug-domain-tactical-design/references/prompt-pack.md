# Tactical Design Prompt Pack

本文档提供 `apihug-domain-tactical-design` 的稳定提示骨架。

## 初始建模提示词

目标：
- 按 `bmad-agent-architect` 的建模标准完成目标大域的主文建模。

提示骨架：
- 基于已读 source，为目标大域生成或重构 `tactical-design.md`。
- 先收敛：领域边界、子域、聚合、服务职责、事件主面、关键约束。
- 不要直接按模板填空，不要先写 `01~06`。
- 若 source 冲突，按 skill 规定的 source priority 收敛；无法收敛时，写入“待确认项”。
- 若上游产物仅为 `Ready With Gaps`，必须继承其未闭环项，不得静默当作已解决。

## Round 1: Source Alignment 审计提示词

目标：
- 检查主文是否忠实覆盖 PRD、context-map、architecture 与域内 source。

提示骨架：
- 逐条核对主文与 source 的一致性。
- 标出越界、遗漏、误引和上游 gap 继承缺失。
- 若发现问题，先提出主文修订点，不要直接修改标准件。

## Round 2: DDD Consistency 审计提示词

目标：
- 检查战术建模是否一致且可回溯。

提示骨架：
- 核对聚合、实体、值对象、服务职责、命令、事件、规则是否一致。
- 检查 `01~06` 是否引入主文未声明的新事实。
- 标出任何重划已锁定大域或 BC 的行为。

## Round 3: Implementation Readiness 审计提示词

目标：
- 判断文档是否已足以支撑后续 Epic / Story 拆解与实现。

提示骨架：
- 从后续实现角度检查阻断项。
- 判断 `epicStoryReadiness` 应为 `Blocked / Conditional / Ready`。
- 明确仍需人工拍板的事项。

## Finalization 提示词

目标：
- 对照 `finalization-standard.md` 做最终收口。

提示骨架：
- 明确当前定稿等级：`Draft / Ready With Gaps / Final Baseline`。
- 同步更新 frontmatter 状态字段。
- 确认 `architecture.md` 入口索引已同步。
- 列出仍保留的待确认项与下游影响。
