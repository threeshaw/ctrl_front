# ctrl_front

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) to make the TypeScript language service aware of `.vue` types.

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```

### Run Unit Tests with [Vitest](https://vitest.dev/)

```sh
npm run test:unit
```

### Run End-to-End Tests with [Playwright](https://playwright.dev)

```sh
# Install browsers for the first run
npx playwright install

# When testing on CI, must build the project first
npm run build

# Runs the end-to-end tests
npm run test:e2e
# Runs the tests only on Chromium
npm run test:e2e -- --project=chromium
# Runs the tests of a specific file
npm run test:e2e -- tests/example.spec.ts
# Runs the tests in debug mode
npm run test:e2e -- --debug
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```

7.25update
新增文件输入功能，保存输入excel为json，并在左边文件列表中增加一个提示。

8.3 update
在主界面newStru界面新增了路由跳转功能，当点击数据/模型后应该在右边框内加载对应模块，但是路由目前存在一些bug待修理
目前构思的两个主界面：dataflow对应io设定与模型选择，datalist对应数据观察，可以加载到APP.vue中看对应模块。

8.4 update
目前看起来数据管理模块可以直接导入，父子组件之间元素相互调用也没问题，但是使用路由导入时遇到未知的问题。

8.5 update
按照需求重构了页面，但是路由的问题很大。同时，删除功能还在优化中。
