---
title: Vben Form
description: How to use the vben form component.
---

# Vben Form 表单

框架提供的表单组件，可适配 `Element Plus`、`Ant Design Vue`、`Naive UI` 等框架。

**写在前面**

如果你觉得现有组件的封装不够理想，或者不完全符合你的目标，大可以直接使用原生组件，亦或亲手封装一个适合的组件。框架提供的组件并非束缚，使用与否，完全取决于你的需求。

## 适配器

表单底层使用 [vee-validate](https://vee-validate.logaretm.com/v4/) 进行表单验证，所以你可以使用 `vee-validate` 的所有功能。对于不同的 UI 框架，我们提供了适配器，以便更好的适配不同的 UI 框架。

### 适配器说明

每个应用都有不同的 UI 框架，所以在应用的 `src/adapter/form` 和 `src/adapter/component` 内部，你可以根据自己的需求，进行组件适配。

## 示例

| 名称 | 路径 | 说明 |
|------|------|------|
| 基础用法 | [basic](demo/basic/index.vue) | 使用 `useVbenForm` 创建最基础的表单。 |
| 查询表单 | [query](demo/query/index.vue) | 用于查询数据的表单，不触发验证只触发查询事件。 |
| 值格式化 | [value-format](demo/value-format/index.vue) | 通过 `valueFormat` 处理组件展示值与后端 payload 不一致的场景。 |
| 表单校验 | [rules](demo/rules/index.vue) | 通过 `rules` 属性进行表单校验，支持预定义规则和 zod。 |
| 表单联动 | [dynamic](demo/dynamic/index.vue) | 通过 `dependencies` 属性实现字段间的显示/隐藏/禁用等联动。 |
| 自定义组件 | [custom](demo/custom/index.vue) | 在表单中使用自定义组件和插槽。 |
| 操作 | [api](demo/api/index.vue) | 通过 `formApi` 进行表单值设置、schema更新等操作。 |

## API

`useVbenForm` 返回一个数组，第一个元素是表单组件，第二个元素是表单的方法。

```vue
<script setup lang="ts">
import { useVbenForm } from '#/adapter/form';

// Form 为表单组件
// formApi 为表单的方法
const [Form, formApi] = useVbenForm({
  // 属性
  // 事件
});
</script>

<template>
  <Form />
</template>
```

### FormApi

useVbenForm 返回的第二个参数，是一个对象，包含了一些表单的方法。

| 方法名 | 描述 | 类型 | 版本号 |
| --- | --- | --- | --- |
| submitForm | 提交表单 | `(e:Event)=>Promise<Record<string,any>>` | - |
| validateAndSubmitForm | 提交并校验表单 | `(e:Event)=>Promise<Record<string,any>>` | - |
| resetForm | 重置表单 | `()=>Promise<void>` | - |
| setValues | 设置表单值, 默认会过滤不在schema中定义的field, 可通过filterFields形参关闭过滤 | `(fields: Record<string, any>, filterFields?: boolean, shouldValidate?: boolean) => Promise<void>` | - |
| getValues | 获取表单值 | `()=>Promise<Record<string, any>>` | - |
| validate | 表单校验 | `()=>Promise<void>` | - |
| validateField | 校验指定字段 | `(fieldName: string)=>Promise<ValidationResult<unknown>>` | - |
| isFieldValid | 检查某个字段是否已通过校验 | `(fieldName: string)=>Promise<boolean>` | - |
| resetValidate | 重置表单校验 | `()=>Promise<void>` | - |
| updateSchema | 更新formSchema | `(schema:FormSchema[])=>void` | - |
| setFieldValue | 设置字段值 | `(field: string, value: any, shouldValidate?: boolean)=>Promise<void>` | - |
| setState | 设置组件状态（props） | `(stateOrFn:\| ((prev: VbenFormProps) => Partial<VbenFormProps>)\| Partial<VbenFormProps>)=>Promise<void>` | - |
| getState | 获取组件状态（props） | `()=>Promise<VbenFormProps>` | - |
| form | 表单对象实例，可以操作表单，见 [useForm](https://vee-validate.logaretm.com/v4/api/use-form/) | - | - |
| getFieldComponentRef | 获取指定字段的组件实例 | `<T=unknown>(fieldName: string)=>T` | >5.5.3 |
| getFocusedField | 获取当前已获得焦点的字段 | `()=>string\|undefined` | >5.5.3 |

## Props

所有属性都可以传入 `useVbenForm` 的第一个参数中。

| 属性名 | 描述 | 类型 | 默认值 |
| --- | --- | --- | --- |
| layout | 表单项布局 | `'horizontal' \| 'vertical'\| 'inline'` | `horizontal` |
| showCollapseButton | 是否显示折叠按钮 | `boolean` | `false` |
| wrapperClass | 表单的布局，基于tailwindcss | `any` | - |
| actionWrapperClass | 表单操作区域class | `any` | - |
| actionLayout | 表单操作按钮位置 | `'newLine' \| 'rowEnd' \| 'inline'` | `rowEnd` |
| actionPosition | 表单操作按钮对齐方式 | `'left' \| 'center' \| 'right'` | `right` |
| handleReset | 表单重置回调 | `(values: Record<string, any>,) => Promise<void> \| void` | - |
| handleSubmit | 表单提交回调 | `(values: Record<string, any>,) => Promise<void> \| void` | - |
| handleValuesChange | 表单值变化回调 | `(values: Record<string, any>, fieldsChanged: string[]) => void` | - |
| handleCollapsedChange | 表单收起展开状态变化回调 | `(collapsed: boolean) => void` | - |
| actionButtonsReverse | 调换操作按钮位置 | `boolean` | `false` |
| resetButtonOptions | 重置按钮组件参数 | `ActionButtonOptions` | - |
| submitButtonOptions | 提交按钮组件参数 | `ActionButtonOptions` | - |
| showDefaultActions | 是否显示默认操作按钮 | `boolean` | `true` |
| collapsed | 是否折叠，在`showCollapseButton`为`true`时生效 | `boolean` | `false` |
| collapseTriggerResize | 折叠时，触发`resize`事件 | `boolean` | `false` |
| collapsedRows | 折叠时保持的行数 | `number` | `1` |
| fieldMappingTime | 用于将表单内的数组值映射成 2 个字段 | `[string, [string, string],Nullable<string>\|[string,string]\|((any,string)=>any)?][]` | - |
| commonConfig | 表单项的通用配置，每个配置都会传递到每个表单项，表单项可覆盖 | `FormCommonConfig` | - |
| schema | 表单项的每一项配置 | `FormSchema[]` | - |
| submitOnEnter | 按下回车键时提交表单 | `boolean` | `false` |
| submitOnChange | 字段值改变时提交表单(内部防抖，这个属性一般用于表格的搜索表单) | `boolean` | `false` |
| compact | 是否紧凑模式(忽略为校验信息所预留的空间) | `boolean` | `false` |
| scrollToFirstError | 表单验证失败时是否自动滚动到第一个错误字段 | `boolean` | `false` |

## Slots

可以使用以下插槽在表单中插入自定义的内容

| 插槽名        | 描述               |
| ------------- | ------------------ |
| reset-before  | 重置按钮之前的位置 |
| submit-before | 提交按钮之前的位置 |
| expand-before | 展开按钮之前的位置 |
| expand-after  | 展开按钮之后的位置 |

**字段插槽**

除了以上内置插槽之外，`schema`属性中每个字段的`fieldName`都可以作为插槽名称，这些字段插槽的优先级高于`component`定义的组件。也就是说，当提供了与`fieldName`同名的插槽时，这些插槽的内容将会作为这些字段的组件，此时`component`的值将会被忽略。
