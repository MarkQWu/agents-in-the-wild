# nexu-io/html-anything — 学习案例

**仓库**：https://github.com/nexu-io/html-anything
**Stars**：3378 | **来源**：本地 audit
**Audit 日期**：2026-05-19（历史快照）| **生成日期**：2026-05-20（基于当前 HEAD，与 audit 近乎同步）
**主题标签**：`template-design`, `examples-driven`, `vague-quantifier`, `manifest-discipline`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
「将任何内容变成 HTML 单文件」的 Claude Code 技能平台——用户描述想要的内容，Claude 调用对应技能模板生成一个可直接在浏览器打开的 HTML 文件，覆盖演示文稿、社交卡片、数据报告、设备展架、海报、电子书等 75 种模板。技术上是一个定制化 Next.js 16.2.6 Web 应用（有 Breaking Changes），技能文件嵌入在 Next.js 源码路径里。

关键事实：
1. 3378 颗星——不是典型「Claude Code 插件仓库」，而是一个完整 Web 应用，技能是它的 AI 接口层
2. 采用 pnpm workspace 多包结构：`next/`（主应用）、`e2e/`（Playwright E2E 测试）、`scripts/`（守护脚本）相互隔离
3. 75 个技能文件质量双峰分布：16 个有示例（score 88-95）vs. 59 个无示例（score 65-85）
4. 双语 [frontmatter](#frontmatter)：每个技能有 `zh_name`（中文名）和 `en_name`（英文名），面向中英文用户
5. AGENTS.md 包含强健的 AI 开发规则（「这不是你熟悉的 Next.js」警告 + 工作区边界约束）

### 1.2 架构剖析
- **目录结构**：
```
html-anything/
  CLAUDE.md              # 只有 @AGENTS.md（无 frontmatter）
  AGENTS.md              # AI 开发规则（工作区边界约束）
  next/                  # 主 Next.js 应用（16.2.6，非标准版本）
    src/lib/templates/skills/
      mockup-device-3d/  # ⭐ 精品技能（score 95）：3D 设备展架
      deck-swiss-international/ # 精品：瑞士风格演示（93）
      social-x-post-card/  # 精品：X 帖子卡片（95）
      web-proto-soft/    # 待改进（score 65）：Apple Soft 原型
      motion-frames/     # 待改进（score 70）
      ... (共 75 个)
    package.json         # xlsx ^0.18.5（安全风险）
  e2e/                   # E2E 测试（Playwright）
  scripts/
    guard.ts             # 工作区边界守护脚本
  pnpm-workspace.yaml    # Monorepo 配置
```
- **文件类型分布**：75 个 skill，0 个 agent，0 个 command，0 个 [manifest](#manifest)（plugin.json），3 个 package.json，1 个 guard.ts
- **编排关系**：无编排。75 个技能完全平列，用户根据内容类型选择调用对应技能，技能生成 HTML 并由 Next.js 渲染。技能文件之间无引用关系
- **跨件契约**：技能文件路径约定（`next/src/lib/templates/skills/{name}/SKILL.md`），Next.js 应用扫描这个目录生成技能选择界面。`AGENTS.md` 定义了 AI 必须遵守的工作区边界：next/ 内的代码不能移到根目录，E2E 测试只能在 e2e/ 下，root package.json 不能代理应用脚本

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「技能作为参数化模板」——每个技能不是「教 Claude 怎么写 HTML」，而是「定义一种 HTML 类型的精确视觉约束」（尺寸、颜色 hex 值、CSS 关键属性、渲染框架），Claude 根据这些约束生成符合标准的输出
- **解决什么问题**：Claude 生成 HTML 时风格不一、格式不规范、无法还原特定视觉效果（如 iPhone 展架的玻璃镜头折射）。精确的设计约束技能让输出可复现、风格一致
- **做了什么 trade-off**：把 75 个技能文件嵌入 `next/src/lib/templates/skills/` 路径是一个「代码和配置混合」的设计选择，优点是技能选择界面和技能文件紧耦合不会失同步；缺点是「技能」不能单独安装到 Claude Code，必须运行整个 Web 应用
- **反映什么认知模型**：作者把 AI 技能视为「视觉设计规格书」——不是自然语言描述（「生成一个漂亮的幻灯片」），而是精确的视觉语言（「Silver/cream canvas + double-bezel 卡片 + spring motion + ambient mesh，1440px 宽」）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「NL 技能嵌入式 Web 应用」（Skills-Embedded Web App）**

技能文件不是独立的 Claude Code 插件，而是嵌在 Web 应用源码里的 Markdown 配置文件。应用的功能（生成 HTML）和 AI 的接口（技能模板）在同一个仓库里共演进。

模式特征清单（5 条）：
- 特征 1：技能路径约定（`next/src/lib/templates/skills/{name}/SKILL.md`）而非 plugin.json
- 特征 2：技能质量双峰：有 `example_id` 的 16 个高质量技能 vs. 无示例的 59 个基础技能
- 特征 3：双语 frontmatter（zh_name + en_name）支持中英文用户
- 特征 4：AGENTS.md 严格约束 AI 开发边界（防止 AI 在错误的目录里改代码）
- 特征 5：技能 = 视觉规格书，不是操作指南

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| AI 功能紧耦合的 Web 产品（技能即产品功能） | ✅ 高度适用 | 技能和应用同仓演进，不会失同步 |
| 需要在 Claude Code marketplace 发布的独立插件 | ❌ 不适用 | 无 plugin.json，无法独立安装 |
| 需要 AI 开发协作的多包 monorepo 项目 | ✅ 适用 | AGENTS.md 的工作区边界规则是可复用的最佳实践 |
| 小型个人 side project | ⚠️ 依情况 | 75 个技能的维护成本和双峰质量问题需要提前规划 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 技能嵌入式 Web 应用（本仓库） | html-anything | 技能和产品同步，UI 一体化 | 无法独立安装，需要运行整个应用 |
| 独立插件（带 plugin.json） | AgriciDaniel/claude-ads | marketplace 可发现，standalone 安装 | 需要独立维护，UI 依赖 Claude Code |
| 单仓多 agent 平铺 | 0xfurai/claude-code-subagents | 零配置，极简 | 无 UI，无 manifest，无 Web 界面 |

### 2.4 改进空间
1. **当前问题**：CLAUDE.md 没有 frontmatter（只有 `@AGENTS.md` 一行），缺少 `name` 和 `description` 字段，NLPM 扫描器无法注册。**改进做法**：在文件顶部加 `---\nname: html-anything-claude\ndescription: Claude AI integration layer for html-anything\n---`。**预期收益**：NLPM 发现此文件，bug 修复，评分从 50 提升。
2. **当前问题**：59 个无示例技能（score 65-82）和 16 个有示例技能（score 88-95）之间存在明显质量鸿沟。**改进做法**：优先给 25 个最常用技能（按 `featured:` 字段排序）添加 `example_id` + `example_desc`，逐步推进。**预期收益**：仓库整体 NLPM 均分从 80 提升到约 88。
3. **当前问题**：`xlsx@^0.18.5` 是 2023 年后实际上停止维护的 SheetJS 社区版，有已知未修复的 CVE。**改进做法**：迁移到 `exceljs` 或 `@e965/xlsx`（社区维护的 fork），审计所有文件解析路径的路径遍历风险。**预期收益**：消除 Medium 安全风险，使用有安全维护的库。

---

## 三、过去审查发现（2026-05-19 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-19 当时得分 **80/100**（76 个文件，含 CLAUDE.md）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 50 | 缺 name frontmatter（-25）、缺 description（-25） |
| web-proto-soft/SKILL.md | 65 | 4 行正文，无输出格式，无尺寸规格 |
| motion-frames/SKILL.md | 70 | "流畅可循环" 是模糊词，无 timing spec |
| hr-onboarding/SKILL.md | 70 | 只列内容章节，无 HTML/CSS 锚点 |
| mockup-device-3d/SKILL.md | 95 | 完整规格（硬性构图、CSS 3D 值、示例） |
| social-x-post-card/SKILL.md | 95 | 完整规格 + 详细约束 |
| deck-swiss-international/SKILL.md | 93 | 22 种布局，single-file HTML 明确 |

### 3.2 当时值得借鉴的模式
1. **视觉规格书技能**（template-design）→ 为什么好：`mockup-device-3d/SKILL.md` 不是「制作一个设备展架」，而是「画布 1920×1080，iPhone 15 Pro，`transform: rotateY(-12deg) rotateX(4deg)`，边框 `#a8a8ad`，屏幕圆角 56px」——精确到 CSS 属性和颜色 hex → 原文路径：`next/src/lib/templates/skills/mockup-device-3d/SKILL.md` → 如何借鉴：在技能文件里给视觉/格式约束写具体值（颜色 hex、尺寸 px、CSS 属性名），不用「美观」「现代感」这种形容词
2. **AGENTS.md 工作区边界约束**（single-purpose）→ 为什么好：明确告诉 AI「next/ 下的代码不能移到根目录」「E2E 测试只能在 e2e/ 下」，这防止了 AI 在代码生成时跨越 monorepo 的包边界 → 原文路径：`AGENTS.md` 的 Workspace Shape 节 → 如何借鉴：在 AGENTS.md 里为每个 AI 协作者定义明确的「可写路径」和「禁止路径」
3. **双语 frontmatter**（template-design）→ 为什么好：`zh_name` + `en_name` 让技能选择界面同时服务中英文用户，frontmatter 本身成为 i18n 的配置层 → 原文路径：所有技能的 frontmatter → 如何借鉴：如果目标用户包含非英语用户，在 frontmatter 里加本地化名称字段

### 3.3 当时的缺陷
1. **CLAUDE.md 无 frontmatter**：文件只有 `@AGENTS.md` 一行，缺失 `name` 和 `description` 字段，NLPM 扫描器对这个文件评分 50/100（-25 -25）。为什么会失败：CLAUDE.md 是 Claude Code 读取项目指令的主文件，缺少 frontmatter 会导致 NLPM 等工具无法正确注册和分类这个文件。**自查**：我的 CLAUDE.md 有没有 frontmatter？
2. **59 个技能缺乏输出格式规范**：低分技能（score 65-75）的正文只有 4-9 行，没有明确的「输出格式」声明（如「单文件 HTML，宽 1440px」）。为什么会失败：Claude 在没有明确输出格式约束时会自行发挥，同一技能被不同用户调用可能得到格式完全不同的输出。**自查**：我的技能文件里有没有「输出格式」（Output Format）节？
3. **`xlsx@^0.18.5` 依赖风险**：SheetJS 社区版自 2023 年改为商业授权后停止 CVE 修复，`^` 范围允许自动升级至 0.18.x 内的最新版，但整条 0.18.x 线都没有安全维护。为什么会失败：如果解析的 Excel 文件包含特制的恶意载荷，未修复的 CVE 可能被触发执行任意代码或路径遍历。**自查**：我的 package.json 里有没有用 `xlsx` 包？版本是多少？

### 3.4 当时的优化机会
1. 给 CLAUDE.md 加 frontmatter（name + description），两行改动
2. 给最常用的 25 个技能（有 `featured:` 字段的）补充输出格式规范，参照 `mockup-device-3d/SKILL.md` 的「硬性构图」节作为模板
3. 迁移 `xlsx` → `exceljs`，同时审计 `marked@^18` 和 `modern-screenshot@^4.7` 固定到精确版本

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

> 注意：本仓库 audit 日期（2026-05-19）与案例生成日期（2026-05-20）只相差 1 天，当前 HEAD 与 audit 快照近乎同步，几乎不存在「修复了但 audit 未覆盖」的情况。

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| CLAUDE.md 缺 frontmatter | `head -3 CLAUDE.md` | **持续存在**：仍只有 `@AGENTS.md`，无 `---` frontmatter 块 | Bug 未修，评分 50 的扣分原因还在 |
| xlsx ^0.18.5 | `grep "xlsx" next/package.json` | **持续存在**：`"xlsx": "^0.18.5"` 仍在 | Medium 安全风险未解 |
| web-proto-soft 无输出格式 | 读取 SKILL.md 正文 | **持续存在**：正文有风格描述但无明确 HTML 输出格式声明 | 低分技能质量状态未变 |

### 4.2 架构演进
Audit 时已是最新状态（仅 1 天差距），无可对比的演进数据。但有一个元架构观察值得记录：仓库有 `PROGRESS.md` 文件（追踪功能开发进度），说明这是一个「有版本规划」的活跃项目，不是静态的技能合集——未来改进的可能性很高。

**横向观察**：相比 0xfurai（3 周无修复）、777genius（3 周无修复），本仓库的 audit 时间非常新（昨天），还来不及产生对比数据。这本身就是一个提醒：对非常新的 audit，要降低「缺陷已修复」的期待。

### 4.3 新增的可学习模式
暂无（audit 与当前 HEAD 同步，无时间窗口形成演进）。但有一个设计细节值得记录：`scripts/guard.ts` 是一个 TypeScript 实现的「工作区边界守护脚本」，运行 `pnpm exec tsx scripts/guard.ts` 可以检查文件是否被放置在错误的目录（例如应用代码出现在根目录）。这种「结构验证即代码」的做法很有意思——不是靠文档规范，而是靠脚本强制执行工作区约定。

---

## 五、校准

### 5.1 我已经在做对的
1. **CLAUDE.md 有 frontmatter**：我的 CLAUDE.md 包含 `name` 和 `description`，和本仓库的 bug 不同
2. **不使用 `xlsx@^0.18.5`**：我的项目没有依赖这个停止维护的库
3. **输出格式明确**：我的技能文件都有明确的「输出格式」节，说明期望的输出类型
4. **AGENTS.md 工作区约束**：我的 monorepo 项目使用类似的 AGENTS.md 边界约束

### 5.2 挑战 / 验证
- **这次案例挑战了我对「技能文件应该怎么写」的假设**：我原来认为技能文件应该写「步骤」（先做 A，再做 B，然后做 C），但本仓库最高分技能（mockup-device-3d 95 分）的成功秘密是「写约束而非步骤」——给出硬性的视觉规格（CSS 属性值、hex 颜色、尺寸），让 Claude 在这些约束范围内自由发挥。**这改变了我下次写视觉类技能的思路**：从「怎么做」改为「做成什么样」，从动词驱动改为规格驱动。
- **验证了一个我已知的做法**：「双峰质量」（少量精品 + 大量基础品）是内容库扩张时的常见陷阱。本仓库 75 个技能里 16 个有示例、59 个无示例，整体均分 80——有示例的 16 个均分约 92，无示例的 59 个均分约 76。数字清楚地说明：宁可做少量精品，也不要用量填充质量空洞。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查自己技能文件里是否有明确的「输出格式」声明
grep -rL "Output Format\|输出格式\|输出.*HTML\|output format" ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中（缺少输出格式）：给技能文件加 ## Output Format 节，明确说明输出类型

# 检查视觉类技能是否用了模糊形容词而非具体规格
grep -rn -E '\b(美观|优雅|现代|漂亮|专业|elegant|beautiful|modern|clean)\b' ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：用具体 hex 颜色、px 尺寸、CSS 属性值替换模糊形容词

# 检查是否有用 xlsx 包（停止维护）
find . -name "package.json" | xargs grep -l '"xlsx"' 2>/dev/null
# 命中后：评估是否可以迁移到 exceljs，或至少固定到已知安全的版本并跟踪 CVE

# 检查 AGENTS.md 或 CLAUDE.md 是否有 frontmatter
for f in CLAUDE.md AGENTS.md; do
  [ -f "$f" ] && head -1 "$f" | grep -q "^---" || echo "MISSING frontmatter: $f"
done
# 命中后：在文件顶部加 ---\nname: xxx\ndescription: xxx\n---
```

### 6.2 灵感 → 实施路径
1. **想法**：仿照 `scripts/guard.ts` 写一个工作区边界守护脚本，防止 AI 在错误的目录里改代码
   - **为何可行**：本仓库的 guard.ts 实现了「结构验证即代码」——比 AGENTS.md 的文档规范更强制；对我有多个子目录的 monorepo 项目很有价值
   - **第一步**：创建 `scripts/guard.ts`，用 `glob` 列出特定路径下的文件，断言不该出现在根目录的文件（如 `*.test.ts`）没有出现在根目录；大约 1 小时
2. **想法**：把我现有的「步骤驱动」视觉技能改写为「规格驱动」
   - **为何可行**：本仓库最高分技能（95 分）的核心差异是精确的视觉规格（CSS 属性值而非形容词）；我的现有视觉技能可能存在同样的问题
   - **第一步**：找一个我自己的视觉技能，把所有「美观」「现代」「简洁」类形容词替换为具体的颜色 hex（用 Apple Human Interface Guidelines 的配色方案作参考）、字体大小（px）、容器宽高（px），约 30 分钟

---

## 七、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读技能文件时先解析 frontmatter 才能知道这个技能如何注册和调用。`---` 必须严格从行首（第 1 列）开始。本仓库的 `CLAUDE.md` 缺少 frontmatter 是 Bug #1 的根本原因。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest。本仓库没有 plugin.json——技能通过 `next/src/lib/templates/skills/` 目录约定被 Next.js 应用扫描发现，而非通过 Claude Code plugin 系统安装。

### <a name="single-file-html"></a>单文件 HTML
> 所有 CSS、JavaScript、图像（base64 编码）都内联在一个 `.html` 文件里，不依赖外部资源。用户下载这一个文件就能在任何浏览器离线运行。本仓库的所有技能输出目标都是单文件 HTML，这是「html-anything」名字的来源。

### <a name="pnpm-workspace"></a>pnpm workspace
> pnpm 的 monorepo 管理功能。`pnpm-workspace.yaml` 文件定义了工作区包含哪些包（如 `next/`、`e2e/`），允许各包有独立的 `package.json` 和依赖，但共享同一个 `node_modules` 提升空间。本仓库用 pnpm workspace 把主应用、E2E 测试、脚本隔离在独立的包边界内，防止依赖污染。

### <a name="GEO-AEO-chart"></a>双峰质量分布
> 内容库扩张时常见的质量分化现象：少数精品文件（有示例、规格完整）评分 88-95，大多数基础文件（无示例、规格简单）评分 65-82，两峰之间没有平滑过渡。这是「快速增量扩张」vs.「精品深耕」之间 trade-off 的可见结果。本仓库 75 个技能里 16 个精品（≥88）vs. 59 个基础（≤85）就是典型的双峰分布。
