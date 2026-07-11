# MD2Pic

**Markdown 驱动的一键文字转社交媒体图片工具**

> 基于 [oneimg](https://github.com/byodian/oneimg) 深度改造，融合 doocs/md 编辑器、Pretext 排版引擎，以及 Viral Writer / Taste Skill / Ian Xiaohei 三大 AI Skill。

支持微信公众号长图、小红书、Instagram、抖音等平台。  
Mac + Windows 桌面端 + Web。  
部署极简，本地优先，AI 能力通过 Claude Code / Codex Skill 嫁接。

---

## ✨ 核心特性

- **Markdown First**：完整支持 Markdown 输入与编辑（融合 doocs/md）
- **固定布局 + 智能排版**：基于 oneimg 模板系统 + Pretext 精确文本测量
- **多平台适配**：公众号 / 小红书 / Instagram / 抖音 / 自定义
- **标准 Theme JSON**：主题可导入导出分享（.mdjson）
- **AI 增强**（Skill 模式）：
  - **Viral Writer**：输入主题 → 完整病毒文案 + 5 标题 + 配图 prompt
  - **Taste Skill**：AI 一键优化排版与设计品味
  - **Ian Xiaohei**：生成小黑怪诞正文配图
- **一键导出**：长图 / 切片 PNG / WebP / PDF
- **跨平台桌面端**：Tauri 2（轻量、原生性能）

---

## 🚀 快速开始（即将完善）

```bash
# 克隆
git clone https://github.com/0xRick0x/MD2Pic.git
cd MD2Pic

# 安装（pnpm）
pnpm install

# 开发
pnpm dev

# Docker
docker run -p 3000:3000 0xrick0x/md2pic
```

---

## 📁 项目结构（Monorepo）

```
MD2Pic/
├── apps/
│   ├── desktop/          # Tauri 2 桌面端
│   └── web/              # Next.js / Vite Web 端
├── packages/
│   ├── editor/           # Markdown 编辑器 (doocs/md 改造)
│   ├── renderer/         # 渲染引擎 (oneimg 基座)
│   ├── layout/           # 布局引擎
│   ├── typography/       # Pretext 排版
│   ├── theme/            # Theme Engine
│   ├── template/         # Template Engine
│   ├── export/           # 导出引擎
│   ├── platform/         # 平台适配
│   ├── parser/           # Markdown AST
│   ├── markdown/         # Markdown 扩展
│   └── shared/           # 领域模型 / 类型 / Skill Runtime
├── skills/
│   ├── viral-writer/     # Viral Writer Skill
│   ├── taste/            # Taste Skill
│   └── xiaohei/          # Ian Xiaohei Illustrations
├── themes/               # 标准 .mdjson 主题库
├── templates/            # 布局模板
├── plugins/              # 插件系统
├── docs/                 # 架构文档
├── tests/
└── scripts/
```

---

## 🛠 技术栈

- **Desktop**: Tauri 2 + React + TypeScript + Vite
- **Web**: Next.js 15 / Vite + React + TypeScript
- **Editor**: 改造自 doocs/md + Tiptap
- **Renderer**: 基于 byodian/oneimg + html2canvas
- **Typography**: @chenglou/pretext
- **State**: Zustand
- **Style**: Tailwind CSS + shadcn/ui
- **Package**: pnpm monorepo + Turborepo (planned)
- **Test**: Vitest + Playwright
- **AI**: 纯 Skill 模式（Claude Code / Codex 友好）

---

## 📋 开发任务

完整任务清单见 [tasks.md](./tasks.md)（严格依据 Master Requirement Document 生成）。

当前状态：Phase 0 进行中。

---

## 🤝 贡献

欢迎 PR 与 Issue。请先阅读 `docs/contribution-guide.md`（即将补充）。

---

## 📄 License

MIT

---

**Made with ❤️ for Chinese content creators**  
Repo: https://github.com/0xRick0x/MD2Pic
