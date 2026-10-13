# Draft Paper

[中文] | [English](README.md)

Draft Paper 是一个轻量的单人在线草稿纸 / 速写板，使用 Vite + Vue 实现。这个仓库保持纯前端架构，草稿数据保存在浏览器本地，并支持多份草稿管理，不依赖后端或账号系统。

这份中文文档基于当前仓库的实际实现整理，重点说明当前项目的真实使用方式与结构。

## 当前应用功能

- 工作台支持创建、查看和恢复最近草稿
- 每份草稿都在浏览器本地存储中保留独立的 autosave 键
- 支持草稿导入和导出，兼容 `.zxkjbb` 文件和旧版 `.json` 数据
- 白板支持绘制、橡皮擦、平移、缩放、旋转、撤销与重做，并可进行截图
- 提供性能模式，适合大面积书写和高频输入场景
- 支持浏览器语言切换（`zh-CN` / `en-US`）以及应用状态持久化

## 实际工作流

项目的真实脚本和配置位于 `package.json` 与 `vite.config.js`：

- `npm install`
- `npm run dev` 启动开发服务器
- `npm run build` 生成生产构建，输出到 `dist/`
- `npm run preview` 本地预览构建结果
- 开发服务器配置为 `host: '0.0.0.0'`，端口为 `5173`

这与当前仓库实际存在的脚本一致；仓库中没有单独的后端、部署脚本或云同步服务。

## 快速开始

```bash
npm install
npm run dev
```

随后在浏览器中打开：

- `http://localhost:5173`

## 生产构建

```bash
npm run build
```

如果需要本地检查构建效果，可继续执行：

```bash
npm run preview
```

## 数据与存储方式

当前应用采用本地模式：

- 应用状态和草稿元数据写入 `localStorage`
- 每份草稿都由 `src/App.vue` 管理独立的 autosave key
- 白板状态、导出 / 导入 与自动保存逻辑位于 `src/components/WhiteboardView.vue`
- 草稿不会同步到远程服务器或用户账户

如果需要长期备份，请在编辑器界面中导出 `.zxkjbb` 文件。导入时支持 `.zxkjbb` 文件和旧版 JSON 草稿数据。

## 推荐开发流程

1. 运行 `npm install` 安装依赖。
2. 开发时执行 `npm run dev` 启动本地调试环境。
3. 在提交或发布前使用 `npm run build` 验证构建结果。
4. 只有在需要检查产物时，才使用 `npm run preview`。

## 仓库结构

- `src/App.vue`：工作台、最近草稿、语言切换与草稿状态管理
- `src/components/WhiteboardView.vue`：白板交互、绘制、导入 / 导出、自动保存、截图以及性能设置
- `vite.config.js`：Vite 开发服务器配置
- `index.html`：应用入口 HTML
- `package.json`：项目脚本与依赖声明

## 项目范围

这个仓库聚焦于单人本地草稿纸工作流，不包含多人协作、云同步或账号系统。

## 语言切换

- 中文
- [English](README.md)
