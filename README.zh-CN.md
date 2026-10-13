# Draft Paper

[中文] | [English](README.md)

Draft Paper 是一个轻量的单人在线草稿纸 / 速写板，使用 Vite + Vue 实现。这个仓库刻意保持纯前端：草稿数据保存在浏览器本地，支持多份草稿管理，并且不依赖后端或账号系统。

这份文档基于当前仓库中的实现编写，而不是过时的原型说明。

## 当前应用能力

- 工作台可创建、列出和恢复最近草稿
- 多份草稿各自保存在浏览器本地存储中的独立 autosave 键
- 支持草稿导入和导出（`.zxkjbb` 以及旧版 `.json` 数据）
- 白板支持绘制、橡皮擦、平移、缩放、旋转、撤销/重做和截图
- 提供性能模式，适合大面积书写或高频输入场景
- 支持浏览器语言切换（`zh-CN` / `en-US`）以及应用状态持久化

## 实际仓库工作流

项目的真实脚本和配置位于 `package.json` 与 `vite.config.js`：

- `npm install`
- `npm run dev` 启动开发服务器
- `npm run build` 生成生产构建到 `dist/`
- `npm run preview` 本地预览构建产物
- 开发服务器配置为 `host: '0.0.0.0'`，端口为 `5173`

这和当前仓库中实际存在的脚本一致；仓库中没有单独的后端、部署脚本或云同步服务。

## 快速开始

```bash
npm install
npm run dev
```

然后在浏览器中打开：

- `http://localhost:5173`

## 生产构建

```bash
npm run build
```

构建后可选本地预览：

```bash
npm run preview
```

## 数据与存储模型

当前应用是本地模式：

- 应用状态和草稿元数据写入 `localStorage`
- 每份草稿都有其独立的 autosave key，由 `src/App.vue` 管理
- 白板状态、导出/导入与自动保存逻辑位于 `src/components/WhiteboardView.vue`
- 草稿不会同步到远程服务器或用户账户

如果需要长期备份，请在编辑器界面中导出 `.zxkjbb` 文件。导入时支持 `.zxkjbb` 文件和旧版 JSON 草稿数据。

## 建议的开发流程

1. 运行 `npm install` 安装依赖。
2. 开发时执行 `npm run dev` 启动调试环境。
3. 在提交或发布前使用 `npm run build` 验证构建。
4. 只有在需要检查生成产物时才使用 `npm run preview`。

## 仓库结构

- `src/App.vue`：工作台、最近草稿、语言切换和草稿状态管理
- `src/components/WhiteboardView.vue`：白板交互、绘制、导入/导出、自动保存、截图和性能设置
- `vite.config.js`：Vite 开发服务器配置
- `index.html`：应用入口 HTML
- `package.json`：项目脚本和依赖

## 项目范围

这个仓库聚焦于单人本地草稿纸工作流，不包含多人协作、云同步或账号系统。

## 语言

- 中文
- [English](README.md)
