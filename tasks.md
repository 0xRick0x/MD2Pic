# MD2Pic 开发任务清单（tasks.md）

> 依据文档：《MD2Pic — Master Requirement Document (MRD) v1.0》（2026-07-11）
> 本任务清单严格依据 MRD 全部 12 章内容生成，不新增 MRD 未提及的功能、模块或技术选型。
> 任务模板依据 **Chapter 12.8 — Task Template** 定义。
> 阶段划分依据 **Chapter 03（六层架构）**、**Chapter 04（开源集成顺序）**、**Chapter 06（目录结构）**、**Chapter 07（渲染流水线）**、**Chapter 08（领域模型）**、**Chapter 09（技术蓝图）**、**Chapter 10（Markdown & Theme 规范）**。
> 所有任务必须遵循 **Chapter 11（工程契约）** 与 **Chapter 12（AI 开发工作流）** 中定义的开发生命周期：Requirement → Research → Architecture Review → Planning → Task Generation → Implementation → Verification → Review → Documentation → Merge。

---

## 全局验收基线（适用于所有任务）

以下基线来自 Chapter 11 / Chapter 12，是每个任务在标记为 Done 前必须满足的通用条件（不重复列在每个任务中，但视为隐含验收项）：

- Build 成功；Lint 通过；类型检查通过（Chapter 11.13, 11.20）
- 单元测试 / 集成测试 / 快照测试按模块要求通过（Chapter 11.12, 9.19）
- 未破坏既有 Theme / Template / Markdown / Skill / Plugin / Export 格式的向后兼容性（Chapter 11.11）
- 未引入循环依赖、跨层依赖或职责混用（Chapter 9.5, 11.19 反模式清单）
- 相关文档（架构 / API / README / 迁移指南）已同步更新（Chapter 11.21, 12.12）
- 未硬编码密钥，未记录用户隐私内容日志（Chapter 11.15, 11.18）
- 完成自查清单（Chapter 11.20）：是否遵循架构 / 是否可复用现有模块 / 是否引入技术债 / 是否可独立测试

---

## Phase 0 — 项目基础设施与仓库结构

**对应章节**：Chapter 06（项目目录结构）、Chapter 09.2（Monorepo 策略）、Chapter 09.6（技术栈选型）

**阶段目标**：建立符合规范的 Monorepo 骨架，使目录结构本身即可表达系统架构（Chapter 6.15）。

### Task 0.1 — 初始化 Monorepo 骨架

- **Purpose**：建立 `apps/ packages/ skills/ plugins/ themes/ templates/ examples/ docs/ scripts/ public/ tests/ tools/` 顶层目录结构（Chapter 6.2），作为后续所有开发的基础。
- **Dependencies**：无
- **Files**：仓库根目录；`pnpm-workspace.yaml`；各顶层目录及占位 `README.md`
- **Acceptance Criteria**：
  1. 目录结构与 Chapter 6.2 定义的树完全一致，无多余或缺失的顶层目录。
  2. `packages/` 下预留 `editor/ renderer/ layout/ typography/ parser/ markdown/ export/ theme/ template/ assets/ platform/ shared/` 十二个空包（Chapter 6.4）。
  3. `apps/` 下预留 `desktop/` 与 `web/` 两个空应用入口（Chapter 6.3）。
  4. 每个目录包含说明其单一职责的 README（Chapter 6.14 三问：是否单一职责 / 是否易理解 / 是否可独立演进）。
- **Tests**：目录结构静态校验脚本（检查必需目录是否存在）。
- **Estimated Complexity**：S
- **Status**：Not Started

### Task 0.2 — 配置技术栈与工具链

- **Purpose**：按 Chapter 9.6 技术栈表配置 Tauri 2、React、TypeScript、Vite、pnpm、Zustand、TailwindCSS、shadcn/ui、Vitest、Playwright、ESLint、Prettier。
- **Dependencies**：Task 0.1
- **Files**：`package.json`、`pnpm-workspace.yaml`、`tsconfig.base.json`、`vite.config.ts`、`tailwind.config.ts`、`.eslintrc`、`.prettierrc`、`apps/desktop/src-tauri/`
- **Acceptance Criteria**：
  1. 桌面端运行时为 Tauri 2（Chapter 9.7：Rust 后端、更小体积、原生性能）。
  2. 前端框架为 React + TypeScript，构建工具为 Vite，包管理器为 pnpm（Chapter 9.6）。
  3. 状态管理使用 Zustand，样式使用 TailwindCSS，组件库使用 shadcn/ui（Chapter 9.6）。
  4. 测试框架为 Vitest（单元/集成）+ Playwright（端到端）（Chapter 9.19）。
  5. `pnpm install && pnpm build` 在空骨架下可成功执行。
- **Tests**：CI 中执行 `pnpm build`、`pnpm lint`、`pnpm test` 均返回成功（即使测试集为空）。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 0.3 — 建立依赖方向静态校验

- **Purpose**：落地 Chapter 9.5 依赖规则（Application → Presentation → Packages，禁止反向/跨层依赖），防止架构腐化。
- **Dependencies**：Task 0.1, 0.2
- **Files**：`tools/dependency-lint/`、ESLint 自定义规则或 `dependency-cruiser` 配置
- **Acceptance Criteria**：
  1. 存在自动化规则禁止 Renderer → Editor、Theme → Markdown、Exporter → AI、Skill → Canvas、Renderer → Platform（Chapter 9.5 明确列出的禁止依赖）。
  2. 规则接入 CI，违规时构建失败。
  3. 提供一条违规示例并验证规则能够拦截。
- **Tests**：单元测试验证违规导入被 lint 规则捕获。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 1 — 领域模型（Domain Model）与 Schema 定义

**对应章节**：Chapter 08（领域模型规范）、Chapter 10（Markdown & Theme 规范）、Final Section 第 7 条（Schema First）

**阶段目标**：先定义 Schema，后有实现（Schema First，Final Section #7）。领域模型是 SSOT，所有模块必须通过其通信（Chapter 8.19）。

### Task 1.1 — 定义核心领域对象类型（`packages/shared`）

- **Purpose**：实现 Chapter 8.2 列出的核心实体的类型定义：`Project, Document, Block, Theme, Template, Illustration, Asset, Platform, Skill, RenderObject, ExportJob`。
- **Dependencies**：Task 0.1, 0.2
- **Files**：`packages/shared/src/types/`
- **Acceptance Criteria**：
  1. 每个实体的字段与 Chapter 8.3–8.14 中定义的字段一一对应，不增不减。
  2. `Document` 字段包含：`id, title, metadata, blocks, theme, template, assets, illustrations, platform, settings, version, createdAt, updatedAt`（Chapter 8.4）。
  3. `RenderObject` 字段包含：`id, type, bounds, style, children, asset, metadata`，且标记为不可变（Chapter 8.13）。
  4. `Block` 支持类型集合：`Heading, Paragraph, Image, Quote, Code, Table, Math, Mermaid, List, Callout, Divider, Button, Card, Columns, Embed`（Chapter 8.5）。
  5. 类型定义不包含任何渲染逻辑或业务逻辑（`shared` 包仅含工具/类型，Chapter 6.4 shared 部分）。
- **Tests**：类型编译通过；针对每个实体编写最小合法示例对象并通过类型检查。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 1.2 — 定义 Document Schema 与 FrontMatter 规范

- **Purpose**：实现 Chapter 10.4–10.5 的 YAML FrontMatter 结构与必填字段校验规则。
- **Dependencies**：Task 1.1
- **Files**：`packages/markdown/src/frontmatter/`
- **Acceptance Criteria**：
  1. 支持字段：`title, subtitle, author, theme, template, platform, language, tags, cover, illustration, created, updated, version`（Chapter 10.4 示例）。
  2. 必填字段校验：`title, theme, template, platform, version`（Chapter 10.5），缺失时校验失败并给出明确错误。
  3. FrontMatter 解析结果可序列化为 JSON（Chapter 10.4 "All metadata must remain serializable"）。
- **Tests**：合法/非法 FrontMatter 样例各至少 3 组，覆盖必填字段缺失、类型错误、多余未知字段（应被安全忽略或警告，不崩溃）。
- **Estimated Complexity**：S
- **Status**：Not Started

### Task 1.3 — 定义 Theme Schema

- **Purpose**：实现 Chapter 8.7 与 Chapter 10.12–10.14 的 Theme 数据结构。
- **Dependencies**：Task 1.1
- **Files**：`packages/theme/src/schema/theme.schema.json`、对应 TypeScript 类型
- **Acceptance Criteria**：
  1. Theme 字段覆盖：`name, version, colors, typography, spacing, radius, shadow, background, platform`（Chapter 10.14）。
  2. Theme 仅为 JSON 数据，不含 JavaScript / TypeScript / React 组件（Chapter 10.12 "Never JavaScript. Never TypeScript. Never React Components."）。
  3. Theme 不存储内容（content）与布局（layout）信息（Chapter 8.7）。
  4. Schema 校验器可拒绝包含可执行代码字段的 Theme 文件。
- **Tests**：使用示例 Theme（如 `glass-dark`）通过 Schema 校验；注入可执行代码字段应被拒绝。
- **Estimated Complexity**：S
- **Status**：Not Started

### Task 1.4 — 定义 Template Schema

- **Purpose**：实现 Chapter 8.8 与 Chapter 10.15 的 Template 数据结构。
- **Dependencies**：Task 1.1
- **Files**：`packages/template/src/schema/template.schema.json`、`layout.schema.json`
- **Acceptance Criteria**：
  1. Template 字段覆盖：`Header, Body, Footer, Hero, Sidebar, Illustration Slot, Background, Card, Grid, Safe Area`（Chapter 8.8）。
  2. Template 不包含颜色（colors）字段（Chapter 10.15 "Templates never contain colors"）。
  3. Template 不包含渲染逻辑，仅为数据驱动结构（Chapter 8.8, 9.12）。
- **Tests**：Schema 校验拒绝包含 `color`/`colour` 字段的 Template 文件。
- **Estimated Complexity**：S
- **Status**：Not Started

### Task 1.5 — 定义 Skill / Platform / ExportJob Schema

- **Purpose**：实现 Chapter 8.11（Skill）、8.12（Platform）、8.14（ExportJob）的数据结构与统一接口。
- **Dependencies**：Task 1.1
- **Files**：`packages/shared/src/types/skill.ts`、`platform.ts`、`export-job.ts`
- **Acceptance Criteria**：
  1. Skill 接口输入为 `Document`，输出为 `Document Patch`，不得直接操作 Renderer 或 Canvas（Chapter 8.11）。
  2. Platform 字段包含：`Canvas Size, Safe Area, Export Scale, Typography Scale, Recommended Template`（Chapter 8.12），不含渲染逻辑。
  3. ExportJob 字段包含：`Platform, Resolution, Scale, Format, Quality, Watermark, Output Path, Timestamp`，不持有内容数据（Chapter 8.14）。
  4. Skill 统一接口另需覆盖 Chapter 4.10：`Metadata, Input Schema, Output Schema, Configuration, Prompt, Version, Capabilities`。
- **Tests**：类型编译通过；构造一个模拟 Skill 输出 Patch 并验证其不包含渲染字段。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 1.6 — 文档校验管线（Validation）

- **Purpose**：实现 Chapter 10.19 定义的文档渲染前校验：FrontMatter、必填字段、不支持的 Block、缺失资源、非法 Theme/Template。
- **Dependencies**：Task 1.2, 1.3, 1.4
- **Files**：`packages/markdown/src/validation/`
- **Acceptance Criteria**：
  1. 校验覆盖 Chapter 10.19 列出的全部五类问题。
  2. 校验失败时渲染流程不得启动（"Rendering should never begin with an invalid document."）。
  3. 校验错误信息可定位到具体字段/行号。
- **Tests**：为每一类校验失败场景各提供一个用例，断言渲染管线在校验失败时被阻止。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 2 — Markdown 解析与 AST（`packages/parser`, `packages/markdown`）

**对应章节**：Chapter 07.4–07.6（Parser / AST / Semantic Analyzer）、Chapter 10（Markdown 规范与扩展块）

### Task 2.1 — 实现 CommonMark + GFM 基础解析器

- **Purpose**：实现 Chapter 10.2 规定的标准（CommonMark + GitHub Flavored Markdown）解析，输出 Chapter 7.5 定义的 Markdown AST。
- **Dependencies**：Task 1.1, 1.6
- **Files**：`packages/parser/src/`
- **Acceptance Criteria**：
  1. 支持解析 FrontMatter、Metadata、Code Block、Mermaid、Math、Table（Chapter 7.4 职责列表）。
  2. 输出 AST 节点类型覆盖：`Heading, Paragraph, Quote, List, Image, Table, Code, Mermaid, Callout`（Chapter 7.5 示例树）。
  3. Parser 不执行任何布局计算（"The parser should never perform layout calculations."，Chapter 7.4）。
  4. AST 为下游模块唯一消费的规范表示，禁止任何模块重新解析 Markdown（Chapter 7.5 "No module should re-parse Markdown."）。
- **Tests**：针对每种 Block 类型提供解析用例；提供一个综合文档的黄金 AST 快照测试。
- **Estimated Complexity**：L
- **Status**：Not Started

### Task 2.2 — 实现 MD2Pic 扩展块解析

- **Purpose**：实现 Chapter 10.7 保留的自定义扩展块语法：`::hero ::card ::callout ::gallery ::timeline ::metric ::tweet ::social ::illustration ::ai`。
- **Dependencies**：Task 2.1
- **Files**：`packages/markdown/src/extensions/`
- **Acceptance Criteria**：
  1. 全部 10 个保留扩展块均可被解析为结构化节点，语法保持人类可读（Chapter 10.7 "Extensions should remain human-readable."）。
  2. `::illustration` 块字段覆盖 `style, position, size, prompt`（Chapter 10.8 示例）。
  3. `::ai` 块字段覆盖 `provider, skill, action`（Chapter 10.9 示例）。
  4. `::social` 块字段覆盖 `platform, theme, ratio`（Chapter 10.10 示例）。
  5. Callout 语义类型支持 `Note, Info, Warning, Danger, Success, Tip`（Chapter 10.11），且 Callout 保持语义化，外观由 Theme 控制（不在解析阶段决定颜色）。
- **Tests**：每个扩展块至少一个解析用例 + 一个含全部扩展块的综合文档快照测试。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 2.3 — 实现语义分析器（Semantic Analyzer）

- **Purpose**：实现 Chapter 7.6 语义分析，将语法节点映射为语义角色。
- **Dependencies**：Task 2.1
- **Files**：`packages/parser/src/semantic/`
- **Acceptance Criteria**：
  1. 至少实现 Chapter 7.6 示例映射：`Heading → Title`、`Paragraph → Description`、`Quote → Highlight`、`List → Checklist`（按上下文可判定时）。
  2. 输出结构为独立的 Semantic Tree，供 Content Intelligence 与 Layout Engine 使用（Chapter 7.6 "Semantic information helps AI and layout decisions."）。
  3. 该阶段不修改原始 AST，只产出附加语义信息。
- **Tests**：给定含标题/段落/引用/列表的文档，验证语义标注结果符合预期映射。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 2.4 — 组装 Document Model

- **Purpose**：将 AST + 语义树 + FrontMatter 组装为 Chapter 7.7 / Chapter 8.4 定义的 `Document` 对象。
- **Dependencies**：Task 2.1, 2.2, 2.3, Task 1.1
- **Files**：`packages/parser/src/document/`
- **Acceptance Criteria**：
  1. 输出对象字段与 Chapter 8.4 `Document` 完全一致。
  2. Document 在渲染期间保持不可变，每次渲染需创建快照（Chapter 8.4 "Document is immutable during rendering."）。
  3. Document 贯穿整个渲染管线生命周期，其余对象均为临时产物（Chapter 8.17）。
- **Tests**：对同一 Markdown 输入两次组装，验证快照互不干扰（不可变性测试）。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 3 — 渲染引擎基础：集成 oneimg（`packages/renderer`）

**对应章节**：Chapter 03.3 / Chapter 04.3（oneimg 职责与保留/替换/扩展策略）、Chapter 07.14（Canvas Renderer）

> 依据 Chapter 2.5 / 4.3："Renderer 应被增强，不应被重写（Never Rewrite）。"

### Task 3.1 — 接入 oneimg 作为渲染基础设施

- **Purpose**：将 oneimg（https://github.com/byodian/oneimg）作为渲染引擎基础集成进 `packages/renderer`，保留其渲染引擎、模板引擎、主题引擎、Canvas 层、导出逻辑、渲染性能优化（Chapter 4.3 "Components To Preserve"）。
- **Dependencies**：Task 0.2
- **Files**：`packages/renderer/src/oneimg-adapter/`
- **Acceptance Criteria**：
  1. oneimg 原始渲染引擎、Canvas 绘制、导出管线保持不变，未被重写（Chapter 4.3 "Never Rewrite"、Chapter 2.5）。
  2. oneimg 原有的手动内容编辑工作流、内容输入界面被标记为待替换但暂不删除渲染能力（Chapter 4.3 "Components To Replace" 仅针对编辑体验，不含渲染）。
  3. 集成方式采用 Wrapper/Adapter，避免深度修改 oneimg 源码（Chapter 4.2 "Minimize Fork Maintenance"）。
- **Tests**：使用一个最小 RenderObject 通过 Adapter 成功生成一张 PNG，验证渲染基础可用。
- **Estimated Complexity**：L
- **Status**：Not Started

### Task 3.2 — 定义并实现 Render Tree 构建器

- **Purpose**：实现 Chapter 7.13 定义的 Render Tree（不含 Markdown、不含 AI 逻辑，仅含可绘制对象）。
- **Dependencies**：Task 3.1, Task 1.1
- **Files**：`packages/renderer/src/render-tree/`
- **Acceptance Criteria**：
  1. Render Tree 节点结构包含 `Bounds, Style, Children, Assets, Metadata`（Chapter 7.13）。
  2. Render Tree 不包含 Markdown 文本源码或 AI 决策逻辑（Chapter 7.13 明确禁止）。
  3. Renderer 仅消费 Render Tree / RenderObject，不直接读取 Document 或 Markdown（Chapter 8.13 "Renderer consumes only RenderObjects."）。
- **Tests**：构造一个 Render Tree 并断言渲染器输出与树结构一一对应（结构化快照测试）。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 3.3 — 实现 Canvas Renderer 核心绘制能力

- **Purpose**：实现 Chapter 7.14 列出的绘制职责：绘制文本、图片、背景、边框、插画、装饰。
- **Dependencies**：Task 3.2
- **Files**：`packages/renderer/src/canvas/`
- **Acceptance Criteria**：
  1. 支持绘制：`text, images, background, borders, illustrations, decorations`（Chapter 7.14 全部六项）。
  2. 渲染具备确定性：相同输入产生相同输出（Chapter 7.14 "The same input should always generate the same output."，呼应 Final Section #14 Deterministic Rendering）。
- **Tests**：同一 Render Tree 渲染两次，像素级对比（Golden Image Test）结果一致；每类绘制对象至少一个快照测试。
- **Estimated Complexity**：L
- **Status**：Not Started

### Task 3.4 — 渲染阶段错误恢复机制

- **Purpose**：实现 Chapter 7.18 定义的分级错误恢复策略。
- **Dependencies**：Task 3.3
- **Files**：`packages/renderer/src/error-recovery/`
- **Acceptance Criteria**：
  1. Parser Error → 停止解析（Stop Parsing）。
  2. Typography Error → 回退默认字体（Fallback Font）。
  3. Illustration Error → 继续渲染（Continue Rendering）。
  4. Skill Error → 跳过该 Skill（Skip Skill）。
  5. Renderer Error → 返回诊断信息（Return Diagnostic），不导致应用崩溃。
  6. 任一可选模块（Skill/Plugin/AI Provider）失败不得导致整体渲染流程崩溃（Chapter 7.18 末段 / Chapter 11.14）。
- **Tests**：针对上述 5 类错误各构造一个故障注入测试，验证系统按预期降级而非崩溃。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 3.5 — 渲染性能基线验证

- **Purpose**：验证渲染延迟满足 Chapter 7.19 / Chapter 11.16 性能目标。
- **Dependencies**：Task 3.3
- **Files**：`tests/renderer/performance/`
- **Acceptance Criteria**：
  1. 小文档渲染 < 100ms；中等文档 < 300ms；大文档 < 800ms（Chapter 7.19，与 Chapter 11.16 一致）。
  2. Preview 刷新 < 100ms（Chapter 11.16）。
  3. 提供基准测试脚本可复现上述指标。
- **Tests**：性能基准测试（不同规模文档各一组），CI 中输出耗时报告；超出阈值时构建告警。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 4 — Markdown 编辑器集成：doocs/md（`packages/editor`）

**对应章节**：Chapter 03.3 / Chapter 04.4（doocs/md 职责与集成要求）

### Task 4.1 — 集成 doocs/md 作为唯一编辑器

- **Purpose**：将 doocs/md（https://github.com/doocs/md）集成为 MD2Pic 唯一 Markdown 编辑器，逐步移除 oneimg 原有编辑体验（Chapter 3.3 doocs/md 部分，Chapter 4.4）。
- **Dependencies**：Task 0.2
- **Files**：`packages/editor/src/`
- **Acceptance Criteria**：
  1. 编辑器提供 Markdown 编辑、解析、预览、导入、导出、剪贴板支持（Chapter 4.4 "Responsibilities"）。
  2. 编辑器与渲染器完全独立：编辑器不包含渲染逻辑，渲染器不包含编辑逻辑（Chapter 2.6，Chapter 4.4 "Editor and renderer must remain independent."）。
  3. 编辑器与渲染器之间仅通过结构化 Markdown 数据通信，渲染器不依赖编辑器内部实现（Chapter 4.4 "Required Integration"）。
- **Tests**：编辑器单独运行的单元测试（不依赖渲染器）；渲染器单独运行的单元测试（不依赖编辑器）；集成测试验证二者通过 Markdown 数据正确对接。
- **Estimated Complexity**：L
- **Status**：Not Started

### Task 4.2 — 编辑器多平台预设适配

- **Purpose**：实现 Chapter 4.4 "Required Enhancements"：为 Twitter/X、Threads、Instagram、Rednote、WeChat、Telegram、LinkedIn 提供发布预设支持入口。
- **Dependencies**：Task 4.1
- **Files**：`packages/editor/src/platform-presets/`
- **Acceptance Criteria**：
  1. 编辑器 UI 支持选择上述 7 个平台预设中的任意一个（与 Chapter 4.4 列出的平台一致，不多不少）。
  2. 预设选择仅影响导出/发布相关配置，不影响 Markdown 源内容本身（保持 Markdown First 原则，Chapter 2.2）。
- **Tests**：为 7 个平台预设各编写一个选择/切换用例，验证 Markdown 内容不因切换预设而改变。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 5 — Theme 与 Template 引擎（`packages/theme`, `packages/template`）

**对应章节**：Chapter 07.11（Theme Engine）、Chapter 08.7–08.8、Chapter 09.11–09.12、Chapter 10.12–10.17、Chapter 06.6–06.7（目录规范）

### Task 5.1 — 实现 Theme Engine

- **Purpose**：实现 Chapter 7.11 定义的 Theme Engine，负责应用颜色、排版、间距、圆角、阴影、背景、装饰。
- **Dependencies**：Task 1.3
- **Files**：`packages/theme/src/engine/`
- **Acceptance Criteria**：
  1. 职责覆盖 Chapter 7.11 全部七项：`color, typography, spacing, radius, shadow, background, decoration`。
  2. Theme Engine 不改变布局（"The Theme Engine never changes layout."，Chapter 7.11）。
  3. Theme 目录遵循 Chapter 6.6 结构：`theme.json, preview.png, fonts/, assets/`。
- **Tests**：应用同一 Theme 于两个不同 Layout Tree，验证布局结构未被 Theme Engine 修改，仅样式变化。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 5.2 — 实现 Theme 平台覆盖（Platform Override）

- **Purpose**：实现 Chapter 10.16 定义的平台级 Theme 覆盖规则。
- **Dependencies**：Task 5.1
- **Files**：`packages/theme/src/platform-override/`
- **Acceptance Criteria**：
  1. 支持按平台覆盖排版/间距/插画尺寸等（Chapter 10.16 示例：Twitter 标题增大、Instagram 间距增大、LinkedIn 插画缩小）。
  2. 平台规则不得修改 Markdown 源内容（Chapter 10.16 "Platform rules never modify Markdown."）。
- **Tests**：同一 Document 在不同 Platform 覆盖下渲染，验证 Markdown 输入保持不变、仅样式输出不同。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 5.3 — 实现 Template Engine

- **Purpose**：实现 Chapter 9.12 / Chapter 8.8 定义的 Template 组合能力：Header、Body、Footer、Slots、Cards、Background、Illustrations。
- **Dependencies**：Task 1.4
- **Files**：`packages/template/src/engine/`
- **Acceptance Criteria**：
  1. Template 定义 `Header, Body, Footer, Image Slots, Illustration Slots, Background, Typography Rules, Safe Area`（Chapter 4.12）。
  2. Template 不包含业务逻辑（Chapter 4.12 "Templates should not contain business logic."）。
  3. Template 目录遵循 Chapter 6.7 结构：`manifest.json, layout.json, preview.png, assets/`。
  4. Template 与 Theme 保持解耦：Template 控制"结构"，Theme 控制"外观"（Chapter 8.8 "Template controls composition. Theme controls appearance."）。
- **Tests**：交换同一 Document 的 Theme 而保持 Template 不变，验证布局结构不变、仅外观变化；反之亦然。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 5.4 — Theme/Template 版本化与兼容性记录

- **Purpose**：实现 Chapter 10.18 版本记录要求，为渲染可复现性提供依据。
- **Dependencies**：Task 5.1, 5.3
- **Files**：`packages/theme/src/versioning/`、`packages/template/src/versioning/`
- **Acceptance Criteria**：
  1. 每次渲染记录 `Document Version, Theme Version, Template Version, Skill Version`（Chapter 10.18）。
  2. Theme / Template / Skill / Plugin / Export Format 各自独立版本化，不与应用版本绑定（Chapter 9.16）。
- **Tests**：渲染一次文档后，验证渲染产物元数据中包含上述四个版本字段。
- **Estimated Complexity**：S
- **Status**：Not Started

---

## Phase 6 — 布局引擎（`packages/layout`）

**对应章节**：Chapter 07.9–07.10（Layout Engine / Layout Tree）

### Task 6.1 — 实现 Layout Engine

- **Purpose**：实现 Chapter 7.9 定义的空间信息计算：定位、间距、网格、分栏、对齐、安全区、图片插槽。
- **Dependencies**：Task 2.4, Task 5.3
- **Files**：`packages/layout/src/engine/`
- **Acceptance Criteria**：
  1. 职责覆盖 Chapter 7.9 全部七项：`positioning, spacing, grid, column, alignment, safe area, image slots`。
  2. Layout Engine 绝不绘制像素（"The Layout Engine should never draw pixels."，Chapter 7.9）。
  3. 输出为 Layout Tree，节点仅持有几何信息，不含渲染信息（Chapter 7.10 "No rendering information exists here."）。
- **Tests**：给定含多种 Block 类型的 Document，验证输出 Layout Tree 中每个节点仅包含几何字段（无颜色/字体等渲染字段）。
- **Estimated Complexity**：L
- **Status**：Not Started

---

## Phase 7 — 排版引擎：集成 Pretext（`packages/typography`）

**对应章节**：Chapter 03.3 / Chapter 04.5（Pretext 职责与集成策略）、Chapter 07.12（Typography Engine）

### Task 7.1 — 提取并集成 Pretext 排版算法

- **Purpose**：从 Pretext（https://github.com/chenglou/pretext）中提取排版算法（文本测量、断行、段落布局），轻量化集成，不迁移整个项目（Chapter 4.5 "Integration Strategy"）。
- **Dependencies**：Task 0.2
- **Files**：`packages/typography/src/pretext-adapter/`
- **Acceptance Criteria**：
  1. 仅提取提升渲染质量所需的排版特性，不做整体迁移（Chapter 4.5 "Only integrate features that improve rendering quality."）。
  2. Pretext 不作为渲染器替代品，不接管 oneimg 的渲染职责（Chapter 4.5 "It is NOT a replacement for oneimg."）。
  3. 集成保持轻量（"Keep the integration lightweight."）。
- **Tests**：使用同一段落文本，对比集成前后的断行结果差异，验证排版质量提升且无渲染职责越界。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 7.2 — 实现 Typography Engine

- **Purpose**：实现 Chapter 7.12 定义的排版职责：断行、自动换行、混合语言、垂直节奏、文本测量、字体回退。
- **Dependencies**：Task 7.1
- **Files**：`packages/typography/src/engine/`
- **Acceptance Criteria**：
  1. 职责覆盖 Chapter 7.12 全部六项：`line breaking, word wrapping, mixed language, vertical rhythm, text measurement, font fallback`。
  2. 中英文混排场景下断行结果符合排版惯例（Chapter 7.12 "mixed language"）。
  3. 字体缺失时触发 Fallback Font 机制（与 Task 3.4 错误恢复联动）。
- **Tests**：中英混排文本用例；字体缺失回退用例；长段落断行快照测试。
- **Estimated Complexity**：L
- **Status**：Not Started

---

## Phase 8 — 平台适配器（`packages/platform`）

**对应章节**：Chapter 07.16（Platform Adapter）、Chapter 08.12（Platform 领域对象）

### Task 8.1 — 实现 Platform Adapter

- **Purpose**：实现 Chapter 7.16 定义的平台导出规格转换，逻辑归属 Platform Adapter 而非 Renderer。
- **Dependencies**：Task 1.5
- **Files**：`packages/platform/src/adapters/`
- **Acceptance Criteria**：
  1. 至少支持 Chapter 7.16 明确给出的平台规格示例：Twitter/X（16:9，1200×675）、Instagram（4:5，1080×1350）、Threads（portrait）、LinkedIn（landscape）、Rednote（portrait）。
  2. 平台相关逻辑完全位于 Platform Adapter 内，不出现在 Renderer 中（Chapter 7.16 "Platform logic belongs here. Not inside Renderer."）。
  3. 业务逻辑保持平台无关，平台特定行为仅存在于 Platform Adapter（Final Section #12 "Platform Independent"）。
- **Tests**：每个受支持平台各一个导出尺寸校验用例，验证输出分辨率/比例与规格一致。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 9 — 导出引擎（`packages/export`）

**对应章节**：Chapter 07.15（Export Engine）、Chapter 08.14（ExportJob）、Chapter 04.13（统一导出接口）

### Task 9.1 — 实现统一导出接口与 PNG/JPG/WebP/PDF 支持

- **Purpose**：实现 Chapter 7.15 与 Chapter 4.13 定义的导出能力。
- **Dependencies**：Task 3.3, Task 1.5
- **Files**：`packages/export/src/`
- **Acceptance Criteria**：
  1. 支持格式：`PNG, JPG, WebP, PDF`（Chapter 7.15, 4.13）。
  2. Exporter 仅接收渲染输出（Render Tree/Canvas 产物），不了解内容来源，不依赖 Markdown（Chapter 7.15 "Exporter must remain independent of Markdown."，Chapter 4.13 "Exporters should never know where the content came from."）。
  3. 导出产物元数据封装为 ExportJob 对象（`Platform, Resolution, Scale, Format, Quality, Watermark, Output Path, Timestamp`，Chapter 8.14）。
  4. 预留 SVG（future）扩展点，但本阶段不实现 SVG/AVIF 导出（Chapter 7.15 明确标注为 Future）。
- **Tests**：四种格式各一个导出成功用例；验证 Exporter 模块不直接 import Markdown 相关包（静态依赖检查）。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 9.2 — 批量导出支持

- **Purpose**：实现 Chapter 6.4 export 包职责中提及的批量导出（Batch export）。
- **Dependencies**：Task 9.1
- **Files**：`packages/export/src/batch/`
- **Acceptance Criteria**：
  1. 支持对多个 Document 或多个 Platform 预设批量执行导出任务。
  2. 单个导出任务失败不阻塞批次中其余任务（与 Chapter 7.18 错误恢复原则一致）。
- **Tests**：批量导出 3 个以上文档，其中 1 个故意构造失败场景，验证其余任务正常完成。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 10 — AI Skill 层统一基础设施

**对应章节**：Chapter 05（Content Intelligence Layer 总纲）、Chapter 04.9–04.11（Skill 生命周期与统一接口）、Chapter 09.9（Provider 架构）、Chapter 09.13（Skill 架构）

### Task 10.1 — 实现 Skill 统一生命周期框架

- **Purpose**：实现 Chapter 4.9 / Chapter 9.13 定义的 Skill 生命周期：`Input → Skill Context → Skill Execution → Structured Output → Renderer`，以及 `Manifest → Prompt → Configuration → Execution → Patch`。
- **Dependencies**：Task 1.5, Task 2.4
- **Files**：`packages/shared/src/skill-runtime/`
- **Acceptance Criteria**：
  1. 每个 Skill 输入为 `Document`，输出为 `Document Patch`，不得修改 Renderer 或直接操作 Canvas（Chapter 8.11, 5.5, 5.6）。
  2. Skill 元数据模型覆盖 Chapter 4.10：`Skill Metadata, Input Schema, Output Schema, Configuration, Prompt, Version, Capabilities`。
  3. 新 Skill 接入无需修改应用架构（Chapter 4.9 "New Skills must never require architecture changes."）。
  4. 单个 Skill 执行失败按 Chapter 7.18 规则处理为 "Skip Skill"，不影响主流程。
- **Tests**：注册一个模拟 Skill 并执行完整生命周期；构造一个执行失败的 Skill，验证主渲染流程继续（联动 Task 3.4）。
- **Estimated Complexity**：L
- **Status**：Not Started

### Task 10.2 — 实现 AI Provider 抽象层

- **Purpose**：实现 Chapter 9.9 / Chapter 5.9 定义的 AI Provider 独立性：业务逻辑不得直接依赖单一模型或直接调用 Provider SDK。
- **Dependencies**：Task 10.1
- **Files**：`packages/shared/src/ai-provider/`
- **Acceptance Criteria**：
  1. Provider 接口支持可插拔的 Provider 列表：`Claude, Codex, OpenAI, Gemini, Local LLM`（Chapter 9.9 示例，与 Chapter 5.9 一致）。
  2. 切换 Provider 仅需修改配置，不需修改业务逻辑（Chapter 9.9 "Switching providers should require configuration only."）。
  3. UI 组件不得直接调用 Provider SDK（Chapter 11.19 反模式 "Calling AI providers directly from UI components." 明确禁止）。
- **Tests**：使用两个不同的 mock Provider 执行同一 Skill 调用，验证业务逻辑代码无需改动即可切换。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 10.3 — 实现 Prompt 独立管理机制

- **Purpose**：实现 Chapter 5.10 定义的 Prompt 独立性：不得散落于代码库各处，需版本化、可替换、可配置。
- **Dependencies**：Task 10.1
- **Files**：`packages/shared/src/prompt-registry/`
- **Acceptance Criteria**：
  1. 每个 Skill 拥有独立的 Prompt 文件，Prompt 与业务代码分离（Chapter 5.10, 5.11）。
  2. Prompt 支持版本号标注。
  3. Skill 打包结构覆盖 Chapter 5.11：`metadata, prompt, examples, configuration, documentation, version`。
- **Tests**：验证代码扫描未发现内联硬编码 Prompt 字符串（静态检查）；替换某 Skill 的 Prompt 版本后验证行为按新版本执行。
- **Estimated Complexity**：S
- **Status**：Not Started

### Task 10.4 — 实现智能管线编排（Intelligence Pipeline）

- **Purpose**：实现 Chapter 5.3 / Chapter 5.12 定义的可选智能管线与推荐执行顺序。
- **Dependencies**：Task 10.1, 10.2, 10.3
- **Files**：`packages/shared/src/intelligence-pipeline/`
- **Acceptance Criteria**：
  1. 管线阶段：`Semantic Analysis → Content Enhancement (optional) → Design Enhancement (optional) → Illustration Generation (optional) → Rendering`（Chapter 5.3）。
  2. 每个可选阶段均可被用户跳过，跳过后主流程仍可继续（Chapter 5.3 "Every stage may be skipped."）。
  3. 推荐执行顺序为 `Content → Design → Illustration`（Chapter 5.12），管线默认按此顺序执行。
- **Tests**：验证跳过全部可选阶段时文档仍可正常渲染；验证默认执行顺序符合 Content → Design → Illustration。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 11 — 内容智能：集成 Viral Writer Skill

**对应章节**：Chapter 03.3 / Chapter 04.8（Viral Writer 职责与集成策略）、Chapter 05.6（Content Intelligence）

### Task 11.1 — 集成 Viral Writer Skill

- **Purpose**：集成 Viral Writer Skill（https://github.com/nashsu/Viral_Writer_Skill），在布局开始前提升写作质量。
- **Dependencies**：Task 10.1, 10.2, 10.3
- **Files**：`skills/viral-writer/`
- **Acceptance Criteria**：
  1. 生成能力覆盖 Chapter 4.8：`Headlines, Hooks, CTA, Story Structure, Viral Copy, Markdown Output`。
  2. 输出必须始终为 Markdown，禁止输出 HTML（Chapter 4.8 "Output should always remain Markdown. Never output HTML."，Chapter 3.3, 5.6）。
  3. Content Intelligence 接收 Markdown/Document，仅可修改 `headline, subtitle, paragraph, CTA, story structure`，输出仍为 Document Model（Chapter 5.6, 7.8）。
  4. 数据流为：`Markdown Draft → Skill → Improved Markdown → Editor → Renderer`（Chapter 4.8 集成策略图）。
- **Tests**：给定一段草稿 Markdown，验证 Skill 输出为合法 Markdown（非 HTML）；验证输出可重新导入 Editor。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 12 — 设计智能：集成 Taste Skill

**对应章节**：Chapter 03.3 / Chapter 04.6（Taste Skill 职责与集成策略）、Chapter 05.7（Design Intelligence）

### Task 12.1 — 集成 Taste Skill

- **Purpose**：集成 Taste Skill（https://github.com/Leonxlnx/taste-skill），生成设计建议而非直接渲染。
- **Dependencies**：Task 10.1, 10.2, 10.3
- **Files**：`skills/taste/`
- **Acceptance Criteria**：
  1. 生成能力覆盖 Chapter 4.6：`Layout Suggestions, Typography Suggestions, Color Suggestions, Visual Hierarchy, Spacing Improvements, Design Consistency`。
  2. Taste Skill 不渲染图像，不直接修改渲染代码（Chapter 4.6 "Taste Skill does NOT render images."；Chapter 3.3 "Taste Skill never directly modifies rendering code."）。
  3. 输出的设计指令由 Theme Engine、Template Engine、Layout Engine、Renderer 消费（Chapter 4.6 集成策略）。
  4. Design Intelligence 从不修改 Markdown 本身（Chapter 5.7 "Design Intelligence never modifies Markdown."），输出示例字段包含 `Typography, Font Scale, Spacing, Hierarchy, Background, Accent Color, Alignment`（Chapter 5.7）。
- **Tests**：验证 Taste Skill 输出结果中不包含 Markdown 文本变更；验证输出结构可被 Layout Engine 消费的最小用例。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 13 — 插画智能：集成 Ian Xiaohei Illustrations

**对应章节**：Chapter 03.3 / Chapter 04.7（Ian Xiaohei 职责与集成策略）、Chapter 05.8（Illustration Intelligence）

### Task 13.1 — 集成 Ian Xiaohei Illustrations

- **Purpose**：集成 Ian Xiaohei Illustrations（https://github.com/helloianneo/ian-xiaohei-illustrations），生成插画资产。
- **Dependencies**：Task 10.1, 10.2, 10.3
- **Files**：`skills/xiaohei/`
- **Acceptance Criteria**：
  1. 生成能力覆盖 Chapter 4.7：`Doodles, Sketches, Funny graphics, Annotations, Hand-drawn illustrations`。
  2. 插画输入可为 Markdown 或 AST，输出为 Illustration Assets（Chapter 5.8）。
  3. 插画为装饰性资产，不决定布局（Chapter 4.7 "Illustrations never determine layout."，Chapter 3.3）；插画资产进入 Layout Engine，由 Layout Engine 决定放置位置（Chapter 4.7 集成策略）。
  4. 渲染器不感知插画来源（Chapter 5.8 "The renderer remains unaware of where they originated."）。
  5. 生成的 Illustration 对象包含元数据字段：`Style, Prompt, Author, Source, Version, License`（Chapter 8.10）。
- **Tests**：生成一个插画资产，验证其元数据字段齐全；验证插画资产接入 Layout Engine 后不改变已计算的布局位置决策权归属。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 14 — 应用层（`apps/desktop`, `apps/web`）

**对应章节**：Chapter 06.3（应用层职责）、Chapter 09.4（Application/Presentation 层职责）

### Task 14.1 — 搭建 Desktop 应用入口

- **Purpose**：实现 Chapter 6.3 定义的应用层职责——仅作为入口，不承载业务逻辑。
- **Dependencies**：Task 0.2, Phase 1–13 中相关 package 已具备最小可用能力
- **Files**：`apps/desktop/`
- **Acceptance Criteria**：
  1. Desktop 应用仅负责启动与装配 packages，不包含业务逻辑（Chapter 6.3 "Business logic belongs inside packages."，Chapter 9.4 "Application 负责 startup"）。
  2. 基于 Tauri 2 运行（Chapter 9.6, 9.7）。
  3. 应用启动时间 < 2s（Chapter 11.16 性能预算）。
- **Tests**：启动性能基准测试；静态检查应用入口代码中不包含渲染/解析/AI 调用等业务逻辑实现（仅允许调用 packages 暴露的 API）。
- **Estimated Complexity**：L
- **Status**：Not Started

### Task 14.2 — 搭建 Web 应用入口

- **Purpose**：实现 Chapter 6.3 定义的 Web 应用入口，复用与 Desktop 相同的 packages。
- **Dependencies**：Task 14.1
- **Files**：`apps/web/`
- **Acceptance Criteria**：
  1. Web 应用与 Desktop 应用共享同一套 `packages/`，不重复实现业务逻辑（Chapter 9.2 Monorepo 策略 "Editor/Renderer/Theme/Skill/Plugin/Desktop/Web share most code."）。
  2. 应用层职责边界与 Task 14.1 一致，仅作装配入口。
- **Tests**：验证 Web 与 Desktop 两个入口引用的核心包版本一致；核心渲染/解析逻辑不存在重复实现（代码重复率检查）。
- **Estimated Complexity**：M
- **Status**：Not Started

### Task 14.3 — 实现本地优先的数据持久化

- **Purpose**：实现 Chapter 2.13 / Final Section #13（Local First / Offline First）：草稿、模板、主题、资源、配置均可本地存储与离线可用。
- **Dependencies**：Task 14.1, Task 1.1
- **Files**：`packages/shared/src/local-storage/`
- **Acceptance Criteria**：
  1. `Drafts, Templates, Themes, Assets, Settings` 均可在无网络连接情况下正常读写（Chapter 2.13, 9.20）。
  2. 核心文档编辑与渲染功能在断网状态下可用；仅 AI 能力允许依赖外部服务（Final Section #13 "AI capabilities may require external services, but document editing and rendering should remain available offline whenever possible."）。
  3. 云同步不作为本阶段必需功能，仅预留未来扩展点（Chapter 2.13 "Cloud synchronization may be added later but must never become mandatory."）。
- **Tests**：断网环境下完成"编辑 Markdown → 渲染预览 → 本地导出"全流程集成测试。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 15 — 插件架构（`plugins/`）

**对应章节**：Chapter 02.8（Plugin First）、Chapter 06.8（Plugin 目录）、Chapter 09.10（Plugin 架构）

### Task 15.1 — 实现插件生命周期框架

- **Purpose**：实现 Chapter 9.10 定义的插件生命周期：`Discover → Load → Validate → Register → Execute → Unload`。
- **Dependencies**：Task 0.3, Task 1.1
- **Files**：`packages/shared/src/plugin-runtime/`
- **Acceptance Criteria**：
  1. 插件生命周期六个阶段均已实现并可观测（日志/状态）。
  2. 插件仅通过 API 与核心通信，核心应用不依赖任何具体插件（Chapter 9.10 "Plugins communicate through APIs only."，Chapter 6.8 "The core application should never depend on plugins."）。
  3. 插件目录结构遵循 Chapter 6.8（预留 Markdown 扩展、Exporter、Theme Provider 等插件类型的注册位置，但本阶段不实现具体业务插件，仅实现框架）。
- **Tests**：注册一个最小示例插件并验证完整生命周期执行；移除该插件后核心应用可正常运行（验证无强依赖）。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## Phase 16 — 测试与验证基础设施

**对应章节**：Chapter 06.10（测试目录规范）、Chapter 09.19（测试策略）、Chapter 07.20（渲染验证要求）、Chapter 11.12（测试要求）

### Task 16.1 — 建立分层测试目录与快照/视觉回归基础设施

- **Purpose**：落地 Chapter 6.10 测试目录结构与 Chapter 9.19 测试层级：`Unit → Integration → Snapshot → Visual Regression → End-to-End`。
- **Dependencies**：Phase 0–3 基本完成
- **Files**：`tests/renderer/`、`tests/layout/`、`tests/parser/`、`tests/skills/`、`tests/integration/`、`tests/snapshot/`
- **Acceptance Criteria**：
  1. 测试目录与 `packages/` 一一镜像，每个 package 拥有自己的测试（Chapter 6.10 "Tests should mirror packages."）。
  2. 渲染相关变更强制要求快照测试（Chapter 6.10 "Snapshot tests are strongly recommended for rendering."，Chapter 7.20）。
  3. 渲染验证方法覆盖 Chapter 7.20：`Snapshot tests, Pixel comparison, Golden image tests, Typography regression tests, Platform export validation`。
  4. CI 流程包含 Unit → Integration → Snapshot → Visual Regression → E2E 五个阶段（Chapter 9.19）。
- **Tests**：CI 配置文件中可见上述五个测试阶段均被执行且互相独立。
- **Estimated Complexity**：L
- **Status**：Not Started

---

## Phase 17 — 文档体系

**对应章节**：Chapter 06.11（文档目录规范）、Chapter 12.12 / 12.15（文档与交付物）

### Task 17.1 — 建立 `docs/` 文档体系

- **Purpose**：落地 Chapter 6.11 建议的文档集合。
- **Dependencies**：Phase 0–16 各阶段完成后同步更新
- **Files**：`docs/architecture.md`、`docs/integration.md`、`docs/theme-specification.md`、`docs/template-specification.md`、`docs/plugin-guide.md`、`docs/skill-guide.md`、`docs/development-rules.md`、`docs/contribution-guide.md`
- **Acceptance Criteria**：
  1. 文档集合覆盖 Chapter 6.11 建议的全部八类文档。
  2. 每完成一个功能阶段，对应文档需同步更新（Chapter 6.11 "Documentation should evolve together with the codebase."，Chapter 12.12）。
  3. 具备生成 Chapter 12.15 列出的工程交付物模板（`implementation-plan.md, tasks.md, progress.md, architecture-review.md, release-notes.md, migration-guide.md, test-report.md`）能力，作为后续每个功能开发的固定产出格式。
- **Tests**：文档链接有效性检查（无死链）；每个文档存在对应的"最后更新"记录。
- **Estimated Complexity**：M
- **Status**：Not Started

---

## 阶段依赖总览

```text
Phase 0（基础设施）
  └─ Phase 1（领域模型/Schema）
       ├─ Phase 2（Markdown 解析/AST）
       │     └─ Phase 3（渲染引擎/oneimg）
       │           └─ Phase 4（编辑器/doocs-md）
       ├─ Phase 5（Theme/Template）
       │     └─ Phase 6（布局引擎）
       │           └─ Phase 7（排版引擎/Pretext）
       ├─ Phase 8（平台适配器）
       └─ Phase 9（导出引擎）
Phase 3,4,5,6,7,8,9 完成后 →
  Phase 10（AI Skill 统一基础设施）
       ├─ Phase 11（Viral Writer）
       ├─ Phase 12（Taste Skill）
       └─ Phase 13（Ian Xiaohei）
Phase 3–13 具备最小可用能力后 →
  Phase 14（应用层 Desktop/Web）
       └─ Phase 15（插件架构）
贯穿全程 → Phase 16（测试基础设施）、Phase 17（文档体系）
```

---

## 完成定义（Definition of Done）— 适用于本清单全部任务

依据 Chapter 11.21，一个任务只有在同时满足以下条件时才可标记为 Done：

1. 需求满足（对应本任务 Acceptance Criteria 全部通过）
2. 架构未被破坏（符合 Chapter 03 / Chapter 09 依赖方向与分层）
3. 测试通过（对应本任务 Tests 全部通过）
4. 文档已更新
5. 无 Lint 错误、无类型错误
6. 视觉输出已验证（涉及渲染的任务）
7. 导出已验证（涉及导出的任务）
8. 未影响既有功能（回归测试通过）

---

**文档版本**：tasks.md v1.0
**依据规格文档版本**：MD2Pic MRD v1.0（2026-07-11）
**状态**：待项目负责人（Owner）审批后进入 Phase 0 实施（依据 Chapter 12.2 标准工作流，Task Generation 之后需 Owner Approval 方可 Implementation）
