# AGENTS.md — 可视化平台框架 Agent 指引

> 适用场景：AI Agent / 结对编程助手快速熟悉本项目，精准定位文件、遵守规范并高效完成任务。

---

## 1. 项目概览

| 属性 | 值 |
|---|---|
| 项目名 | 可视化平台框架（前端） |
| 包名 | `healthy-management` v3.8.5 |
| 框架 | Vue 3.2 + Vite 3 + JavaScript |
| UI 库 | Element Plus 2.2（中文语言包） |
| 状态管理 | Pinia 2.0 & Vuex 4（并存，新代码优先用 Pinia） |
| 路由 | Vue Router 4，`createWebHistory` 模式 |
| 可视化 | ECharts 5 + `@kjgl77/datav-vue3` |
| CSS 预处理器 | Sass |
| 后端项目路径 | **`D:\enrionment\框架\后端服务`**（需要时可直接读取后端文件，详见第 11 章） |

---

## 2. 目录结构

```
可视化平台框架/
├── src/
│   ├── api/              # 接口层
│   │   ├── api.js        # ★ 主 axios 实例（带 token、加密）；导出 GET/POST/PUT/DELETE/FILE/FILESubmit
│   │   ├── index.js      # 业务接口统一出口
│   │   ├── date.js       # 日期相关接口
│   │   └── modules/      # 按业务拆分的接口模块
│   ├── assets/
│   │   ├── css/          # public.scss、index.scss（全局样式）
│   │   └── ...           # 图片、图标、字体等
│   ├── components/       # 全局注册组件
│   │   ├── echart/       # <Echart> 封装（全局组件）
│   │   ├── item-wrap/    # <ItemWrap> 大屏卡片容器（全局组件）
│   │   ├── message/      # <Message> 消息提示（全局组件）
│   │   ├── reacquire/    # <Reacquire> 重试组件（全局组件）
│   │   └── kong.vue      # 空状态占位组件
│   ├── config/
│   │   └── UtilVar.js    # baseUrl（读 VITE_APP_BASE_API）、ENC 加密开关
│   ├── layout/           # 布局组件
│   ├── router/
│   │   └── index.js      # 路由：/ → /index（大屏首页）；404 通配
│   ├── store/            # Vuex store
│   │   └── modules/      # 子模块
│   ├── utils/
│   │   ├── request.js    # 次 axios 实例（Ruoyi 风格，走 /dev-api 代理）
│   │   ├── ruoyi.js      # parseTime / resetForm / handleTree 等全局工具方法
│   │   ├── auth.js       # token 读写（localStorage）
│   │   ├── validate.js   # 表单校验工具
│   │   ├── flexible.js   # rem 自适应
│   │   ├── drawMixin.js  # 大屏自适应 mixin
│   │   └── ...           # 其他工具
│   ├── views/
│   │   ├── indexs/       # ★ 大屏主页面（拆分为多块）
│   │   │   ├── index.vue       # 入口，组合左/中/右各区块
│   │   │   ├── left-top.vue    # 左上区块
│   │   │   ├── left-center.vue # 左中区块
│   │   │   ├── left-bottom.vue # 左下区块
│   │   │   ├── center-top.vue  # 中上区块
│   │   │   ├── center-bottom.vue
│   │   │   ├── right-top.vue
│   │   │   ├── right-center.vue
│   │   │   ├── right-bottom.vue
│   │   │   └── dev.vue         # 开发调试用页面
│   │   ├── home.vue      # 布局壳（路由 / 的 component）
│   │   ├── home.scss     # home 样式
│   │   ├── setting.vue   # 设置页
│   │   └── error/
│   │       └── 404.vue
│   ├── main.js           # 应用入口；注册全局组件、DataV、Element Plus
│   ├── App.vue           # 根组件
│   └── settings.js       # 应用级配置
├── vite.config.js        # 构建配置；路径别名 @→src, ~→根；代理 /dev-api→localhost:9090
├── vite/                 # Vite 插件集中配置
├── .env.development      # 开发环境变量（端口 8013）
├── .env.staging          # 测试环境变量
├── .env.production       # 生产环境变量
└── package.json
```

---

## 3. 路由约定

- 根路径 `/` 加载 `home.vue` 并重定向到 `/index`
- `/index` 渲染 `src/views/indexs/index.vue`（可视化大屏主页）
- 404 页面通配 `/:pathMatch(.*)*`
- **新增页面**：在 `router/index.js` 的 `constantRoutes` 或 `dynamicRoutes` 中添加

---

## 4. API 调用规范

项目存在**两套** axios 实例，用途不同，请按需选用：

### 4.1 `src/api/api.js`（主实例，推荐用于大屏数据接口）

```js
import { GET, POST, PUT, DELETE, FILE, FILESubmit } from '@/api/api.js'

// GET 请求
const res = await GET('/your/endpoint', { param1: 'value' })

// POST 请求
const res = await POST('/your/endpoint', { key: 'value' })

// 文件下载（blob/arraybuffer）
const res = await FILE({ url: '/file/download', method: 'get', responseType: 'blob' })

// 文件上传（multipart/form-data）
const res = await FILESubmit('/file/upload', formData, { onProgress, setCancel })
```

- `baseUrl` 来自 `UtilVar.baseUrl`（即 `VITE_APP_BASE_API` 环境变量）
- 自动在 header 注入 `token`（从 `localStorage` 读取）
- 响应拦截：`code===200` 时直接返回 `res.data`，并附加 `success: true`

### 4.2 `src/utils/request.js`（次实例，Ruoyi 风格）

- 请求走 Vite 代理前缀 `/dev-api` → `localhost:9090`
- token 字段为 `Authorization: Bearer xxx`（当前已注释，无鉴权状态）
- 适用于沿用 Ruoyi 风格 API 的场景

---

## 5. 全局组件

以下组件已在 `main.js` 全局注册，**无需局部 import**，直接在模板使用：

| 标签 | 文件 | 用途 |
|---|---|---|
| `<Echart>` | `components/echart/index.vue` | ECharts 图表封装 |
| `<ItemWrap>` | `components/item-wrap/item-wrap.vue` | 大屏面板卡片容器 |
| `<Message>` | `components/message/message.vue` | 消息提示 |
| `<Reacquire>` | `components/reacquire/reacquire.vue` | 数据重试/刷新 |

DataV 组件（`<dv-*>`）通过 `app.use(DataVVue3)` 全局注册，也无需局部引入。

---

## 6. 全局工具方法

在 `main.js` 挂载到 `app.config.globalProperties`，组件内可直接使用：

```js
// Options API
this.parseTime(date, '{y}-{m}-{d}')
this.handleTree(list, 'id', 'parentId')

// Composition API（需手动 import）
import { parseTime, handleTree } from '@/utils/ruoyi'
```

---

## 7. 路径别名

| 别名 | 解析目标 |
|---|---|
| `@` | `src/` |
| `~` | 项目根目录 |

---

## 8. 环境变量

| 变量 | 说明 |
|---|---|
| `VITE_APP_ENV` | 运行环境：`development` / `staging` / `production` |
| `VITE_APP_BASE_API` | API baseUrl，用于 `src/api/api.js` 的 baseUrl |

开发时代理配置：`/dev-api` → `http://localhost:9090`（见 `vite.config.js:35`）

---

## 9. 大屏布局说明

`src/views/indexs/index.vue` 按**九宫格布局**组合子组件：

```
┌──────────┬──────────────┬──────────┐
│ left-top │  center-top  │ right-top│
├──────────┼──────────────┼──────────┤
│left-center│             │right-center│
├──────────┤  (center)    ├──────────┤
│left-bottom│             │right-bottom│
└──────────┴──────────────┴──────────┘
                center-bottom
```

**新增大屏板块**：在 `src/views/indexs/` 新建 `.vue` 文件，在 `index.vue` 引入并布局。

---

## 10. 开发规范

- **文件搜索**：优先使用 `rg`（ripgrep），保持 diff 最小化
- **源码范围**：仅修改 `src/`，禁止触碰 `node_modules/`
- **安装依赖**：新增包或修改 lockfile 前**必须询问用户**
- **组件样式**：所有 Vue SFC 的 `<style>` 必须加 `scoped`
- **页面布局**：新视图需与 `src/views/` 下现有页面风格保持一致
- **Composition API**：新组件优先使用 `<script setup>` 语法
- **响应式布局**：大屏页面使用 `flexible.js` rem 自适应方案，使用 `drawMixin.js` 处理 resize

---

## 11. 后端项目（可直接读取源码）

**根路径**：`D:\enrionment\框架\后端服务`

### 11.1 技术栈

| 属性 | 值 |
|---|---|
| 框架 | Spring Boot 2.2.12.RELEASE |
| 语言 | Java 1.8 |
| 构建工具 | Maven（多模块 POM） |
| ORM | MyBatis + PageHelper |
| 连接池 | Druid 1.2.4 |
| 认证 | JWT（jjwt 0.9.1） |
| API 文档 | Springfox Swagger2 2.10.5 |
| JSON | FastJSON 1.2.74 |
| 定时任务 | Quartz（neu-quartz 模块） |
| Excel | Apache POI 4.1.2 |
| groupId | `com.neu` |
| artifactId | `carbon` |
| version | `3.3.0` |

### 11.2 模块目录

```
后端服务/
├── neu-admin/              # ★ 启动入口模块（Spring Boot 主类 NeuApplication.java）
│   └── src/main/java/com/neu/
│       ├── NeuApplication.java       # @SpringBootApplication 启动类
│       └── web/
│           ├── controller/           # 通用 Web 控制器（登录、文件下载等）
│           └── core/                 # 核心配置类
├── neu-framework/          # 框架核心（安全、缓存、拦截器、异常处理等）
├── neu-system/             # 系统管理（用户、角色、菜单、字典等）
├── neu-common/             # 通用工具（utils、annotation、core 基类等）
├── neu-common-biz/         # 通用业务（文件上传/下载等公共业务）
├── neu-quartz/             # 定时任务模块
├── neu-carbon-wms/         # WMS 仓储管理模块
├── neu-carbon-mes/         # MES 生产管理模块
├── neu-carbon-scm/         # SCM 供应链/采购管理模块
├── neu-carbon-bid/         # 招投标管理模块
├── neu-carbon-footprint/   # 碳足迹模块
├── neu-carbon-report/      # 报表模块
├── neu-carbon-service/     # 碳服务模块
├── neu-carbon-mapper/      # 公共 Mapper（跨模块 SQL 映射）
├── neu-carbon-chat/        # 聊天/消息模块
└── neu-carbon-distribusion/# 分销/配送模块
```

### 11.3 包路径约定

所有业务模块遵循统一包结构，以 `neu-carbon-wms` 为例：

```
com.neu.carbon.wms/
├── controller/   # @RestController，REST 接口层，映射 URL
├── domain/       # 实体类（@TableName / MyBatis 映射）
├── mapper/       # MyBatis Mapper 接口（对应 resources/mapper/**/*.xml）
└── service/      # 业务逻辑层（接口 + impl 实现）
```

> 其他业务模块路径规律：`com.neu.carbon.<模块名>/controller|domain|mapper|service`

### 11.4 如何定位接口实现

1. **按 URL 搜索**：

   ```powershell
   # 在后端根目录搜索 URL 路径片段
   rg "@RequestMapping|@GetMapping|@PostMapping" -r "D:\enrionment\框架\后端服务" -l
   rg "/wms/inventory" "D:\enrionment\框架\后端服务" -r
   ```

2. **按类名/方法名搜索**：

   ```powershell
   rg "class WmsInventoryController" "D:\enrionment\框架\后端服务" -r
   ```

3. **快速定位模块**：根据接口前缀判断所在模块

   | URL 前缀 | 模块 |
   |---|---|
   | `/wms/**` | `neu-carbon-wms` |
   | `/mes/**` | `neu-carbon-mes` |
   | `/scm/**` 或 `/procurement/**` | `neu-carbon-scm` |
   | `/bid/**` | `neu-carbon-bid` |
   | `/system/**` | `neu-system` |
   | `/common/**` | `neu-admin/web/controller` 或 `neu-common-biz` |

### 11.5 前后端联调说明

- 前端开发时，代理配置 `/dev-api` → `http://localhost:9090`
- 后端启动入口：`neu-admin` 模块下 `NeuApplication.java`，默认端口 `9090`
- 前端 `src/api/api.js` 使用 `VITE_APP_BASE_API`（development 环境为 `/dev-api`）向后端发起请求


