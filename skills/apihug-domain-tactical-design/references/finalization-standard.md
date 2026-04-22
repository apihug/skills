# Domain Tactical Design Finalization Standard

本文件用于判定一个大域战术设计文档包当前处于什么成熟度等级。

## 等级定义

### 1. Draft

适用条件：

- `tactical-design.md` 已生成，但尚未完成 Structure Completeness + 三轮审查
- `01~06` 未全部生成，或已生成但未完成一致性检查
- 主文中仍存在较多待确认项
- 结构性矛盾尚未收敛

此时：

- 可继续讨论和审查
- 不建议直接进入 Epic / Story 拆解
- frontmatter 建议为：`finalizationLevel: Draft`，`epicStoryReadiness: Blocked`

### 2. Ready With Gaps

适用条件：

- `tactical-design.md` 已完成 Structure Completeness + 三轮审查
- `01~06` 已全部生成
- 关键边界、聚合、事件、规则、落库结论基本一致
- 仍有少量待确认项，但不阻塞后续 Epic / Story 拆解

此时：

- 可以进入 Epic / Story 拆解
- 可以作为当前大域的实施前基线
- 仍允许在后续审查中继续收尾
- `architecture.md` 中应已登记该大域路径与“可拆解但仍有待确认项”的状态
- frontmatter 建议为：`finalizationLevel: Ready With Gaps`，`epicStoryReadiness: Conditional`

### 3. Final Baseline

适用条件：

- 主文与 `01~06` 已全部收敛
- 不存在“主文未声明、标准件单独补出”的关键设计事实
- Structure Completeness + 三轮审查已完成，且无阻塞实现的结构性问题
- 待确认项仅剩后续扩展问题，不影响当前 phase
- `architecture.md` 中已登记该大域战术设计路径
- `architecture.md` 中已明确该大域可直接作为 Epic / Story / Implementation 的上游设计入口

此时：

- 可以作为正式大域战术设计基线
- 可供后续 Epic / Story / Implementation 直接引用
- frontmatter 建议为：`finalizationLevel: Final Baseline`，`epicStoryReadiness: Ready`

## 判定输出要求

每次宣布某个大域战术设计“已完成”时，应显式给出：

- 当前等级：`Draft / Ready With Gaps / Final Baseline`
- 判定理由
- 剩余待确认项
- `architecture.md` 是否已同步
- 主文 frontmatter 是否已同步
- 是否可继续做：
  - Epic / Story
  - Implementation
  - 后续大域扩展
