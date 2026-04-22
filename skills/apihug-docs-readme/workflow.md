# apihug-docs-readme Workflow

生成或更新 `docs/README.md`，使其成为单模块 BMAD + DDD 项目的文档治理入口。

---

## Step 1: 收集项目信息

读取以下文件，存在则读取，不存在则跳过：

1. `docs/project-context.md`
2. `docs/planning-artifacts/prd.md`
3. `docs/planning-artifacts/product-brief*.md`
4. `docs/input-artifacts/product-brief*.md`
5. `settings.gradle`

从中提取：

- 项目名称：优先从 `project-context.md` 推断；若没有，再从 `settings.gradle` 推断
- 项目定位：从 PRD 或 Product Brief 提取
- 技术基线：从 `project-context.md` 提取
- 运行模块：从 `project-context.md` 提取真实单模块名称

如果以上信息都无法稳定获得，则向用户补问：

- 项目名称
- 唯一运行模块名称
- 一句话项目定位
- 技术栈关键字

## Step 2: 生成 README

使用 `./template.md` 作为固定骨架。

替换规则：

- `{project_name}` -> 实际项目名称
- `{project_positioning}` -> 项目定位
- `{tech_baseline}` -> 技术基线
- `{runtime_description}` -> 单模块运行形态描述
- `{runtime_module}` -> 唯一运行模块名

占位规则：

- 若某项信息无法从现有文档稳定推断，则使用 `（待确定）`
- 不允许为了填满 README 自行发明领域边界、包路径或架构决策

## Step 3: 固定骨架验证

确认 `docs/README.md` 存在且为合法 Markdown，并包含以下固定章节：

1. 项目基线
2. 文档分层
3. 正式基线文档
4. 推荐阅读顺序
5. 文档职责边界
6. 单模块项目约束
7. 使用规则

## Step 4: 内容边界验证

确认 README：

- 不包含 BC 名称列表
- 不包含具体包路径设计
- 不包含具体目录树设计
- 不包含某一大域的战术正文
- 不重复架构文档中的物理结构细节

README 应该只回答：

- 项目 docs 如何分层
- 哪些文档是正式基线
- 推荐按什么顺序阅读
- 各文档分别负责什么
- 单模块项目的文档治理约束是什么
