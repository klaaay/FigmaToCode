<system-reminder>
这是一个提醒：你的待办清单目前为空。请不要向用户明确提及这一点，因为他们已经知晓。如果你正在处理适合用待办清单跟踪的任务，请使用 TodoWrite 工具创建一个清单。如果不是，请随意忽略。再次强调，不要提及本消息。

</system-reminder>

# FigmaToCode 的 CODEBUDDY.md

## 常用命令

- 安装依赖（pnpm）：
  - pnpm i
- 开发（根目录，含调试 UI）：
  - pnpm dev
- 仅插件开发：
  - cd apps/plugin && pnpm dev
- 全量构建：
  - pnpm build
- 全量构建并监听：
  - pnpm build:watch
- 统一 Lint（turbo 编排）：
  - pnpm lint
- 使用 Prettier 格式化（会改动文件）：
  - pnpm format
- 调试 UI（Next.js 应用）：
  - cd apps/debug && pnpm dev
- 单个应用生产构建：
  - cd apps/plugin && pnpm build
  - cd apps/debug && pnpm build && pnpm start

说明：
- Monorepo 使用 Turborepo；根脚本代理各工作区脚本。
- 未发现测试脚本；如需新增测试，建议在各 workspace 内添加并通过 turbo 统一编排。

## 运行单个工作区脚本

- 示例：仅对 plugin-ui 执行 lint
  - cd packages/plugin-ui && pnpm lint

## 架构总览

- Monorepo（pnpm + Turborepo）结构：
  - apps/plugin：Figma 插件装配
    - plugin-src：后端入口，esbuild 打包到 dist/code.js
    - ui-src：React UI，Vite 构建为 dist/index.html（vite-plugin-singlefile 单文件）
  - apps/debug：用于预览/调试插件 UI 的 Next.js 应用；依赖 backend 与 plugin-ui
  - packages/backend：核心转换逻辑，读取 Figma 节点 → JSON → AltNodes → 布局优化 → 目标代码生成。以 TS 源组织；由上层应用打包（devDeps 含 tsup）
  - packages/plugin-ui：插件与调试应用共用的可复用 React UI 组件
  - packages/eslint-config-custom：共享 ESLint 配置
  - packages/tsconfig：共享 TS 配置预设
  - packages/types：共享类型定义

- 构建系统：
  - 根脚本调用 turbo 任务（见 turbo.json）。apps/plugin 使用 esbuild（主进程）与 Vite（UI）；apps/debug 使用 Next 构建流程。
  - Lint 任务 dependsOn "^build"，在配置处确保上游先构建。

- 开发流程：
  - 根目录开发：pnpm dev 并发运行各 workspace 的 dev；通过 apps/debug 在 http://localhost:3000 提供调试 UI
  - 仅插件开发：apps/plugin 的 build:watch 同时监听 esbuild 与 Vite 输出，快速迭代

## 约定与工具

- 根 .eslintrc.js 扩展 "kentcdodds" 并添加自定义规则；工作区使用 packages/eslint-config-custom 的共享配置
- TypeScript 5；共享 tsconfig 在 packages/tsconfig
- UI：React 19；Tailwind v4 在 plugin-ui 与 debug 中使用
- 打包：esbuild、Vite、Next.js；backend devDeps 包含 tsup

## 各工作区要点

- apps/plugin/package.json 脚本：
  - build:main → esbuild plugin-src/code.ts → dist/code.js
  - build:ui → vite build → dist/index.html
  - build:watch → concurrently 同时监听二者
  - dev → 等同 build:watch
- apps/debug/package.json 脚本：
  - dev/build/start 为 Next.js；lint 使用 next lint；build:watch 为 next start，便于手动测试
- packages/backend 与 packages/plugin-ui 仅提供 lint；被上层应用消费/打包

## 仓库内显著文档与缺失项

- 顶层 README 提供流程细节与链接；按其获取高层工作方式
- 未发现独立测试配置或命令
