---
name: apihug-impl-front-vben-guide
description: Canonical frontend page-generation rules for APIHug Vben Admin apps.
version: 3.0.0
author: APIHug Team / H.O.P.E. Infra
updated: 2026-05-09
audience: code agents and frontend developers designing and implementing APIHug pages
---

# APIHug Frontend Vben Guide

## 1. Purpose And Authority

This guide is the single source of truth for APIHug frontend page generation.

It is written for code agents and frontend implementers who need to:

- design a page before writing code
- choose an approved page pattern
- verify backend, i18n, and shared-infrastructure contracts
- implement the page without inventing local conventions
- validate the result before marking the task complete

This guide owns frontend policy and contract decisions. If another guide, skill, example, scaffold, or playground file conflicts with this document, follow this document.

## 2. How Code Agents Must Use This Guide

Use this guide in this order:

1. Load this guide before designing or implementing any frontend task.
2. Choose the page pattern from `## 8. Page Pattern Decision`.
3. Run the pre-write checklist in `## 13. Agent Checklists`.
4. If the page pattern is allowed and all contracts exist, invoke skill `vben-expert`.
5. Read only the Vben references routed by `vben-expert`.
6. Implement the page.
7. Run the final checklist in `## 13. Agent Checklists`.
8. Reject the implementation if any item in `## 12. Failure Modes` appears.

Skill `vben-expert` is a routing layer. It does not override this guide.

## 3. Scope And Boundaries

This guide applies to generated business pages under:

```text
apps/{app}/src/views/**
```

It governs:

- page layout and approved primitives
- generated SDK and schema usage
- route metadata expectations
- i18n completeness
- sort, reference, hierarchy, and business-code contracts
- failure conditions that require halting instead of improvising

It does not replace component-level Vben docs. Those are routed internally by skill `vben-expert` and are read only after this guide has approved the pattern.

## 4. Golden Rules

- Edit app code only under `apps/{app}/src/**`.
- Do not manually edit generated route modules under `apps/{app}/src/router/routes/modules/*.ts`.
- Do not manually edit generated menu files under `apps/{app}/src/menus/**`.
- Do not manually edit generated SDK files under `packages/@sdk/{domain}-{module}/src/**`.
- Do not use `fetch`, `axios`, or `requestClient` in feature code. Use generated services only.
- Do not call `configureApiClient` in pages or components.
- Use `#/adapter/form` and `#/adapter/vxe-table` as the local Vben wrappers.
- Use `RequestSchema.*` for forms and search forms.
- Use `ResponseSchema.*` for table columns.
- `PageRequest.page` is zero-based. VXE `page.currentPage` is one-based. Convert exactly once with `page.currentPage - 1`.
- Treat missing backend, i18n, or shared-infrastructure contracts as stop conditions, not as invitations to improvise page-local workarounds.

## 5. Generated Output And Routing Contracts

### 5.1 What is generated

Running `gradlew :{ui_module}:ui` generates the important frontend outputs:

- `apps/{app}/src/router/routes/modules/*.ts`
- `apps/{app}/src/menus/index.ts`
- `apps/{app}/src/menus/menu.json`
- `apps/{app}/src/locales/langs/{lang}/page.json`
- `packages/@sdk/{domain}-{module}/src/api/*.ts`
- `packages/@sdk/{domain}-{module}/src/types/*.ts`
- `packages/@sdk/{domain}-{module}/src/model/*.ts`
- `packages/@sdk/{domain}-{module}/src/schema/request/*.ts`
- `packages/@sdk/{domain}-{module}/src/schema/response/*.ts`
- `packages/@sdk/{domain}-{module}/src/schema/index.ts`

Generated outputs are read-only from page code.

### 5.2 Page discovery and route metadata

Pages are discovered from:

```text
apps/{app}/src/views/**/*.vue
```

Every routable page must contain:

```ts
defineOptions({
  name: 'StablePageName',
  meta: {
    title: 'Static Plain Title',
  },
});
```

Required routing rules:

- `defineOptions({ name, meta })` is mandatory for routable pages.
- `meta.title` must be a static string literal.
- Do not use `$t(...)`, computed expressions, or runtime values in `meta.title`.
- Give every page a stable unique `name`.
- `hidden: true` is treated as `hideInMenu: true`.
- `meta.order` controls route and menu ordering.
- `meta.path` may override the default path when needed.

### 5.3 Path mapping contract

- `views/system/index.vue` -> `/system`
- `views/system/role.vue` -> `/system/role`
- `views/system/user/[id].vue` -> `/system/user/:id`
- `views/system/user/[[id]].vue` -> `/system/user/:id?`

### 5.4 Route, menu, and page-i18n behavior

Current generator behavior:

- route modules, menus, and page i18n are generated from the same discovered page tree
- routes are grouped by the first path segment
- generated route and menu titles use `$t(i18key)`
- `page.json` stores the plain text title authored in `defineOptions`

Dynamic segments are part of the i18key contract and must not collapse onto the list-page key.

Examples:

- `/operator/knowledge` -> `page.operator.knowledge.title`
- `/operator/knowledge/:id` -> `page.operator.knowledge.id.title`
- `/system/user/:id` -> `page.system.user.id.title`

### 5.5 Locale merge rules for generated page titles

When generating `apps/{app}/src/locales/langs/{lang}/page.json`:

- recurse through all generated children, not only top-level route groups
- preserve existing translated values
- add placeholders only for newly discovered routes
- keep existing root group titles stable

## 6. SDK, HTTP, And Schema Contracts

### 6.1 SDK and HTTP ownership

Feature code must call generated services only.

Good:

```ts
import { RoleService } from '@sdk/{domain}-{module}/api';

await RoleService.createRole(values);
```

Bad:

```ts
import { requestClient } from '#/api/request';

await requestClient.post('/api/roles/roles', values);
```

`packages/@sdk/{domain}-{module}/src/http.ts` is internal SDK glue. Do not import it directly in feature code.

### 6.2 Paging and grid return shape

Current paging contract:

```ts
interface PageRequest {
  page: number; // zero-based
  size: number;
  sort?: string[];
}
```

For standard ApiHug apps, the VXE adapter expects:

- `list: data`
- `total: totalCount`

Do not override `proxyConfig.response` in page code. Fix the shared adapter if the envelope is wrong.

### 6.3 Generated schema contract

Generated schema is intentionally neutral.

Use:

- `toVbenFormSchema(...)` from `@hope/api-antd-adapter`
- `toVxeTableColumns(...)` from `@hope/api-antd-adapter`

Prefer the local wrappers:

- `useVbenForm` from `#/adapter/form`
- `useVbenVxeGrid` from `#/adapter/vxe-table`

Do not edit generated schema files to satisfy one page. Build from schema first, then patch in page code by `fieldName` or `field`.

## 7. Shared Infrastructure And Approved Primitives

Before generating the first business page in an app, verify the shared layer has the required adapters and renderers. If it does not, add the shared capability first instead of compensating with page-local hacks.

### 7.1 Required shared infrastructure

Required local adapters:

- `#/adapter/form` exports `useVbenForm`
- `#/adapter/vxe-table` exports `useVbenVxeGrid`
- `#/adapter/schema` or equivalent exports schema adapter context and field/column selection helpers

Required shared renderers and helpers:

- `CellOperation` for standard row actions and destructive-action confirmation
- `CellTag` for enum/status display
- `CellSwitch` for confirmed state toggles

Register shared renderers in the adapter layer, not inside page files.

### 7.2 Approved primitives

Use these primitives by default:

| Concern | Approved primitives |
| --- | --- |
| Page shell | `Page` from `@vben/common-ui` |
| Split layout | `ColPage` from `@vben/common-ui` |
| List and tree tables | `useVbenVxeGrid` from `#/adapter/vxe-table` |
| Search and standard forms | `useVbenForm` from `#/adapter/form` |
| Form mutation surfaces | `useVbenDrawer`, `useVbenModal` |
| Read-only trace/detail drawer | `Drawer` from `antdv-next` when no submit lifecycle is needed |
| Tabs | `Tabs` from `antdv-next` |
| Steps | `Steps` from `antdv-next` |
| Feedback | `message`, `Modal.confirm`, `Alert`, `Result`, `Spin`, `Empty`, `Fallback` |
| Monitoring widgets | Vben dashboard components or `Row`, `Col`, `Card`, `Statistic`, `Tabs`, `Spin` |

Use raw `Table` only for the embedded read-only summary-table exception.

Use raw `Form` only for approved complex-configuration or diagnostic/test surfaces when schema-driven `useVbenForm` cannot express the control structure cleanly.

## 8. Page Pattern Decision

Choose the page pattern before writing code.

| Task shape | Page pattern | Primary building blocks |
| --- | --- | --- |
| Query, filter, maintain rows | List workbench | `Page`, `useVbenVxeGrid`, `formOptions`, row actions, form drawer or modal |
| Parent-child entity maintenance | Tree table workbench | `Page`, `useVbenVxeGrid`, `treeConfig`, shared tree title renderer |
| Entity reference selection in a form | Selector or lookup field | `Select`, `TreeSelect`, controlled slot, or lookup modal |
| Simple create/edit | Form modal | `useVbenModal`, `useVbenForm` |
| Medium or complex create/edit | Form drawer | `useVbenDrawer`, `useVbenForm` |
| Multi-section dynamic configuration | Complex configuration form | drawer or modal shell, `Tabs` or menu, schema form where possible |
| Master-detail configuration | Split configuration page | `ColPage`, panel components, tabs when justified |
| Audit/log trace | Audit list or trace drawer | `Page`, grid, detail drawer |
| Permission tree assignment | Form drawer with tree field | `useVbenDrawer`, `useVbenForm`, controlled tree slot |
| Monitoring dashboard | Dashboard overview | `Page`, `Row`, `Col`, `Card`, `Statistic`, `Tabs`, `Spin` |
| Read-only entity detail | Detail page | `Page`, `Spin`, `Descriptions`, `Tag`, optional tabs |
| Trace or audit chain | Trace drawer | `Drawer`, `Spin`, `Timeline`, `Descriptions` |
| Read-only lookup in modal | Lookup modal | `useVbenModal`, `Spin`, `Descriptions`, `Empty`, optional embedded read-only table |
| Upload/import | Upload panel | `Upload.Dragger`, `message`, grid for uploaded records with actions |
| Diagnostic/test panel | Diagnostic workbench | `Page`, `useVbenForm` by default, `Spin`, `Empty`, `Result` |

After choosing a pattern, use skill `vben-expert` to route to the matching reference docs.

## 9. Design And Implementation Rules

These rules apply while designing and implementing pages.

1. Author pages only under `apps/{app}/src/views/**`, with page-local components under that page's `components/` folder.
2. Do not edit generated route, menu, SDK, or schema files.
3. List pages use `Page` and `useVbenVxeGrid`.
4. Search forms use `formOptions.schema` derived from `RequestSchema.*`.
5. Table columns start from `toVxeTableColumns(ResponseSchema.*)`.
6. Standard create/edit forms start from `toVbenFormSchema(RequestSchema.*)`.
7. Backend calls use generated SDK services or a thin app-local adapter that delegates to them.
8. Do not import `requestClient`, `fetch`, or `axios`.
9. Convert `PageRequest.page` exactly once at the service-call boundary.
10. Match schema and column patches by `fieldName` or `field`, never by array index.
11. Use shared action/status/tree helpers before inventing page-local markup.
12. Use `formOptions`, not `gridOptions.formConfig`.
13. Do not override VXE `proxyConfig.response` in a page.
14. Use `Upload` from `antdv-next` for upload/import workflows.
15. Use `Tabs` only when the page has real sibling views or modes.
16. Use `Steps` only for staged workflows.
17. Use `ColPage` or another existing layout primitive for split pages, not page-local flex shells.
18. Use confirmation before high-risk mutation and structured result feedback when the business impact is material.
19. Treat missing shared infrastructure as a shared-layer task, not as a reason to write page-local substitutes.
20. When a pattern repeats across more than one page, extract the helper into the shared adapter or component layer.

## 10. Internationalization And Text Rules

Missing translations are a generation failure, not a cosmetic issue.

### 10.1 Locale coverage

- Discover supported locales from `apps/{app}/src/locales/langs/**`.
- Every page-level key used by the page must exist in every supported locale.
- Every selected `RequestSchema.*` field and `ResponseSchema.*` column must have a non-empty locale value for its `i18key` in every supported locale.
- Generated English `title` values are diagnostics only. They are not acceptable production UI text.
- Empty-string locale values count as missing.

### 10.2 Visible text

Use `$t(...)` or schema-adapter i18n for:

- page titles rendered in templates
- table titles and raw table column titles
- form labels
- button text
- modal and drawer titles
- confirmation text
- message text
- alert and result text
- tab and step labels
- upload/import hints
- descriptions labels

Do not hard-code visible English or Chinese text in page code except for static `defineOptions().meta.title`, which remains a route-generator literal.

Do not use `tFallback(...)` or equivalent fallback helpers in newly generated page code.

## 11. Domain Contracts For Sort, References, Hierarchy, And Business Codes

### 11.1 Sort contract

Sortable UI is a backend contract, not a visual guess.

- Only enable sortable columns when the backend operation supports sort input and the response field maps to a real sortable backend field.
- For pageable grids, sortable columns must use remote sorting and pass `PageRequest.sort`.
- Clearing sort must omit `PageRequest.sort`.
- For non-pageable list or tree endpoints, disable sortable controls unless there is an explicit sort request contract.
- Do not accept adapter-inferred numeric sorting without contract verification.

### 11.2 Reference-field contract

Reference fields are not generic scalar inputs.

- Do not render user-facing entity references as raw `Input`, `InputNumber`, or typed IDs.
- Use backend-backed selectors, tree selectors, or lookup modals.
- Selector labels must show human meaning, not raw IDs only.
- If the backend has no lookup/search/tree API for the field, stop frontend generation and report the missing contract.

### 11.3 Hierarchy contract

Hierarchy maintenance requires hierarchy-aware UI.

- If the response has `id` plus `parentId` or equivalent hierarchy fields, use a tree table unless the story explicitly calls for a flat audit/search/reporting view.
- Tree tables must use a non-pageable tree/list SDK method that returns the full hierarchy or full filtered hierarchy.
- A left-side tree filter does not replace a required hierarchy-aware maintenance grid on the right side.
- If creation under a parent is supported, expose a row-level create-child or append action.
- Do not ask users to type a parent ID.

### 11.4 Tenant-scoped business-code contract

Business code fields such as `organizationCode`, `departmentCode`, `roleCode`, or other tenant-unique `*Code` identifiers are backend-managed unless the story explicitly defines an import or batch-matching exception.

- Do not make backend-managed business-code fields editable in normal create/edit pages.
- Display them read-only in lists, details, import previews, and audit output.
- If create or update schema still requires a non-empty backend-managed business code, stop frontend generation and report the backend contract defect.
- Do not hide the field and submit empty, placeholder, random, or copied values.
- This rule does not apply to enum or discriminator fields such as `statusCode` or `typeCode`.

## 12. Styling Rules And Failure Modes

### 12.1 Styling rules

Generated pages should be style-light.

Allowed:

- `Page auto-content-height`
- `Page` title, description, extra, and footer slots
- `ColPage` for split layouts
- Vben form layout props
- VXE `height: 'auto'`
- `Row`, `Col`, `Card`, and `Statistic` for monitoring widgets
- small utility sizing classes when already normal in the app

Forbidden in generated pages:

- `<style scoped>`
- page-level `lang="scss"`
- `:deep(...)`
- `.ant-*` internal selector overrides
- raw CSS to control Vben or Ant internals
- `ant-design-vue` imports
- introducing another UI, chart, upload, table, modal, or form library

### 12.2 Failure modes

Reject generated frontend code if any of these appear:

- manual edits to router, menu, SDK, or generated schema output
- direct use of `requestClient`, `fetch`, or `axios`
- hard-coded visible text outside static `defineOptions().meta.title`
- `tFallback(...)` or equivalent fallback helper in new page code
- missing or empty locale entries for any selected page/schema key
- raw `Table` used for search, pagination, row actions, mutation, or selection
- raw `Form` used outside approved complex or diagnostic surfaces
- search forms built in templates or under `gridOptions.formConfig`
- page-level `proxyConfig.response` override
- sortable headers without a verified backend sort contract
- pageable grid sort UI without `PageRequest.sort`
- typed-ID reference fields instead of selectors or lookup modals
- hierarchy maintenance implemented as a flat pageable table without explicit story justification
- hierarchy maintenance backed only by pageable search data
- no row-level create-child action where child creation under a selected parent is supported
- editable backend-managed business-code fields
- continuing page generation after detecting a required non-empty backend-managed business code input
- page-local style blocks or deep overrides
- ad hoc action buttons or status rendering where shared helpers already exist
- high-risk mutations without confirmation
- dashboard/detail/trace/result states built with custom CSS instead of approved primitives

## 13. Agent Checklists

### 13.1 Before writing code

- Verify shared infrastructure exists: form adapter, VXE adapter, schema helpers, and shared renderers.
- Discover supported locale files.
- Choose the page pattern from `## 8. Page Pattern Decision`.
- Locate the generated service methods.
- Locate matching `RequestSchema` and `ResponseSchema`.
- Verify every selected schema item has a non-empty locale value in every supported locale.
- Verify the backend sort contract for every sortable column you plan to expose.
- Identify every reference field and the backend lookup/search/tree method that feeds it.
- Identify hierarchy fields and verify a non-pageable tree/list SDK method exists when hierarchy maintenance is required.
- Identify backend-managed business-code fields and confirm frontend input is not required.
- Plan the page-level locale keys for toolbar actions, titles, confirmation text, messages, results, tabs, steps, and raw table columns.
- Invoke skill `vben-expert` and read only the routed references.

### 13.2 Before marking the task complete

- No failure mode from `## 12.2` appears.
- Route metadata is static and route-generator compatible.
- Search uses `formOptions`.
- Forms and tables still start from generated schema.
- Visible text is backed by `$t(...)` or schema-adapter i18n.
- Sort UI is backed by a real backend sort contract.
- Reference fields use selectors, tree selectors, or lookup modals.
- Hierarchy maintenance preserves the hierarchy in the table and uses a non-pageable tree/list call.
- Create/edit forms omit backend-managed business-code fields.
- Shared action/status helpers are used where available.
- Delete, status, and other high-risk actions confirm before mutation.
- The implementation also passes the `vben-expert` review checklist.

## 14. Routed Vben References

After this guide has approved the page pattern, use skill `vben-expert` to route to the minimum internal Vben references needed for implementation.

Those routed references are implementation support only. They do not replace this guide.
