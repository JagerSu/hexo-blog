# Vue 3 BPMN流程设计器详细集成方案

## 项目概述

本方案基于 `moon-studio/vite-vue-bpmn-process` 为Vue 3项目提供完整的BPMN流程设计器集成方案，支持源码级别的二次开发。

## 技术栈

- **Vue 3.4+** + **TypeScript 5+**
- **Vite 5+** 构建工具
- **bpmn-js 17+** 流程引擎
- **Element Plus** 或 **Naive UI**（可选）
- **Pinia** 状态管理
- **SCSS** 样式预处理

## 第一步：项目初始化

### 1. 创建Vue 3项目

```bash
# 使用 Vite 创建 Vue 3 + TypeScript 项目
npm create vue@latest my-bpmn-project
cd my-bpmn-project

# 选择配置：
# ✅ TypeScript
# ✅ Router
# ✅ Pinia
# ✅ ESLint
# ✅ Prettier
```

### 2. 安装核心依赖

```bash
npm install bpmn-js@^17.0.0 \
           diagram-js@^14.0.0 \
           bpmn-js-properties-panel@^5.0.0 \
           @bpmn-io/properties-panel@^3.0.0 \
           camunda-bpmn-moddle@^7.0.0 \
           bpmn-moddle@^8.0.0
```

### 3. 安装UI库和工具依赖

```bash
# Element Plus 版本
npm install element-plus @element-plus/icons-vue

# 或者 Naive UI 版本
npm install naive-ui @vueuse/core

# 开发依赖
npm install --save-dev \
    @types/file-saver \
    raw-loader \
    sass
```

## 第二步：项目结构设计

```
src/
├── components/
│   └── bpmn-designer/              # BPMN设计器核心组件
│       ├── Designer.vue            # 主设计器组件
│       ├── Toolbar.vue             # 工具栏组件
│       ├── PropertyPanel.vue       # 属性面板组件
│       ├── modules/                # bpmn.js 扩展模块
│       │   ├── palette/            # 自定义画板
│       │   ├── context-pad/        # 上下文菜单
│       │   ├── renderer/           # 自定义渲染器
│       │   └── rules/              # 自定义规则
│       └── config/                 # 配置文件
├── stores/                         # Pinia状态管理
│   └── bpmn.ts                     # BPMN相关状态
├── utils/                          # 工具函数
│   ├── bpmn-utils.ts               # BPMN工具函数
│   └── file-utils.ts               # 文件处理工具
├── types/                          # TypeScript类型定义
│   └── bpmn.d.ts                   # BPMN相关类型
└── assets/
    ├── bpmn/                       # BPMN相关资源
    │   ├── empty.bpmn              # 空流程模板
    │   └── extensions/             # 扩展配置
    └── styles/
        └── bpmn.scss               # BPMN样式
```

## 第三步：下载和集成源码

### 1. 克隆源码

```bash
# 克隆 Vue 3 版本源码
git clone https://github.com/moon-studio/vite-vue-bpmn-process.git bpmn-source
cd bpmn-source
git checkout backed  # 使用backed分支（较为稳定）
```

### 2. 复制核心文件

将以下文件复制到你的项目中：

```bash
# 复制核心组件
cp -r bpmn-source/src/components/* your-project/src/components/bpmn-designer/

# 复制工具函数
cp -r bpmn-source/src/utils/* your-project/src/utils/

# 复制类型定义
cp -r bpmn-source/types/* your-project/src/types/

# 复制样式文件
cp -r bpmn-source/src/styles/* your-project/src/assets/styles/

# 复制配置文件
cp -r bpmn-source/src/config/* your-project/src/components/bpmn-designer/config/
```

## 第四步：配置构建工具

### 1. 更新 vite.config.ts

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@bpmn': resolve(__dirname, 'src/components/bpmn-designer')
    }
  },
  define: {
    __VUE_PROD_HYDRATION_MISMATCH_DETAILS__: false
  },
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/assets/styles/variables.scss";`
      }
    }
  },
  assetsInclude: ['**/*.bpmn'],
  optimizeDeps: {
    include: [
      'bpmn-js',
      'bpmn-js-properties-panel',
      'diagram-js',
      'bpmn-moddle'
    ]
  }
})
```

### 2. 更新 tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "preserve",
    "strict": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@bpmn/*": ["src/components/bpmn-designer/*"]
    }
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue"
  ],
  "exclude": ["node_modules"]
}
```

## 第五步：核心组件实现

### 1. 主设计器组件

详见后续文件：`BpmnDesigner.vue`

### 2. 状态管理

详见后续文件：`bpmn-store.ts`

### 3. 工具函数

详见后续文件：`bpmn-utils.ts`

## 第六步：样式集成

### 1. 引入必要样式

```scss
// src/assets/styles/bpmn.scss
@import 'bpmn-js/dist/assets/diagram-js.css';
@import 'bpmn-js/dist/assets/bpmn-font/css/bpmn.css';
@import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-codes.css';
@import '@bpmn-io/properties-panel/assets/properties-panel.css';

// 自定义样式
.bpmn-container {
  width: 100%;
  height: 100%;
  position: relative;
  
  .bjs-container {
    height: 100%;
  }
}
```

## 第七步：使用示例

### 1. 在主应用中使用

```vue
<template>
  <div class="app">
    <BpmnDesigner
      v-model:xml="processXml"
      :height="600"
      @element-click="handleElementClick"
      @save="handleSave"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import BpmnDesigner from '@bpmn/Designer.vue'

const processXml = ref('')

const handleElementClick = (element: any) => {
  console.log('Element clicked:', element)
}

const handleSave = (xml: string) => {
  console.log('Process saved:', xml)
}
</script>
```

## 二次开发指南

### 1. 自定义画板元素

详见后续文件：`custom-palette.ts`

### 2. 自定义属性面板

详见后续文件：`custom-properties.ts`

### 3. 自定义渲染器

详见后续文件：`custom-renderer.ts`

## 常见问题解决

### 1. 依赖版本冲突

```bash
# 如果遇到版本冲突，删除 node_modules 重新安装
rm -rf node_modules package-lock.json
npm install
```

### 2. 样式问题

确保正确引入所有必要的CSS文件，特别是bpmn-js的字体文件。

### 3. TypeScript类型错误

复制完整的类型定义文件，并确保tsconfig.json配置正确。

## 总结

这个方案提供了完整的Vue 3集成方案，支持：
- 完全的源码级别控制
- TypeScript支持
- 模块化的组件设计
- 便于二次开发的架构

接下来的文件将提供具体的组件实现代码。