---
title: Vben Drawer
description: How to use the vben drawer component.
---

# Vben Drawer 抽屉

框架提供的抽屉组件，支持`自动高度`、`loading`等功能。

## 示例

| 名称 | 路径 | 说明 |
|------|------|------|
| 基础用法 | [basic](demo/basic/index.vue) | 使用 `useVbenDrawer` 创建最基础的抽屉。 |
| 组件抽离 | [extra](demo/extra/index.vue) | 通过 `connectedComponent` 参数将内外组件连接，方便复用。 |
| 自动计算高度 | [auto-height](demo/auto-height/index.vue) | 自动计算内容高度，结合 `loading` 效果和 `prepend-footer` 插槽。 |
| 使用 Api | [dynamic](demo/dynamic/index.vue) | 通过 `drawerApi` 调用方法和使用 `setState` 更新状态。 |
| 数据共享 | [shared-data](demo/shared-data/index.vue) | 通过 `connectedComponent` 参数实现内外组件数据共享。 |

**注意**

- `VbenDrawer` 组件对于参数的处理优先级是 `slot` > `props` > `state`(通过api更新的状态以及useVbenDrawer参数)。如果你已经传入了 `slot` 或者 `props`，那么 `setState` 将不会生效，这种情况下你可以通过 `slot` 或者 `props` 来更新状态。
- 如果你使用到了 `connectedComponent` 参数，那么会存在 2 个`useVbenDrawer`, 此时，如果同时设置了相同的参数，那么以内部为准（也就是没有设置 connectedComponent 的代码）。比如 同时设置了 `onConfirm`，那么以内部的 `onConfirm` 为准。`onOpenChange`事件除外，内外都会触发。
- 使用了`connectedComponent`参数时，可以配置`destroyOnClose`属性来决定当关闭弹窗时，是否要销毁`connectedComponent`组件（重新创建`connectedComponent`组件，这将会把其内部所有的变量、状态、数据等恢复到初始状态）。
- 如果抽屉的默认行为不符合你的预期，可以在对应应用的 `apps/<app>/src/bootstrap.ts` 中修改 `setDefaultDrawerProps` 的参数来设置默认属性，例如修改默认 `zIndex` 等。

## API

```ts
// Drawer 为弹窗组件
// drawerApi 为弹窗的方法
const [Drawer, drawerApi] = useVbenDrawer({
  // 属性
  // 事件
});
```

### Props

所有属性都可以传入 `useVbenDrawer` 的第一个参数中。

| 属性名 | 描述 | 类型 | 默认值 |
| --- | --- | --- | --- |
| appendToMain | 是否挂载到内容区域（默认挂载到body） | `boolean` | `false` |
| connectedComponent | 连接另一个Drawer组件 | `Component` | - |
| destroyOnClose | 关闭时销毁 | `boolean` | `false` |
| title | 标题 | `string\|slot` | - |
| titleTooltip | 标题提示信息 | `string\|slot` | - |
| description | 描述信息 | `string\|slot` | - |
| isOpen | 弹窗打开状态 | `boolean` | `false` |
| loading | 弹窗加载状态 | `boolean` | `false` |
| closable | 显示关闭按钮 | `boolean` | `true` |
| closeIconPlacement | 关闭按钮位置 | `'left'\|'right'` | `right` |
| modal | 显示遮罩 | `boolean` | `true` |
| header | 显示header | `boolean` | `true` |
| footer | 显示footer | `boolean\|slot` | `true` |
| confirmLoading | 确认按钮loading状态 | `boolean` | `false` |
| closeOnClickModal | 点击遮罩关闭弹窗 | `boolean` | `true` |
| closeOnPressEscape | esc 关闭弹窗 | `boolean` | `true` |
| confirmText | 确认按钮文本 | `string\|slot` | `确认` |
| cancelText | 取消按钮文本 | `string\|slot` | `取消` |
| placement | 抽屉弹出位置 | `'left'\|'right'\|'top'\|'bottom'` | `right` |
| showCancelButton | 显示取消按钮 | `boolean` | `true` |
| showConfirmButton | 显示确认按钮 | `boolean` | `true` |
| class | modal的class，宽度通过这个配置 | `string` | - |
| contentClass | modal内容区域的class | `string` | - |
| footerClass | modal底部区域的class | `string` | - |
| headerClass | modal顶部区域的class | `string` | - |
| zIndex | 抽屉的ZIndex层级 | `number` | `1000` |
| overlayBlur | 遮罩模糊度 | `number` | - |

**appendToMain**

`appendToMain`可以指定将抽屉挂载到内容区域，打开抽屉时，内容区域以外的部分（标签栏、导航菜单等等）不会被遮挡。默认情况下，抽屉会挂载到body上。但是：挂载到内容区域时，作为页面根容器的`Page`组件，需要设置`auto-content-height`属性，以便抽屉能够正确计算高度。

### Event

以下事件，只有在 `useVbenDrawer({onCancel:()=>{}})` 中传入才会生效。

| 事件名 | 描述 | 类型 | 版本限制 |
| --- | --- | --- | --- |
| onBeforeClose | 关闭前触发，返回 `false` 或 Promise reject 则禁止关闭 | `()=>Promise<boolean \| undefined>\|boolean\|undefined` | >5.5.2 支持 Promise |
| onCancel | 点击取消按钮触发 | `()=>void` | - |
| onClosed | 关闭动画播放完毕时触发 | `()=>void` | >5.5.2 |
| onConfirm | 点击确认按钮触发 | `()=>void` | - |
| onOpenChange | 关闭或者打开弹窗时触发 | `(isOpen:boolean)=>void` | - |
| onOpened | 打开动画播放完毕时触发 | `()=>void` | >5.5.2 |

### Slots

除了上面的属性类型包含`slot`，还可以通过插槽来自定义弹窗的内容。

| 插槽名         | 描述                                               |
| -------------- | -------------------------------------------------- |
| default        | 默认插槽 - 弹窗内容                                |
| prepend-footer | 取消按钮左侧                                       |
| center-footer  | 取消按钮和确认按钮中间（不使用 footer 插槽时有效） |
| append-footer  | 确认按钮右侧                                       |
| close-icon     | 关闭按钮图标                                       |
| extra          | 额外内容(标题右侧)                                 |

### drawerApi

| 方法 | 描述 | 类型 | 版本限制 |
| --- | --- | --- | --- |
| setState | 动态设置抽屉状态属性 | `(((prev: DrawerState) => Partial<DrawerState>)\| Partial<DrawerState>)=>drawerApi` | - |
| open | 打开弹窗 | `()=>void` | - |
| close | 关闭弹窗 | `()=>void` | - |
| setData | 设置共享数据 | `<T>(data:T)=>drawerApi` | - |
| getData | 获取共享数据 | `<T>()=>T` | - |
| useStore | 获取可响应式状态 | - | - |
| lock | 将抽屉标记为提交中，锁定当前状态 | `(isLock:boolean)=>drawerApi` | >5.5.3 |
| unlock | lock方法的反操作，解除抽屉的锁定状态，也是lock(false)的别名 | `()=>drawerApi` | >5.5.3 |

**lock**

`lock`方法用于锁定抽屉的状态，一般用于提交数据的过程中防止用户重复提交或者抽屉被意外关闭、表单数据被改变等等。当处于锁定状态时，抽屉的确认按钮会变为loading状态，同时禁用取消按钮和关闭按钮、禁止ESC或者点击遮罩等方式关闭抽屉、开启抽屉的spinner动画以遮挡弹窗内容。调用`close`方法关闭处于锁定状态的抽屉时，会自动解锁。要主动解除这种状态，可以调用`unlock`方法或者再次调用lock方法并传入false参数。
