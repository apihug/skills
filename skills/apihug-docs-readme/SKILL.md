---
name: apihug-docs-readme
description: "Initialize or update docs/README.md as the documentation governance portal for single-module BMAD + DDD projects."
---

# apihug-docs-readme

轻量文档治理 skill。用于初始化或更新 `docs/README.md`，将其收敛为项目文档入口页。

## 主职责

- 生成或更新 `docs/README.md`
- 将 `docs/README.md` 收敛为单模块项目的文档治理入口

## 适用范围

本 skill 只适用于以下项目：

- 单体单模块项目
- 采用 BMAD 文档链
- 采用 DDD 规划链
- 已使用或计划使用 `project-context.md`、`prd.md`、`context-map.md`、`architecture.md`、`tactical-design.md`

## 非目标

本 skill 不负责：

- 生成或修改 `context-map.md`
- 生成或修改 `architecture.md`
- 生成或修改任一大域 `tactical-design.md`
- 生成多模块项目 README
- 定义具体包路径、目录树或领域关系细节

## 输出

- 主输出：`docs/README.md`

## 输入来源

执行时按需读取以下输入：

- `docs/project-context.md`
- `docs/planning-artifacts/prd.md`
- `docs/planning-artifacts/product-brief*.md`
- `docs/input-artifacts/product-brief*.md`
- `settings.gradle`

## 核心规则

- README 是文档入口页，不是战略设计、架构设计或战术设计正文。
- README 只总结正式基线文档及其职责边界，不重复定义领域边界或物理结构细节。
- 本 skill 只服务单模块项目，运行模块描述必须使用真实模块名。
- 若信息缺失且无法从现有文档稳定推断，可使用 `（待确定）` 占位。

## 骨架规则

- `template.md` 是固定骨架。
- `workflow.md` 定义输入发现、替换规则和验证规则。
- 不允许删除 README 的核心治理章节，除非后续模板本身被正式修改。

按照 `./workflow.md` 执行。
