# Retro CRT Simulator

一个基于 Vue 3 和 Vite 的复古 CRT 终端模拟器。你可以调整文字、字体、颜色、扫描线、曲率、噪点等参数，并导出当前画面。

## 功能

- 实时编辑终端文字内容
- 切换像素字体和文字对齐方式
- 调整辉光、色差、噪点和扫描线效果
- 切换曲面屏、暗角、Dreamcore 扭曲、Camcorder 模式
- 自定义前景色和背景色
- 导出当前 CRT 画面为 PNG 图片

## 技术栈

- Vue 3
- Vite 5
- @iconify/vue
- html-to-image

## 本地开发

安装依赖：

```bash
npm install
```

启动开发环境：

```bash
npm run dev
```

构建生产版本：

```bash
npm run build
```

本地预览生产构建：

```bash
npm run preview
```

## GitHub Pages 部署

这个项目当前使用 GitHub Actions 工作流部署，配置文件在 [.github/workflows/deploy.yml](.github/workflows/deploy.yml)。

正确的发布方式：

1. 把代码 push 到 `main` 分支。
2. 打开仓库的 GitHub Settings。
3. 进入 Pages。
4. 在 Build and deployment 的 Source 中选择 GitHub Actions。
5. 等待 Actions 里的 `Deploy to GitHub Pages` 工作流完成。

## 项目结构

```text
.
├── index.html
├── package.json
├── vite.config.js
├── src
│   ├── App.vue
│   ├── constants.js
│   ├── main.js
│   ├── assets
│   │   └── style.css
│   └── components
│       ├── Controls.vue
│       └── CRTMonitor.vue
└── .github
    └── workflows
        └── deploy.yml
```