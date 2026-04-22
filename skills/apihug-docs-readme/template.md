# {project_name} 项目文档中心

本目录是 `{project_name}` 项目的统一文档入口，覆盖从需求输入到架构落地、再到领域战术设计与交付拆解的正式设计产物。

## 项目基线

- 项目定位：{project_positioning}
- 技术基线：{tech_baseline}
- 运行形态：{runtime_description}
- 设计链路：Product Brief -> PRD -> Strategic Design -> Architecture -> Tactical Design -> Epics -> Stories

## 文档分层

| 层级 | 目录 | 职责 | 说明 |
|------|------|------|------|
| 输入层 | `input-artifacts/` | 保存原始需求、调研、业务输入 | 事实来源，只读，不反向覆盖下游设计文档 |
| 规划层 | `planning-artifacts/` | 保存 PRD、战略设计、架构设计、战术设计、Epic/Story 规划 | 项目正式设计基线 |
| 实施层 | `implementation-artifacts/` | 保存单条 Story 的实施上下文与交付辅助文档 | 服务开发执行，不替代规划层基线 |

## 正式基线文档

| 文档 | 路径 | 职责 |
|------|------|------|
| Project Context | `project-context.md` | AI 执行规则、技术栈、项目级约束 |
| PRD | `planning-artifacts/prd.md` | 产品范围、FR/NFR、阶段目标 |
| Context Map | `planning-artifacts/context-map.md` | 一级业务大域、BC 映射、上下文关系、边界规则 |
| Architecture | `planning-artifacts/architecture.md` | 单模块物理结构、技术选型、依赖约束、落地规则 |
| Tactical Design | `planning-artifacts/domains/<domain>/tactical-design.md` | 某一一级业务大域的战术设计主文与标准件来源 |
| Epics & Stories | `planning-artifacts/epics.md` | 交付拆解、Epic/Story 基线 |

## 推荐阅读顺序

1. `project-context.md`：先理解项目级 AI 约束、技术栈和 Contract-First 规则。
2. `planning-artifacts/prd.md`：确认产品范围、优先级和阶段目标。
3. `planning-artifacts/context-map.md`：理解战略边界、一级业务大域、BC 关系。
4. `planning-artifacts/architecture.md`：理解单模块物理结构与依赖治理。
5. `planning-artifacts/domains/<domain>/tactical-design.md`：阅读目标大域的战术设计。
6. `planning-artifacts/epics.md`：进入 Epic/Story 拆解与实施规划。
7. `implementation-artifacts/<story>.md`：实施具体 Story 时再读取对应交付文档。

## 文档职责边界

| 文档 | 管什么 | 不管什么 |
|------|------|------|
| `project-context.md` | AI 规则、技术栈、项目级实现约束 | 业务边界划分、领域关系设计 |
| `context-map.md` | 一级业务大域、BC 归属、关系类型、边界规则、面向架构的战略约束 | 具体目录树、包结构、依赖矩阵实现细节 |
| `architecture.md` | 单模块物理结构、域包组织、技术选型、依赖治理、ArchUnit 规则 | 重划 BC、重定义战略边界 |
| `tactical-design.md` | 域内聚合、实体、值对象、领域服务、领域事件、域内约束 | 跨域关系主定义、物理模块拓扑主定义 |

## 单模块项目约束

- `{runtime_module}` 是唯一后端运行模块，所有领域代码都在该模块内按域包承载。
- `context-map.md` 定义逻辑边界，不直接决定物理目录结构。
- `architecture.md` 决定单模块内如何以域包方式承载这些逻辑边界。
- `domains/` 按一级业务大域组织，不按平铺 Bounded Context 组织。

## 使用规则

- `input-artifacts/` 中的文档是事实输入，不直接冒充正式设计结论。
- `implementation-artifacts/` 不得重新定义产品范围、领域边界或架构规则。
- Story 文档服务实施，但必须显式引用其上游 PRD、战略、架构、战术设计来源。
- 若 `context-map.md` 已存在，后续架构设计必须将其视为正式上游输入。
