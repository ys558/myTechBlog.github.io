---
title: 创建并发布的npm ui库(npm packages)
date: 2024-02-17 10:00:00
tags:
    - npm
    - react
    - vite
    - rollup
    - storybook
    - npm packag
---
![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.53/npm-package-image.png)
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

上面的步骤，已经用了一次 `vite` 作为打包工具，这里我直接用 `rollup` 作为打包工具，即 `vite`的底层打包工具，创建一个react的npm package，步骤如下：

### 创建项目文件夹安装依赖
```bash
mkdir yuwing-react-ui && cd yuwing-react-ui
npm init -y

# 安装 react 及 ts：
npm i -D @types/react typescript react

# 生成tsconfig.json 文件，配置ts检查规则
npx tsc --init

# rollup 所需的依赖：
npm i -D rollup @rollup/plugin-node-resolve @rollup/plugin-typescript @rollup/plugin-commonjs rollup-plugin-dts

```

`tsconfig.json` 文件，配置ts检查规则，代码如下：

```json
{
  "compilerOptions": {
    /* Language and Environment */
    "target": "ESNext",
    "jsx": "react-jsx",

    /* Modules */
    "module": "ESNext",                                /* Specify what module code is generated. */
    "moduleResolution": "Node",                     /* Specify how TypeScript looks up a file from a given module specifier. */

    /* JavaScript Support */
    "allowJs": false,                                  /* Allow JavaScript files to be a part of your program. Use the 'checkJS' option to get errors from these files. */
    "maxNodeModuleJsDepth": 1,                        /* Specify the maximum folder depth used for checking JavaScript files from 'node_modules'. Only applicable with 'allowJs'. */

    /* Emit */
    "declaration": true,                              /* Generate .d.ts files from TypeScript and JavaScript files in your project. 将所有写在源码的types文件生成到types文件夹下 */
    "emitDeclarationOnly": true,                      /* Only output d.ts files and not JavaScript files. */
    "sourceMap": true,                                /* Create source map files for emitted JavaScript files. */
    "outDir": "dist",                                   /* Specify an output folder for all emitted files. */
    "declarationDir": "types",                           /* Specify the output directory for generated declaration files. */

    /* Interop Constraints */
    "allowSyntheticDefaultImports": true,             /* Allow 'import x from y' when a module doesn't have a default export. */
    "esModuleInterop": true,                             /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables 'allowSyntheticDefaultImports' for type compatibility. */
    "forceConsistentCasingInFileNames": true,            /* Ensure that casing is correct in imports. */

    /* Type Checking */
    "strict": true,                                      /* Enable all strict type-checking options. */
    "noUnusedLocals": true,                           /* Enable error reporting when local variables aren't read. */
    "noUnusedParameters": true,                       /* Raise an error when a function parameter isn't read. */
    "noImplicitReturns": true,                        /* Enable error reporting for codepaths that do not explicitly return in a function. */
    "noFallthroughCasesInSwitch": true,               /* Enable error reporting for fallthrough cases in switch statements. */
    "noUncheckedIndexedAccess": true,                 /* Add 'undefined' to a type when accessed using an index. */
    "allowUnreachableCode": true,                     /* Disable error reporting for unreachable code. */

    /* Completeness */
    "skipLibCheck": true                                 /* Skip type checking all .d.ts files. */
  }
}

```

创建rollup.config.mjs 文件，配置rollup打包规则，代码如下：

```bash
# 这里用 mjs 扩展名，因为我们文件里用到 import 语句，不再用require()
touch rollup.config.mjs
```
```js
import commonjs from "@rollup/plugin-commonjs";
import resolve from "@rollup/plugin-node-resolve"
import typescript from "@rollup/plugin-typescript";
import dts from "rollup-plugin-dts"
import packageJson from './package.json' assert { type: "json" };

export default [
  {
    input: "src/index.ts",
    output: [
      {
        file: packageJson.main,
        format: "cjs",
        sourcemap: true,
      },
      {
        file: packageJson.module,
        format: "esm",
        sourcemap: true,
      },
    ],
    external: ["@types/node"],
    plugins: [
      resolve(),
      commonjs(),
      typescript({
        tsconfig: "./tsconfig.json"
      }),
    ],
  },
  {
    input: 'dist/esm/types/index.d.ts',
    output: [
      {
        file: 'dist/index.d.ts',
        format: 'esm'
      }
    ],
    plugins: [
      dts()
    ]
  }
]
```

`package.json` 文件，配置打包规则，代码如下：
```json
{
  "name": "@yuwing/react-ui",
  "version": "0.0.1",
  "description": "simple react compoent library",
  "main": "dist/cjs/index.js",
  "module": "dist/esm/index.js",
  "type": "module",
  "types": "dist/index.d.ts",
  "files": [
    "dist"
  ],
  "scripts": {
    "dev": "rollup -c --watch",
    "build": "rollup -c"
  },
  "keywords": [
    "react","react-ui","react-component"
  ],
  "author": "Yu Yi (Kyle Yu)",
  "license": "MIT",
  "devDependencies": {
    "@rollup/plugin-commonjs": "^25.0.7",
    "@rollup/plugin-node-resolve": "^15.2.3",
    "@rollup/plugin-typescript": "^11.1.6",
    "@types/react": "^18.2.58",
    "react": "^18.2.0",
    "rollup": "^4.12.0",
    "rollup-plugin-dts": "^6.1.0",
    "typescript": "^5.3.3"
  }
}
```
### 发布自己的包
这一步和上面的vue 发布一样，不过多说，这里就不写了。

值得注意，`package.json` 文件中可以加入 `"repository"` 字段，用来配置发布包的配置，可以在上传到GitHub后，直接发布到npm仓库：

```json
{
  "repository": "https://github.com/ys558/yuwing-react-ui.git"
}
```

我们可以在GitHub上配置，然后将package的压缩文件发布在GitHub上，必须先去GitHub上创建一个token
点击右上角的头像，选择 Settings -> Developer Settings -> Personal access tokens -> Generate new token (classic) -> Select scopes -> repo -> Generate token -> Copy the token -> 把生成的token粘贴到你的.npmrc文件里，默认文件路径是 `~/.npmrc`，并添加以下字段：

```bash
registry=https://registry.npmjs.org/
@YOUR_GITHUB_USERNAME:registry=https://npm.pkg.github.com/
//npm.pkg.github.com/:_authToken=<YOUR_GITHUB_TOKEN>
```

至此，我们react组件库的基本流程已完成。
