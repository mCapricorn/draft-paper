# Shared Whiteboard

这是从现有白板能力抽离出来的独立项目，目标是做一个可独立运行、可公开发布的共享草稿/白板应用。

## 已完成的改造

- 从主工程中抽离画板核心代码，保留独立运行能力
- 创建 GitHub public 仓库：https://github.com/mCapricorn/shared-whiteboard
- 增加轻量级的草稿名与性能模式入口，便于后续扩展为共享链接和协作场景
- 保留原有画板的本地恢复、导入导出与截图能力

## Features

- 画布绘制与橡皮擦
- 缩放 / 平移 / 旋转
- 撤销与重做
- 笔迹识别与形状优化
- 导入导出
- 自动保存
- 截图导出
- 高性能模式：在大面积书写时降低复杂重绘并减少接缝细节处理

## Quick Start

```bash
npm install
npm run dev
```

## Build

```bash
npm run build
```

## Performance notes

- 通过 coalesced redraw 和离屏绘制层控制，减少高频输入时的无效重绘。
- 在高性能模式下降低形状简化阈值和接缝细节，以提升大面积书写时的响应速度。
- 后续可继续扩展为 WebSocket 联动、实时共享、云端草稿同步等能力。
