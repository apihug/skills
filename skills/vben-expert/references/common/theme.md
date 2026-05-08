---
title: 主题与样式
description: Theme configuration, CSS variables, Tailwind CSS, and style conventions.
---

# 主题与样式

## 概述

框架基于 [shadcn-vue](https://www.shadcn-vue.com/themes.html) 和 [Tailwind CSS v4](https://tailwindcss.com/) 构建，提供了完整的主题体系、CSS 变量规范和样式管理方案。

## 主题配置

### 更改品牌主色

在应用目录下的 `preferences.ts` 中覆盖品牌色：

```ts
import { defineOverridesPreferences } from '@vben/preferences';

export const overridesPreferences = defineOverridesPreferences({
  theme: {
    colorDestructive: 'hsl(348 100% 61%)',  // 错误色
    colorPrimary: 'hsl(212 100% 45%)',      // 主题色
    colorSuccess: 'hsl(144 57% 58%)',       // 成功色
    colorWarning: 'hsl(42 84% 61%)',        // 警告色
  },
});
```

**注意**：
- CSS 变量中使用 `hsl` 格式（如 `0 0% 100%`），不需要加 `hsl()` 和 `,`
- `preferences.ts` 中品牌色使用 `hsl()` 包装（如 `'hsl(212 100% 45%)'`）
- 修改后需要清空缓存才生效

### 切换内置主题

框架内置 16 种主题，通过 `theme.builtinType` 切换：

```ts
import { defineOverridesPreferences } from '@vben/preferences';

export const overridesPreferences = defineOverridesPreferences({
  theme: {
    builtinType: 'default',  // 'deep-blue' | 'green' | 'orange' | 'pink' | ...
  },
});
```

**内置主题列表**：

`custom` · `deep-blue` · `deep-green` · `default` · `gray` · `green` · `neutral` · `orange` · `pink` · `red` · `rose` · `sky-blue` · `slate` · `stone` · `violet` · `yellow` · `zinc`

### 默认 CSS 变量

主题颜色基于 shadcn-vue 规范，核心变量：

```
--background        body 背景色
--foreground        主体文字色
--primary           主色
--primary-foreground  主色上的文字色
--destructive       错误色
--success           成功色
--warning           警告色
--secondary         次要色
--accent            强调色
--muted             柔和背景
--border            边框色
--card              卡片背景
--sidebar           侧栏背景
--header            头部背景
--radius            基础圆角
--font-size-base    基础字号
```

覆盖方式（在项目 CSS 中）：

```css
/* 默认主题下 */
:root {
  --card: 0 0% 30%;
}

/* 黑暗模式下 */
.dark {
  --card: 222.34deg 10.43% 12.27%;
}
```

## Tailwind CSS

项目使用 **Tailwind CSS v4**，配置统一在 `internal/tailwind-config/src/theme.css` 中，不再使用 `tailwind.config.*` 文件。

### 在 Vue SFC 中使用

```vue
<template>
  <div class="bg-background text-foreground">
    <p class="text-primary p-4">hello world</p>
  </div>
</template>
```

### 在 SFC 中使用 `@apply`

项目中 `@apply` 会自动注入 `@reference "@vben/tailwind-config/theme"`，无需手动引用：

```vue
<style scoped>
.box {
  @apply bg-white p-4 text-green;
}
</style>
```

### 包使用 Tailwind CSS

所有应用和包统一复用 `@vben/vite-config` 接入 `@tailwindcss/vite`，扫描范围在 `@vben/tailwind-config` 中统一维护。纯 SDK 包不需要使用 Tailwind 时，无需额外配置。

## 样式预处理器

### SCSS

项目使用 SCSS 作为样式预处理器：

```vue
<style lang="scss" scoped>
$font-size: 30px;

.box {
  .title {
    color: green;
    font-size: $font-size;
  }
}
</style>
```

### PostCSS + CSS Variables

不习惯 SCSS 时可使用 PostCSS + CSS 变量，项目内置 `postcss-nested`：

```vue
<style scoped>
.box {
  --font-size: 30px;
  .title {
    color: green;
    font-size: var(--font-size);
  }
}
</style>
```

### BEM 命名规范

样式冲突时使用 `useNamespace` 函数：

```vue
<script lang="ts" setup>
import { useNamespace } from '@vben/hooks';

const { b, e, is } = useNamespace('menu');
</script>

<template>
  <div :class="[b()]">
    <div :class="[e('item'), is('active', true)]">item1</div>
  </div>
</template>

<style lang="scss" scoped>
@use '@vben/styles/global' as *;
@include b('menu') {
  color: black;
  @include e('item') {
    background-color: black;
    @include is('active') {
      background-color: red;
    }
  }
}
</style>
```

### CSS Modules

```vue
<template>
  <p :class="$style.red">This should be red</p>
</template>

<style module>
.red {
  color: red;
}
</style>
```

## 样式文件结构

全局样式位于 `@vben/styles`，包含重置样式、全局变量等，继承自 `@vben-core/design`。
