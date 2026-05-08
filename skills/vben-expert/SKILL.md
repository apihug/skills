---
name: vben-expert
description: Use when implementing or refining APIHug Vben Admin frontend pages during story execution and you need Vben sub-mode routing for grids, forms, modal or drawer lifecycle, alert or confirm prompts, ellipsis or tooltip text handling, access control, theme or icon work, or playground-based UI analogs. Do not use for API or SDK calling rules; follow the ApiHug frontend Vben guide for those.
---

# Vben Expert

## Invocation Contract

This skill is the secondary router for APIHug frontend story execution.

- It MUST be invoked for all frontend tasks inside `apihug-dev-story`.
- It does not replace the ApiHug frontend guide.
- It routes the current task to the minimum Vben references needed for implementation.
- It supports one or more frontend sub-modes for the same task.

## Precedence

Always apply frontend decisions in this order:

1. `{project-root}/_bmad/apihug/rules/apihug-impl-front-vben-guide.md`
2. `vben-expert`
3. selected reference docs, demos, and playground files

If a generic Vben example conflicts with the ApiHug guide, follow the ApiHug guide.

## Scope

This skill is responsible for:

- Vben component behavior
- UI interaction patterns
- modal or drawer lifecycle choices
- search-form and table composition details
- lightweight prompt selection
- long-text display behavior
- access-control UI patterns
- picking the nearest safe playground analog

This skill is NOT responsible for:

- generated SDK usage
- route or menu generation contracts
- `RequestSchema` and `ResponseSchema` source-of-truth rules
- paging contract or zero-based page conversion rules
- any decision involving `@sdk/...`, `@hope/api`, `http.ts`, or `requestClient`

## Sub-Mode Routing

Determine one or more frontend sub-modes, then read only the matching references.

| Sub-Mode | Use When | Read |
|------|------|------|
| `LIST_GRID` | table, grid, columns, search area, tree table, custom cell, toolbar | `references/components/vben-vxe-table/docs.md` |
| `FORM_SCHEMA` | fields, rules, validation, dependencies, `valueFormat`, schema updates | `references/components/vben-form/docs.md` |
| `DETAIL_PRESENTATION` | read-only detail, overview block, long text in detail page | `references/components/vben-ellipsis-text/docs.md` when long text is involved |
| `MODAL_FLOW` | modal dialog lifecycle, submit locking, data injection | `references/components/vben-modal/docs.md` |
| `DRAWER_FLOW` | drawer lifecycle, side panel editing, state handoff | `references/components/vben-drawer/docs.md` |
| `ALERT_PROMPT` | lightweight alert, confirm, prompt, one-off interaction | `references/components/vben-alert/docs.md` |
| `ELLIPSIS_TEXT` | overflow text, tooltip-on-ellipsis, expand or collapse | `references/components/vben-ellipsis-text/docs.md` |
| `ACCESS_CONTROL` | route authority, button visibility, `v-access`, `AccessControl` | `references/common/access.md` |
| `THEME_ICON` | icon choice, visual consistency, theme polish | `references/common/icons.md`, `references/common/theme.md` |
| `PLAYGROUND_ANALOG` | need the closest business-module interaction analog | `references/playground/system-dept/`, `references/playground/system-menu/`, `references/playground/system-role/`, `references/playground/file-download/` |

## Output Contract

After invoking this skill, the agent must determine and keep explicit:

- the selected frontend sub-modes
- the exact references used for the current task
- the concrete Vben behavior points that must be applied
- any generic example pattern that was rejected because it conflicts with ApiHug rules

## Task Routing Notes

### `LIST_GRID`

Use `references/components/vben-vxe-table/docs.md` when you need:

- toolbar and table slots
- search-form behavior
- tree table configuration
- custom cell rendering
- grid API methods such as `query`, `reload`, `setGridOptions`, `toggleSearchForm`

### `FORM_SCHEMA`

Use `references/components/vben-form/docs.md` when you need:

- field-level validation rules
- dynamic show or hide behavior via `dependencies`
- `valueFormat` for payload transformation
- `formApi` operations such as `setValues`, `getValues`, `updateSchema`, `setFieldValue`

### `MODAL_FLOW` and `DRAWER_FLOW`

Use `references/components/vben-modal/docs.md` or `references/components/vben-drawer/docs.md` when you need:

- `connectedComponent`
- `setData(...).open()`
- `getData()` during open lifecycle
- submit locking via `lock()` and `unlock()`
- dynamic state updates through `setState(...)`

### `ALERT_PROMPT`

Use `references/components/vben-alert/docs.md` when you need:

- one-off `alert`, `confirm`, or `prompt` interactions
- lightweight confirmation without building a full modal component
- `useAlertContext()` inside custom prompt content

Prefer modal or drawer for complex forms or multi-step editing.

### `ELLIPSIS_TEXT` and `DETAIL_PRESENTATION`

Use `references/components/vben-ellipsis-text/docs.md` when you need:

- long-text truncation in tables or detail pages
- tooltip-on-ellipsis behavior
- click-to-expand text blocks

### `ACCESS_CONTROL`

Use `references/common/access.md` for:

- route authority
- button-level visibility
- `AccessControl`, `useAccess`, or `v-access`

### `THEME_ICON`

Use `references/common/icons.md` and `references/common/theme.md` for:

- icon selection
- visual consistency
- small UI polish within existing product language

### `PLAYGROUND_ANALOG`

Use playground files only as layout and interaction references:

- `references/playground/system-dept/`: tree table plus modal CRUD
- `references/playground/system-menu/`: drawer flow plus field linkage
- `references/playground/system-role/`: paged table plus permission tree
- `references/playground/file-download/`: download interaction patterns

## ApiHug Guardrails

- Do not copy `#/api/*`, `requestClient`, or generic request wrappers from Vben examples.
- Treat playground source code as UI composition reference, not as ApiHug backend integration reference.
- Prefer generated `RequestSchema` and `ResponseSchema` first, then patch fields or columns in page code.
- Keep list-page search forms in `formOptions`, not `gridOptions.formConfig`.
- Keep paging conversion, service imports, and route metadata aligned with the ApiHug frontend guide.
- Do not let UI access-control choices drift into backend authorization logic.

## Completion Gate

Before marking a frontend task complete, read and apply:

- `references/review-checklist.md`

Use the checklist with all current sub-modes and confirm the implementation still follows ApiHug frontend rules.
