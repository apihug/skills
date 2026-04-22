# Context Map Finalization Standard

此文件定义 `context-map.md` 何时可以被判定为“定稿版”。
目的：
- 统一“可以继续往下做 architecture / tactical design / module design”的判定口径
- 防止因为会话中一时主观判断而过早收口

## 定稿等级

### Level A: Draft

特征：
- 已有初稿
- 但尚未完成结构检查或三轮审计

限制：
- 不应作为正式下游基线
- 只适合继续评审和修订
- frontmatter 应保持：`finalizationLevel: Draft`

### Level B: Ready With Gaps

特征：
- 已完成结构完整性检查
- 已完成 Source Alignment / DDD Consistency / Implementation Readiness 三轮审计
- 没有结构性错误
- 仍存在少量非阻塞待确认项

允许：
- 可作为 architecture / tactical design 的阶段性基线
- 允许继续下游工作，但要显式保留 gap 列表
- frontmatter 应至少反映：`finalizationLevel: Ready With Gaps`、`readyForArchitecture: true`

### Level C: Final Baseline

特征：
- 已完成结构完整性检查
- 已完成三轮审计
- 无结构性问题
- 无前后矛盾
- 待确认项仅剩非当前阶段决策所必需的事项，或为空
- 文档已可直接作为战略设计正式基线

允许：
- 可作为正式 `context-map.md`
- 可直接支撑后续 architecture / module structure / tactical design
- frontmatter 应至少反映：`finalizationLevel: Final Baseline`、`readyForArchitecture: true`、`projectContextSynced: true`

## 判定条件

## 1. 结构完整

必须同时满足：
- 固定 11 个章节完整存在
- 大域表、BC 映射表、关系表、边界规则、外部系统表、模块映射表齐全
- 图表与正文没有互相替代
- 审计记录已落文档
- frontmatter 状态字段完整
- 文档 UTF-8 正常可读、无乱码、无结构破损

若不满足：
- 最多只能是 `Draft`

## 2. Source 对齐完成

必须同时满足：
- source 输入表完整
- 锁定前提表完整
- 已明确哪些输入是锁定的
- 没把旧 `context-map.md` 当唯一真相
- 当前阶段范围、状态与 source 一致

若不满足：
- 不能高于 `Draft`

## 3. DDD 一致性通过

必须同时满足：
- 一级业务大域是真正的业务域，而非技术域
- BC 层级和大域层级清晰
- 关系类型使用正确
- 没混入聚合、实体、值对象等战术元素
- 没滥用 `Partnership`

若不满足：
- 不能高于 `Draft`

## 4. 实施就绪性通过

必须同时满足：
- 一级业务大域可映射为一级模块
- 依赖方向足以指导代码依赖控制
- 不存在会让 architecture / tactical design 返工的结构悬空项
- 面向模块划分的落地含义足够明确
- `docs/project-context.md` 已同步 architecture 上游约束

若不满足：
- 最多只能是 `Ready With Gaps`

## 5. 待确认项受控

可接受的待确认项：
- 不影响当前阶段模块划分
- 不影响当前阶段大域边界
- 不影响关键依赖方向

不可接受的待确认项：
- 一级业务大域是否成立
- 核心 BC 归属是否成立
- 关键关系类型是否成立
- source 支持的关键边界类别边界是否成立

若存在不可接受待确认项：
- 不能高于 `Draft`

## 输出要求

每次完成战略设计后，必须在最终说明中显式给出：
- 当前定稿等级：`Draft / Ready With Gaps / Final Baseline`
- 判定理由
- 剩余待确认项
- `docs/project-context.md` 是否已同步
- 主文 frontmatter 是否已同步
- 是否可继续：
  - architecture
  - module structure
  - tactical design

## 推荐默认标准

跨项目复用时，建议最低收口标准为：
- `Ready With Gaps`

若要作为公司内正式战略基线沉淀，建议标准为：
- `Final Baseline`
