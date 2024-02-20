---
title: 创建并发布的npm库(npm packages)
date: 2024-02-17 10:00:00
tags:
  - npm
  - vite
  - npm package
---

这里分别介绍两种方法创建自己的npm package，分别是最流行的两个框架 vite 和 react。

<!-- more -->

# 用 vite 创建 npm 库 

首先，确保你已经安装了 [vite](https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project) , vite 5+ 的版本直接用以下命令就可以创建一个项目：

```bash
npm init vite VueButtonYy
```
运行后会在命令行里选择一个模板，选择 `vue` 即可。

## 创建一个vue的组件：

### 创建文件夹

项目创建完成后，我们将原来的 `src` 文件夹删除，然后新建一个 `src` 文件夹，然后在 `src` 文件夹下新建一个 `index.js` 文件和一个 `components` 文件夹，文件夹目录结构如下：

```bash
/src
  |__index.js
  |__/components
      |__VueButtonYy.vue
```

### 创建组件
其中 `VueButtonYy.vue` 文件就是一个简单的vue组件。我这里随便写了一个button组件，代码如下：

```vue
<template>
  <button class="vue-button-yy">
    <slot />
  </button>
</template>

<style scoped>
  .vue-button-yy {
    background: #43b883;
    color: white;
    outline: none;
    border: none;
  }
</style>
```

### 编写vite配置文件

vite 的功能强大，可以通过配置文件 `vite.config.js` 来实现一些功能，比如打包成一个npm package。这里我用 `vite.config.js`，配置文件 `vite.config.js` 代码如下：

```jsx
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.js'),
      name: 'VueButtonYy',
      // 输出文件的格式名：rollup是用 umd和es进行区分的：
      fileName: (format) => `vue-button-yy.${format}.js`
    },
    rollupOptions: {
      external: ['vue'],
      globals: {
        vue: 'Vue'
      }
    }
  },
  plugins: [vue()],
})

```

这里需要注意
- fileName参数，用来指定输出文件的格式名，这里我用的是 `vue-button-yy.${format}.js`，这样打包出来的文件名就是 `vue-button-yy.cjs.js` 和 `vue-button-yy.esm.js`。
- vite是用rollup打包的，所以需要配置rollupOptions，配置rollupOptions的external参数，将vue的包名 `vue` 添加到external中，这样打包的时候，vue的包不会打包进去，出来的文件会更小。

## 编写入口文件 `index.js`

```jsx
import VueButtonYy from './components/VueButtonYy.vue'

// 具名导出：
export { VueButtonYy };

// 默认导出：
export default {
  install: (app) => {
    app.component('VueButtonYy', VueButtonYy);
    // 如有多个插件：
    // app.component('Xxxx', xxxx);
  },
};
```

## 编写 `package.json` 文件

```json
{
  "name": "vue-button-yy",
  "version": "1.0.0",
  "files": [
    "dist"
  ],
  "main": "./dist/vue-button-yy.umd.js",
  "module": "./dist/vue-button-yy.es.js",
  "exports": {
    ".": {
      "import": "./dist/vue-button-yy.es.js",
      "require": "./dist/vue-button-yy.umd.js"
    },
    "./dist/style.css": "./dist/style.css"
  },
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "vue": "^3.4.15"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^5.0.3",
    "vite": "^5.1.0"
  }
}
```

- name: 这里我用的是 `vue-button-yy`，这个就是你创建的npm包的名称，这个名称在npm上不能重复，所以要自己取一个。
- version: 这里我用的是 `1.0.0`，这个就是你创建的npm包的版本号，每次发布一个版本，都需要在这里修改版本号，然后在命令行里输入 `npm publish`，发布一个新版本。
- files: 这里我用的是 `dist`，这个就是打包后的文件夹名称，打包后的文件会放在 `dist` 文件夹下。
- main: 这里我用的是 `./dist/vue-button-yy.umd.js`，这个就是打包后的入口文件，打包后的入口文件会放在 `dist` 文件夹下。`vite` 用的是 rollup.js 打包，rollup 默认的格式为 `umd` (require 导入) 和 `es` （import 导入） 两种文件名格式，写在 packge.json 里做一个说明，用的时候会根据main 和 module的文件名进行自动选择。

## 本地测试
在一个包编写完成后，我们需要在本地进行测试，在包文件夹的根目录下跑命令

```bash
npm link
```
生成一个本地全局软链接，再去另外一个vue的现成项目里，通过 
```bash
npm link vue-button-yy
``` 
进行引用即可。

比如我在另一个项目里跑了 `npm link vue-button-yy`，可以看到该项目的 `package.json` 文件里添加了 `vue-button-yy` 的依赖如下，这是一种本地引用的方式

```json
  "dependencies": {
    "vue-button-yy": "file:../vueDemoComp"
  },
```

然后在 `main.js` 里引用了 `vue-button-yy` 的组件，代码如下：

```jsx
import VueButtonYy from 'vue-button-yy'
import 'vue-button-yy/dist/style.css'

const app = createApp(App)

app.use(VueButtonYy);
app.mount('#app')
```

在 `App.vue` 里引用了 `vue-button-yy` 的组件，代码如下：

```jsx
<script setup>
</script>

<template>
  <vue-button-yy>
    hello world
  </vue-button-yy>
</template>

<style scoped>
</style>
```

## 发布你的npm包

```bash
npm login
npm publish --access=public
```

## 创建一个react的npm package
（待更新）
