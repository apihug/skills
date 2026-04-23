---
name: apihug-impl-front-vben-guide
description: Current frontend development rules for APIHug Vben UI modules.
version: 2.2.0
author: APIHug Team / H.O.P.E. Infra
updated: 2026-04-23
audience: code agents and frontend developers
---

# APIHug Frontend Vben Guide

## 1. Purpose

This guide is the current rule set for developing pages inside an APIHug UI module.

It describes the real contract between:

- the auto router generator
- the generated SDK package `@sdk/{domain}-{module}`
- the pre-installed shared module `@hope/api`
- the pre-installed shared module `@hope/api-antd-adapter`
- the app wrappers `#/adapter/form` and `#/adapter/vxe-table`

If an older example conflicts with this guide, follow this guide.

## 2. Golden Rules

- Edit app code under `apps/{app}/src/**`.
- Do not manually edit generated route modules under `apps/{app}/src/router/routes/modules/*.ts`.
- Do not manually edit generated SDK files under `packages/@sdk/{domain}-{module}/src/api`, `src/types`, `src/model`, or `src/schema`.
- Do not use `fetch`, `axios`, or `requestClient` in feature code. Use generated services only.
- Do not call `configureApiClient` in pages or components. App bootstrap already does that.
- Do not modify any code in `packages/@sdk` (these are generated files).
- In `admin-center`, use `#/adapter/form` and `#/adapter/vxe-table`.
- For grid search forms, use `formOptions`. Do not use VXE `formConfig`.
- For generated schema:
  - `RequestSchema.*` is for forms and search forms.
  - `ResponseSchema.*` is for table columns.
- `PageRequest.page` is zero-based. VXE `page.currentPage` is one-based. Always subtract `1`.

## 3. What Is Generated

### 3.1 Pre-installed vs generated

These modules are pre-installed shared infrastructure and are not generated per app:

- `@hope/api`
- `@hope/api-antd-adapter`

The generated business SDK is different:

- package import name: `@sdk/{domain}-{module}`
- filesystem path: `packages/@sdk/{domain}-{module}`

Running `gradlew :{ui_module}:ui` generates these important outputs:

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

Meaning:

- `router/routes/modules/`: generated route modules, grouped by first path segment
- `menus/index.ts`: generated frontend menu tree using `$t(i18key)`
- `menus/menu.json`: generated raw menu payload for external/backend usage
- `locales/langs/{lang}/page.json`: generated page i18n namespace
- `api/`: generated service methods
- `types/`: DTO types
- `model/`: enums, authorities, errors
- `schema/request/`: neutral `RequestItem[]`
- `schema/response/`: neutral `ResponseItem[]`
- `schema/index.ts`: namespace exports `RequestSchema` and `ResponseSchema`

## 4. Routing Rules

### 4.1 Source of truth

Pages are discovered from:

```text
apps/{app}/src/views/**/*.vue
```

A file is picked up by the router generator only if it contains:

```ts
defineOptions({ meta: { ... } })
```

If `meta` is missing, no route is generated for that file.

The generator treats page `defineOptions({ name, meta })` as the single source of truth for:

- route path and route meta
- menu title/i18key
- page locale placeholders under `page.json`

### 4.2 Path mapping

- `views/system/index.vue` -> `/system`
- `views/system/role.vue` -> `/system/role`
- `views/system/user/[id].vue` -> `/system/user/:id`
- `views/system/user/[[id]].vue` -> `/system/user/:id?`

Routes are grouped by the first path segment and written to:

```text
apps/{app}/src/router/routes/modules/{segment}.ts
```

Do not maintain those files by hand.

### 4.3 Required page pattern

Use `defineOptions` and give every page a unique `name`.

Recommended minimum:

```vue
<script setup lang="ts">
defineOptions({
  name: 'RoleList',
  meta: {
    title: 'Role Management',
    icon: 'lucide:shield',
    order: 10,
  },
});
</script>
```

Notes:

- `meta.title` should always be set.
- `meta.i18key` is optional. If omitted, the generator derives one from the page path.
- `meta.order` controls route/menu ordering.
- `meta.path` can override the default path when needed.
- `hidden: true` is treated as `hideInMenu: true`.
- `meta.title` must be a static string literal. Do not use `$t(...)`, computed values, or runtime expressions in `defineOptions`.
- Give every page a stable unique `name`. Do not reuse the same `name` across pages.

Good:

```vue
<script setup lang="ts">
defineOptions({
  name: 'KnowledgeDetail',
  meta: {
    path: '/operator/knowledge/:id',
    title: 'Knowledge Detail',
    hideInMenu: true,
  },
});
</script>
```

Bad:

```vue
<script setup lang="ts">
defineOptions({
  name: 'KnowledgeDetail',
  meta: {
    path: '/operator/knowledge/:id',
    title: $t('page.operator.knowledge.id.title'),
  },
});
</script>
```

### 4.4 Current route/menu/i18n contract

Current generator behavior is:

- route modules, menu modules, and page i18n are generated from the same discovered page tree
- top-level routes are grouped by the first path segment such as `system`, `operator`, `monitoring`
- menu ordering follows `meta.order`
- menu visibility follows `hideInMenu` / `hidden`
- route `meta.title` in generated files is always emitted as `$t(i18key)`
- `page.json` keeps the actual plain title text

That means the page author writes plain titles in `defineOptions`, while generated outputs transform them into:

- `router`: `meta.title: $t('page.xxx.title')`
- `menu`: `$t('page.xxx.title')`
- `i18n`: `"page.xxx.title": "Plain Title"`

### 4.5 Dynamic route i18key rules

Dynamic segments are part of the i18key contract and must not collapse onto the list page key.

Examples:

- `/operator/knowledge` -> `page.operator.knowledge.title`
- `/operator/knowledge/:id` -> `page.operator.knowledge.id.title`
- `/system/user/:id` -> `page.system.user.id.title`

Do not assume `/foo` and `/foo/:id` can share the same i18key.

### 4.6 Locale merge rules

When generating `apps/{app}/src/locales/langs/{lang}/page.json`:

- recurse through all generated children, not only the top-level group nodes
- preserve existing translated values when the key already exists
- only add missing placeholders for newly discovered routes
- keep root group titles stable; do not overwrite them with `null`

This is important because the generator must be safe to run repeatedly in a real project without destroying hand-edited translations.

## 5. HTTP And SDK Rules

### 5.1 Do not call the HTTP client directly

The app should configure `@hope/api` during bootstrap before feature pages call generated services.

Feature code should only call generated services.

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

### 5.2 What `http.ts` is

Each generated SDK package has `src/http.ts`.

It is an internal bridge that re-exports helpers from `@hope/api`.
Generated service files import `../http` so the SDK package stays self-contained.

Feature code should not import `src/http.ts` directly.

### 5.3 Current paging contract

`@hope/api` uses:

```ts
interface PageRequest {
  page: number; // zero-based
  size: number;
}

interface PageableResult<T> {
  pageIndex: number;
  pageSize: number;
  totalCount: number;
  totalPage: number;
  data: T[];
}
```

Generated paged services now correctly accept both request body and page request:

```ts
RoleService.searchRoles(filters, {
  page: currentPage - 1,
  size: pageSize,
});
```

### 5.4 Grid return shape

In the default VXE adapter mapping used by this guide, proxy response fields are:

- `list: data`
- `total: totalCount`

That means a grid query can return raw `PageableResult<T>` from the generated service.

Do not reshape it to `items/total` unless your local adapter explicitly uses a different mapping.

## 6. Generated Schema Rules

The generated schema is intentionally neutral.

Do not try to hand-write Vben-only fields into generated SDK files.

Current rule:

- `RequestSchema.*` -> form schema and search form schema
- `ResponseSchema.*` -> table columns

Use:

- `toVbenFormSchema(...)` from `@hope/api-antd-adapter`
- `toVxeTableColumns(...)` from `@hope/api-antd-adapter`

Prefer app-local wrappers for Vben integration:

- `useVbenForm` from your local form adapter wrapper
- `useVbenVxeGrid` from your local VXE grid adapter wrapper

Do not use `toAntdTableColumns(...)` for VXE table pages.

Current customization surface:

- schema hints from backend/generated schema:
  - `ui.widget`
  - `ui.props`
- adapter context:
  - `t`
  - `enumLabelPolicy`
  - `objectMode`
  - `formatDate`
- page-level post-processing after `toVbenFormSchema(...)` or `toVxeTableColumns(...)`

`@hope/api-antd-adapter` does not currently expose a field-specific callback hook such as `onBuildField`, `onBuildColumn`, or `fieldOverrides`.

## 7. Current Form And Table Mapping

### 7.1 Request schema -> Vben form

`toVbenFormSchema(RequestSchema.Xxx)` builds Vben-compatible form items using:

- `fieldName`
- `rules`
- `valueFormat`
- current app component names
- `ui.widget` when provided by schema
- merged `ui.props` into `componentProps`

Current built-in component names expected by the default Vben form mapping:

- `Input`
- `InputNumber`
- `Select`
- `Switch`
- `Textarea`
- `Upload`
- `DatePicker`
- `RangePicker`
- `TimePicker`

### 7.2 Response schema -> VXE columns

`toVxeTableColumns(ResponseSchema.Xxx)` builds VXE-compatible columns using:

- `field`
- `title`
- `sortable`
- `formatter`
- merged `ui.props` into the final column

### 7.3 Search form placement

For VXE grid pages, the search form belongs in `formOptions`, not in `gridOptions.formConfig`.

## 8. Standard Development Pattern

Replace `@sdk/{domain}-{module}` in the examples below with the actual generated SDK package in your UI module.

### 8.1 Normal form page or modal

Use request schema to build the form, then submit with the generated service.

```ts
import { reactive } from 'vue';

import { useVbenForm } from '<local-form-adapter>';
import { toVbenFormSchema } from '@hope/api-antd-adapter';
import { RoleService } from '@sdk/{domain}-{module}/api';
import { RequestSchema } from '@sdk/{domain}-{module}/schema';

const [Form, formApi] = useVbenForm(
  reactive({
    schema: toVbenFormSchema(RequestSchema.CreateRoleRequest),
    showDefaultActions: false,
  }),
);

async function handleSubmit() {
  const { valid } = await formApi.validate();
  if (!valid) {
    return;
  }
  await RoleService.createRole(await formApi.getValues());
}
```

### 8.2 List page with search form and table

Use request schema for search, response schema for columns, and return raw `PageableResult<T>` from the service.

```ts
import { useVbenVxeGrid } from '<local-vxe-grid-adapter>';
import { toVbenFormSchema, toVxeTableColumns } from '@hope/api-antd-adapter';
import { RoleService } from '@sdk/{domain}-{module}/api';
import { RequestSchema, ResponseSchema } from '@sdk/{domain}-{module}/schema';

const [Grid, gridApi] = useVbenVxeGrid({
  formOptions: {
    collapsed: false,
    schema: toVbenFormSchema(RequestSchema.SearchRolesRequest),
  },
  gridOptions: {
    columns: [
      { type: 'seq', width: 60 },
      ...toVxeTableColumns(ResponseSchema.RoleSummary),
      {
        field: 'action',
        fixed: 'right',
        slots: { default: 'action' },
        width: 160,
      },
    ],
    pagerConfig: {},
    proxyConfig: {
      ajax: {
        query: async ({ page }, formValues) =>
          await RoleService.searchRoles(formValues, {
            page: page.currentPage - 1,
            size: page.pageSize,
          }),
      },
    },
    toolbarConfig: {
      search: true,
    },
  },
});
```

### 8.3 Page-specific customization

Start from adapter output, then patch it in page code.
Do not edit generated schema files to satisfy one page.
To customize one field or one column, generate first, then patch the result in page code.

Good:

```ts
const searchSchema = toVbenFormSchema(RequestSchema.SearchRolesRequest).map(
  (item) =>
    item.fieldName === 'keyword'
      ? {
          ...item,
          componentProps: {
            ...item.componentProps,
            placeholder: 'Search role code or name',
          },
        }
      : item,
);
```

```ts
const formSchema = toVbenFormSchema(RequestSchema.CreateRoleRequest).map((item) =>
  item.fieldName === 'status'
    ? {
        ...item,
        componentProps: {
          ...item.componentProps,
          onChange: (value: string) => console.log(value),
        },
      }
    : item,
);
```

```ts
const columns = toVxeTableColumns(ResponseSchema.RoleSummary).map((col) =>
  col.field === 'status'
    ? {
        ...col,
        formatter: ({ cellValue }) => `Status: ${cellValue ?? ''}`,
      }
    : col,
);
```

```ts
const gridColumns = [
  ...toVxeTableColumns(ResponseSchema.RoleSummary),
  {
    field: 'action',
    title: 'Action',
    width: 160,
    slots: { default: 'action' },
  },
];
```

Bad:

- editing `packages/@sdk/{domain}-{module}/src/schema/request/*.ts`
- editing `packages/@sdk/{domain}-{module}/src/schema/response/*.ts`
- duplicating DTO fields manually in page code

## 9. Optional Adapter Context

When default mapping is not enough, pass adapter context:

```ts
const searchSchema = toVbenFormSchema(RequestSchema.SearchRolesRequest, {
  t: (key, fallback) => fallback || key,
  enumLabelPolicy: 'description2',
  objectMode: 'json',
});

const columns = toVxeTableColumns(ResponseSchema.RoleSummary, {
  formatDate(value) {
    return value ? String(value) : '';
  },
});
```

Supported context fields:

- `t`
- `enumLabelPolicy`
- `objectMode`
- `formatDate`

## 10. What A Code Agent Should Do First

When asked to build a page in a UI module:

1. Find the local generated SDK package under `packages/@sdk/{domain}-{module}`.
2. Find the target service in `packages/@sdk/{domain}-{module}/src/api/*.ts`.
3. Find the matching `RequestSchema` and `ResponseSchema` under `packages/@sdk/{domain}-{module}/src/schema`.
4. Create the page under `apps/{app}/src/views/...` with `defineOptions({ name, meta })`.
5. Use `useVbenForm` for standalone forms.
6. Use `useVbenVxeGrid` for list pages.
7. Use `toVbenFormSchema(RequestSchema...)` for forms and search forms.
8. Use `toVxeTableColumns(ResponseSchema...)` for tables.
9. Use generated services for all backend calls.
10. Run `gradlew :{ui_module}:ui` when route or SDK regeneration is needed.
11. Verify generated `router`, `menu`, and `page.json` together when changing route structure or page metadata.

## 11. Hard Do-Not-Do List

- Do not edit `src/router/routes/modules/*.ts`.
- Do not edit `src/menus/index.ts` or `src/menus/menu.json`.
- Do not hand-maintain `src/locales/langs/{lang}/page.json` route keys outside the generator contract.
- Do not edit generated SDK files under `packages/@sdk/{domain}-{module}/src`.
- Do not use `fetch`, `axios`, or `requestClient` in business/page code.
- Do not call `configureApiClient` in page code.
- Do not use `gridOptions.formConfig` for search forms.
- Do not use `toAntdTableColumns(...)` for VXE table pages.
- Do not forget `page.currentPage - 1` when building `PageRequest.page`.
- Do not assume old `items/total` paging shape. Current contract is `data/totalCount`.
- Do not import `packages/@sdk/{domain}-{module}/src/http.ts` in feature code.
- Do not put `$t(...)` inside `defineOptions({ meta: { title } })`.

## 12. One-Line Mental Model

Route comes from `views`, backend calls come from generated `api`, forms come from `RequestSchema`, tables come from `ResponseSchema`, and page code should only compose those pieces.
