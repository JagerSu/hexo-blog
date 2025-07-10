# BPMN Process Designer 源码集成指南

## 项目概述

`miyuesc/bpmn-process-designer` 是一个基于 Vue.js 和 bpmn.js 的流程编辑器，支持 BPMN 2.0 规范的流程设计。该项目提供了多个版本分支：

- **main 分支**: Vue 2 + JavaScript + bpmn-js@8+ + ElementUI
- **v2 分支**: Vue 2 + JavaScript + bpmn-js@9+ + ElementUI（推荐）
- **next 分支**: pnpm workspace 版本（开发中）
- **Vue 3 版本**: 在 `moon-studio/vite-vue-bpmn-process` 仓库

## 集成方式选择

### 方式一：Git Submodule 集成（推荐）

这种方式可以保持源码的独立性，便于后续更新。

#### 1. 添加子模块

```bash
# 在你的项目根目录下执行
git submodule add https://github.com/miyuesc/bpmn-process-designer.git packages/bpmn-designer

# 如果需要特定分支（推荐使用 v2 分支）
git submodule add -b v2 https://github.com/miyuesc/bpmn-process-designer.git packages/bpmn-designer
```

#### 2. 初始化和更新子模块

```bash
git submodule init
git submodule update
```

#### 3. 安装依赖

```bash
cd packages/bpmn-designer
npm install
```

#### 4. 在主项目中引用

```javascript
// main.js
import BpmnDesigner from './packages/bpmn-designer/package/index.js'
import Vue from 'vue'

Vue.use(BpmnDesigner)
```

### 方式二：直接复制源码集成

#### 1. 下载源码

```bash
# 克隆项目
git clone https://github.com/miyuesc/bpmn-process-designer.git
cd bpmn-process-designer

# 切换到推荐的 v2 分支
git checkout v2
```

#### 2. 复制核心文件

将以下目录复制到你的项目中：

```
src/
├── components/
│   └── bpmn-designer/           # 复制整个 package 目录
├── styles/                      # 复制相关样式文件
└── utils/                       # 复制工具函数
```

#### 3. 安装必要依赖

```bash
npm install bpmn-js@^9.0.0 \
           bpmn-js-properties-panel \
           camunda-bpmn-moddle \
           element-ui \
           vue@^2.6.0
```

### 方式三：npm/yarn link 本地开发集成

适合需要同时开发和调试的场景。

#### 1. 克隆并构建源码

```bash
git clone https://github.com/miyuesc/bpmn-process-designer.git
cd bpmn-process-designer
git checkout v2
npm install
npm run build
npm link
```

#### 2. 在主项目中链接

```bash
cd your-project
npm link bpmn-process-designer
```

## 具体集成步骤

### 1. 安装依赖

根据你选择的版本，安装对应的依赖：

**Vue 2 版本依赖：**
```json
{
  "dependencies": {
    "vue": "^2.6.0",
    "element-ui": "^2.15.0",
    "bpmn-js": "^9.0.0",
    "bpmn-js-properties-panel": "^1.0.0",
    "camunda-bpmn-moddle": "^6.1.0",
    "diagram-js": "^8.0.0"
  }
}
```

**Vue 3 版本依赖：**
```json
{
  "dependencies": {
    "vue": "^3.2.0",
    "naive-ui": "^2.34.0",
    "bpmn-js": "^13.0.0",
    "vite": "^4.0.0",
    "typescript": "^4.9.0"
  }
}
```

### 2. 配置构建工具

#### Webpack 配置（Vue 2）

```javascript
// vue.config.js
module.exports = {
  configureWebpack: {
    resolve: {
      alias: {
        '@bpmn-designer': path.resolve(__dirname, 'packages/bpmn-designer')
      }
    },
    module: {
      rules: [
        {
          test: /\.bpmn$/,
          use: 'raw-loader'
        }
      ]
    }
  }
}
```

#### Vite 配置（Vue 3）

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@bpmn-designer': resolve(__dirname, 'packages/bpmn-designer')
    }
  },
  assetsInclude: ['**/*.bpmn']
})
```

### 3. 组件引入和使用

#### Vue 2 使用示例

```vue
<template>
  <div id="app">
    <my-process-designer
      :value="xmlString"
      :additional-model="customModules"
      :moddleExtension="moddleExtensions"
      prefix="camunda"
      @init-finished="handleInit"
      @change="handleChange"
    />
  </div>
</template>

<script>
import MyProcessDesigner from '@bpmn-designer/package/index.js'

export default {
  name: 'App',
  components: {
    MyProcessDesigner
  },
  data() {
    return {
      xmlString: '',
      customModules: [],
      moddleExtensions: {}
    }
  },
  methods: {
    handleInit(modeler) {
      console.log('BPMN Modeler initialized:', modeler)
    },
    handleChange(xml) {
      console.log('Process changed:', xml)
      this.xmlString = xml
    }
  }
}
</script>

<style>
@import '@bpmn-designer/package/theme/index.scss';
</style>
```

#### Vue 3 使用示例

```vue
<template>
  <div id="app">
    <BpmnDesigner
      v-model:xml="xmlString"
      :height="600"
      @save="handleSave"
      @element-click="handleElementClick"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import BpmnDesigner from '@bpmn-designer/src/components/Designer.vue'

const xmlString = ref('')

const handleSave = (xml: string) => {
  console.log('Saved XML:', xml)
}

const handleElementClick = (element: any) => {
  console.log('Element clicked:', element)
}
</script>
```

### 4. 样式集成

#### 引入必要的样式文件

```scss
// main.scss
@import 'bpmn-js/dist/assets/diagram-js.css';
@import 'bpmn-js/dist/assets/bpmn-font/css/bpmn.css';
@import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-codes.css';

// 如果使用 ElementUI
@import 'element-ui/lib/theme-chalk/index.css';

// 引入设计器样式
@import '@bpmn-designer/package/theme/index.scss';
```

### 5. 自定义配置

#### 自定义模块示例

```javascript
// custom-modules.js
import CustomPalette from './custom-palette'
import CustomContextPad from './custom-context-pad'

export default {
  __init__: ['customPalette', 'customContextPad'],
  customPalette: ['type', CustomPalette],
  customContextPad: ['type', CustomContextPad]
}
```

#### 扩展属性配置

```javascript
// moddle-extensions.js
export default {
  activiti: {
    name: 'Activiti',
    uri: 'http://activiti.org/bpmn',
    prefix: 'activiti',
    xml: {
      tagAlias: 'lowerCase'
    },
    associations: [],
    types: [
      // 自定义类型定义
    ]
  }
}
```

## 不同版本对比

| 特性 | Vue 2 (main) | Vue 2 (v2) | Vue 3 |
|------|-------------|------------|-------|
| 稳定性 | ❌ 问题较多 | ✅ 相对稳定 | ✅ 现代化 |
| TypeScript | ❌ | ❌ | ✅ |
| UI 库 | ElementUI | ElementUI | Naive UI |
| bpmn.js 版本 | 8.x | 9.x | 13.x |
| 维护状态 | 停止 | 维护中 | 活跃 |

## 推荐集成方案

### 对于新项目：
- **Vue 3 项目**: 使用 `moon-studio/vite-vue-bpmn-process`
- **Vue 2 项目**: 使用 v2 分支

### 对于现有项目：
1. **Git Submodule 方式** - 便于跟踪更新
2. **源码复制方式** - 便于深度自定义

## 常见问题解决

### 1. 依赖冲突

```bash
# 如果遇到依赖版本冲突，使用 resolutions 强制指定版本
# package.json
{
  "resolutions": {
    "bpmn-js": "^9.0.0"
  }
}
```

### 2. 样式问题

```scss
// 确保正确引入所有必要的样式文件
@import 'bpmn-js/dist/assets/diagram-js.css';
@import 'bpmn-js/dist/assets/bpmn-font/css/bpmn.css';
```

### 3. 构建问题

```javascript
// webpack.config.js - 处理 bpmn 文件
module.exports = {
  module: {
    rules: [
      {
        test: /\.bpmn$/,
        use: 'raw-loader'
      }
    ]
  }
}
```

## 后续维护

1. **定期更新**: 如果使用 Git Submodule，定期更新子模块
2. **版本锁定**: 在生产环境中锁定具体版本
3. **自定义分支**: 对于深度定制，考虑创建自己的分支

## 总结

推荐使用 **Git Submodule + v2 分支** 的集成方式，这样既能保持代码的独立性，又能获得相对稳定的功能。对于 Vue 3 项目，建议直接使用专门的 Vue 3 版本仓库。