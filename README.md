# 可视化平台框架（前端）

## 项目简介

该项目是一个基于 **Vue 3 + Vite + JavaScript** 的可视化平台前端框架，集成了 **Ruoyi 前端常用结构与工具能力**，用于构建数据大屏/可视化页面。

当前代码包含：
- 可视化大屏首页（`/index`）
- 公共布局与路由
- 接口请求封装（Axios）
- ECharts 与 DataV Vue3 组件支持
- Element Plus UI 组件

## 技术栈

- Vue 3
- Vite 3
- Vue Router 4
- Pinia / Vuex（项目中同时存在）
- Element Plus
- ECharts
- `@kjgl77/datav-vue3`
- Axios
- Sass

## 运行环境

- Node.js 16+（建议）
- npm（项目已包含 `package-lock.json`）

## 本地开发

```bash
# 进入项目目录
cd 可视化平台框架

# 安装依赖
npm install

# 启动开发环境（Vite）
npm run dev
```

默认开发配置：
- 启动端口：`8013`
- 本地代理前缀：`/dev-api`
- 代理目标：`http://localhost:9090`

## 打包命令

```bash
# 开发环境打包
npm run build:dev

# 测试环境打包
npm run build:stage

# 生产环境打包
npm run build:prod

# 预览构建结果
npm run preview
```

## 项目结构（主要目录）

```text
src/
├─ api/            接口定义与模块封装
├─ assets/         静态资源、样式、图片、图标
├─ components/     公共组件（Echart、ItemWrap 等）
├─ layout/         布局组件
├─ router/         路由配置
├─ store/          状态管理
├─ utils/          工具方法与请求封装
├─ views/          页面视图（含大屏页面模块）
├─ main.js         应用入口
└─ App.vue         根组件
```

## 页面说明

- 根路径 `/` 会重定向到 `/index`
- 可视化大屏主页面位于：`src/views/indexs/index.vue`
- 404 页面位于：`src/views/error/404.vue`

## 后端联调（本地）

根据项目约定，后端项目路径为：
- `D:\enrionment\框架\后端服务`

请确保后端服务启动后，再进行前后端联调。

## 说明

当前仓库中部分历史文件（如环境变量注释、包名等）仍保留旧项目命名痕迹，不影响本项目作为可视化平台前端框架使用。后续如需统一命名，可再做一次整理。
